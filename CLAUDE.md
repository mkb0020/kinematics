<repo_context>

<meta>
repo: kinematics (renamed from AnimationExperiments on 2026-10-05)
local_path: C:\Users\mkb00\PROJECTS\GitRepos\kinematics
owner: MK (mkb0020 Studios), solo indie dev
purpose: R&D sandbox for a future 2D metroidvania. NOT the game itself. Used to test character animation and VFX techniques before committing to a production pipeline.
stage: early planning / pre-production. Art style still being decided.
doc_audience: Claude (optimized for machine parsing, not human reading)
doc_date: 2026-10-05
doc_basis: written after reading the repo's actual files (headers, config blocks, JSON). Function-level behavior not fully verified; read the file before editing.
</meta>

<art_direction>
style: anime-influenced, 2D. NOT pixel art.
status: unsettled. Do not assume final proportions, line weight, resolution, or palette.
implications:
- Avoid pixel-art techniques (nearest-neighbor-only assumptions, tiny fixed grids, palette-limited sprites).
- Smooth/high-res rendering, anti-aliasing, and sub-pixel motion are acceptable and expected.
- Techniques should be resolution-tolerant, since final sprite size is undecided.
caveat: index.html still sets kaplay crisp:true and some sprites (e.g. spawn.png) are pixel-art-era leftovers. Treat as legacy, not as the direction.
palette_files: colorPalette/colors.json and colorPalette/colors.css (NOTE: folder is "colorPalette", camelCase). Source of truth for colors. JSON shape: {color_palette:[{group, colors:[{hex,name}]}]}, groups like "Darks & Neutrals", "Deep & Royal Blues", "Electric & Neon Blues", "Cyans & Teals", etc.
visual_theme_default: sci-fi / cyberpunk. Deep violet backgrounds (#0c001f / #0c001f-ish), neon cyan/magenta/violet accents, glow.
</art_direction>

<engine_direction>
current: index.html uses Kaplay (JS game lib), pinned via CDN: kaplay@3001.0.19 (ES module from cdn.jsdelivr.net).
plan: migrate away from Kaplay to either plain JS (canvas/WebGL) or Godot. MK leans Godot but is NOT decided. MK has never used Godot.
implications for Claude:
- Do not deepen Kaplay lock-in. Prefer engine-agnostic logic (plain functions, data in JSON, math not tied to Kaplay components) so techniques port to plain JS or GDScript.
- Kaplay-specific code in index.html (components like area()/body(), k.onKeyPress, k.add, Kaplay shader format, preventResolution hacks, hand-rolled oneWay platform) will NOT port directly. When writing new logic, separate the algorithm (verlet, springs, squash/stretch, particles, gait sampling) from the Kaplay API calls.
- When relevant, flag how something would map to Godot (e.g. Skeleton2D/Bone2D, AnimationPlayer, CPUParticles2D/GPUParticles2D, shaders in Godot shading language, CharacterBody2D, Line2D for cloth ribbons).
- Since MK is new to Godot, explain Godot concepts plainly if migration work starts. Do not assume familiarity.
- The rig tools (animatedRig/staticRig) and vectors.html are plain canvas 2D with no engine dependency.
</engine_direction>

<repo_structure>
format: four standalone HTML files at repo root. Each is single-file (inline CSS+JS), no build step, no bundler, no framework. Open directly or via a static server.
serving_note: animatedRig.html fetches json_files/*.json. fetch() fails over file:// so it falls back to embedded copies. For the JSON to load, serve over http (e.g. python -m http.server).

files:

- index.html  (title: "Sprite Test Playground", ~78 KB)
  role: Kaplay mini-platformer sandbox for physics feel, sprite frames, cloak, spells, FX, level art
  engine: Kaplay 3001.0.19 (to be replaced; see engine_direction)
  sheet: images/player.png, 9 cols x 2 rows. Row 1: cols 0-7 run cycle (col 8 unused). Row 2: cols 0-3 idle (cape baked in), 4 jump/launch, 5 fall, 6 spell attack, 7 kick (cape baked in), 8 wall slide (cape baked in).
  features present: run/jump/fall, wall slide + wall jump, dash with ghost trail, spell attack + kick attack with swoosh, one-way platforms (hand-rolled), spring-driven squash and stretch, hit-stop, camera shake, particles (single-array, one draw object), landing puff, spawn dissolve, lamps with flicker + custom lighting fragment shader, tiles + ground art, debug hitboxes.
  cloak modes (key V cycles): 0 "sprites (original)" = layered cloakBottom/cloakTop PNG frames; 1 "spring + warp" = damped spring swing + strip-warp ripple of same art (CAPE_WARP_STRIPS etc.); 2 "verlet cloth" = no art, verlet ribbon chain drawn procedurally (CAPE_CLOTH, CAPE_COLORS). So cloak experiments ALREADY EXIST here; todo #2 can build on them.
  controls: arrows/WASD move, up/W/space jump, dash keys (see DASH_KEYS), Z spell, X kick, R respawn (dissolve), V cape mode, L lighting toggle, F1 debug hitboxes.
  style note: code comments are ALL CAPS by MK's habit. Match local comment style when editing this file.
  config: tunables are top-of-file consts (SCALE 0.5, MOVE_SPEED, JUMP_FORCE, GROUND_Y, wall-slide consts, CAPE_*, SQUASH_*, etc.).

- staticRig.html  (title: "ANIMATION RIG", ~75 KB)
  role: posable skeleton editor / frame-by-frame animation tool. Earlier-generation rig. Drag joints, drag/resize/rotate loose images, keyframes as whole-pose frames, onion skin, front and side view sheets.
  keys: Tab rig/image mode, B bones, V front/side view, G ghost images, R reset, N new frame, Left/Right frames, Space play, O onion skin, Delete frame.
  STALE PATHS: references images/rig/mannequin-front.png, images/rig/mannequin-side.png, and assets/images/background.png. None of these exist in the repo now (images/ is flat, contains mannequin_side.png and mannequin.png). Likely does not load its art as-is. Confirm with MK before fixing.
  status: older than animatedRig; low activity.

- animatedRig.html  (title: "Rig Lab — Character Art", a.k.a. RIG//LAB, ~244 KB, mtime most recent)
  role: side-view procedural run + walk cycle on a joint rig, with real character body-part art drawn over the skeleton.
  status: ACTIVE WORK. Primary file.
  coordinate conventions: character faces +x. Angles in degrees from straight-down vertical, positive = forward. Screen y down. FPS=24 constant.
  art: images/mannequin_side.png = sheet of ten 800x800 frames, each part drawn in place on the mannequin. Parts (ART_NAMES): head, chest, pelvis, upperArm, forearm, handR, handL, thigh, calf, shoe. Falls back to a base64 copy embedded in the HTML if the file cannot load (the embedded copy can go stale vs the PNG). ART_BOX holds per-part bounding boxes. images/mannequin.png is used as a ghost/reference overlay for the pivot editor (must be 800 px wide).
  rig method: each part's own pivot-to-pivot axis is mapped onto the matching skeleton bone (rotate + stretch along the bone). Skeleton bone lengths follow the drawn art's pivots (setArtLengths / recomputeArt).
  pivots (ART_PIV): hip, knee, ankle, shoulder, elbow, wrist, rib, neck, headPivot, neckFrom, neckTo, stomachFrom, stomachTo. Editable live in an "Edit pivots on the art" mode, saved to localStorage (key rigArtPivots_v1; auto-discarded if defaults change), exportable as a JS block.
  stomach: a fill drawn between pelvis and chest (config body.stomach, stomachWChest/stomachWPelvis, stomachColor) to hide the gap when the torso twists/bends. Recently added (see old_versions/*_before_stomach*).
  motion model: key poses per cycle sampled with Hermite/spline interpolation (sampleH/sampleS), phase u in [0,1). Run poses: passing, takeoff, up, contact (per leg, mirrored 4+4 = 8). Walk poses: contact, down, passing, up. Features: twist/hipTwist, headBob/headJut, airtime, lag/footLag (follow-through), squash and stretch per pose, per-pose foot angles for boot vs bare foot, stride and strideBoot, twistPhase/liftPhase.
  UI: slider panels, editable angle grids (GRIDS, FGRIDS), Export/Import buttons for rig_config.json (cfgBuild/cfgApply). Play/pause Space, step poses with Left/Right arrow.
  rendering: canvas 2D, DPR-aware, tinted far-side limb copies (farTint), grid + floor, scrolling background.
  data flow: loadData() fetches json_files/bone_lengths.json, gait_angles.json, rig_config.json, with embedded fallbacks (BONES_FALLBACK, gaitFallback()). Display string dataSrc reports which source was used.
  cloak/hood: NOT in this file yet (grep finds no hood/cloak). Art exists: images/mannequin_side_with_hood.png, images/cloakTop.png, images/cloakBottom.png.

- vectors.html  (title: "Vector Spell Lab", ~22 KB)
  role: procedural/vector spell VFX. Every effect is a pure function of progress p in [0,1]; no stored frames.
  spells: spellCircle (magic circle), spellBeam, spellLightning (jagged bolt via jag()), spellNova. Plus sparks, a background, a simple wizard + staff placeholder, theme sets (arcane, frost, void) built from MK's palette.
  cost-relevant techniques currently used: ctx.shadowBlur (glow; up to ~30), globalCompositeOperation 'lighter' (additive), createRadialGradient glows. These are the hot spots to optimize for todo #4.
  DPR clamped to 2.

folders:
- images/: flat raster assets. mannequin.png, mannequin_with_accessories.png, mannequin_side.png, mannequin_side_with_hood.png, player.png (run/idle/jump/attack sheet), cloakTop.png, cloakBottom.png, spell.png (7 frames), swoosh.png (4 frames, 400x400 cells), puff.png, spawn.png, tiles.png (6 frames, 60x60), ground.png (125x80, tiled).
- colorPalette/: colors.json, colors.css
- json_files/: consumed by animatedRig.html
  - bone_lengths.json: bone lengths (thigh 173.2, shin 147, upperArm 96.3, forearm 50 px) measured from front-view reference mannequin on an 800x800 canvas, plus torso measurements and referenceJoints. Only lengths carry to the side view, not joint positions. Note: forearm is 50 here vs 79.7 in the embedded fallback (BONES_FALLBACK). This is a known, intentional lag: MK edited the JSON and has not synced the HTML fallback yet. Do not "fix" it unprompted (see todo #6). Live bones are overwritten by art-fit lengths from pivots anyway.
  - gait_angles.json: side-view joint angle RANGES [min,max] per pose for run and walk (thigh, shin, upperArm, forearm per left/right limb). Convention: positive forward, measured from vertical-down from parent joint. Shin = thigh minus knee bend. Notes embedded in file give knee/elbow bend guidance. Stylized, not mocap.
  - rig_config.json: exported slider state ("Rig Lab slider defaults for one character"): shared.body/motion/poseSpeed/feet/art/view, cycles.run/walk params (spineAngle, chestAngle, neckAngle, cycleSec, airtime, twist, hipTwist, headBob, headJut, ssAmt, stride, strideBoot, twistPhase, liftPhase), footPresets (boot/bare), squashStretch, footAngles (bare/boot x run/walk), pivots. Written by the Export button; do not hand-edit unless asked, or MK's next export overwrites it.
- old_versions/: pre-git snapshots (animatedRig.html, animatedRig_before_stomach.html, rig_config_before_stomach.json). IGNORE by default. Slated for deletion.

independence: each HTML file stands alone. Only shared resources are images/, colorPalette/, json_files/ (animatedRig only).
</repo_structure>

<architecture_notes>
- No shared module layer. Duplication across files is accepted. Do not propose extracting shared libs unless asked.
- Keep single-file structure. Prefer minimal diffs, scoped to the file in question. Ask before large restructures.
- Data-driven where possible: tuning values in top-of-file consts or JSON, not buried in logic.
- animatedRig.html is a large dense single file written with compact style (many one-line functions). Preserve that style when editing it; do not reformat.
- Tools persist state in two ways: JSON export/import (rig_config.json) and localStorage (pivots). Remember localStorage can mask changed defaults when testing.
</architecture_notes>

<todo priority="unordered">
1. Fine-tune animatedRig.html (walk/run quality, timing, joint behavior).
2. Cloak animation experiments (verlet cloth and/or spring physics). Prior art: index.html cape modes 1 and 2. Reuse the algorithms, decouple from Kaplay.
3. Add hood and cloak to animatedRig.html. Art starting points: images/mannequin_side_with_hood.png, cloakTop.png, cloakBottom.png. Will need new pivots (e.g. collar/neck attach), a layer order decision (cloak behind vs in front of body), and a driver (spring/verlet fed by pelvis/chest motion).
4. Hybrid vector + sprite magic spells to cut cost, especially blur/glow. Hot spots in vectors.html: shadowBlur, many radial gradients, additive 'lighter' draws. Ideas: pre-render glow/gradient to offscreen sprites and draw scaled, additive blend, cached layers, fewer shadowBlur calls, lower-res glow buffers.
5. (Implied) Decide engine: Kaplay -> plain JS vs Godot. Not yet decided.
6. LATER, discuss with MK: are the embedded fallbacks in animatedRig.html (BONES_FALLBACK, gaitFallback(), embedded base64 art sheet, embedded rig defaults) still necessary, or can they be removed so the JSON files in json_files/ are the single source of truth? Tradeoff: fallbacks let the file work over file:// with no server; removing them means duplicated data goes away and cannot drift, but a local server (or a loader fix) is required. Until discussed, leave fallbacks alone and do not sync them unless asked.
</todo>

<open_questions>
- Final art style, sprite resolution, character proportions: undecided.
- Engine for the real game: Kaplay is temporary. Plain JS vs Godot undecided (leaning Godot).
- Runtime-procedural animation vs baked frames in the final game: undecided. The rig tools lean procedural; index.html uses baked frames in player.png.
- Whether staticRig.html is still wanted (stale asset paths).
</open_questions>

<working_conventions>
- Read the target file before modifying; do not rely on this doc for implementation details.
- If code contradicts this doc, trust the code and point out the discrepancy.
- Perf matters: the real game needs many simultaneous effects. Flag expensive ops (blur, shadowBlur, per-frame allocations, big offscreen canvases, per-particle objects).
- Colors: pull from colorPalette/ rather than inventing new ones.
- Prefer engine-agnostic algorithms (see engine_direction).
- Ignore old_versions/ unless told otherwise.
- Visual/UI output defaults to MK's sci-fi/cyberpunk aesthetic.
</working_conventions>

</repo_context>
