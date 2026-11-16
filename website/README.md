# PERA — Skills as Plug-ins.

English research website built with Vite, React and plain CSS. Website code, media rendering and evidence exports are independent of training code. See [the source audit](docs/ASSET-AUDIT.md) and [verification notes](docs/VALIDATION.md).

## Run locally

Requires Node 22+ and npm.

```bash
cd website
npm ci
npm run dev
```

Open the printed local URL. To build and inspect production output:

```bash
npm run build
npm run preview
```

`dist/` is a complete static site, including local videos, posters, provenance JSON, data and the downloadable SVG. Default relative URLs support repository subpaths. To set an explicit Pages path:

```bash
PAGES_BASE_PATH=/pera-project/ npm run build
```

## Content and resource updates

- `src/content.js`: project title, release URLs and skill definitions. Keep unavailable author/citation fields null until verified. The Code link points to the public research repository, https://github.com/shangchaohu718/PERA. Add actual links only when public resources exist.
- `src/styles.css`: responsive design, typography, motion preferences.
- `public/method.svg`: downloadable method diagram; reflect the implemented route.
- `public/data/evidence.json`: compact source-backed experimental snapshot. Never enter estimated values or fabricated curves. Re-export from the project with `python website/scripts/export_evidence.py` from the parent repository root.
- `public/data/curve.csv`, `roundhouse.csv`: exact plotted data downloads.
- `public/media/<id>.mp4`, `.webp`, `.json`: clip, poster and provenance. Update the skill configuration and manifest together. Videos should be H.264/yuv420p MP4 with faststart and unchanged playback speed.

The hero uses **31 seconds of four independent native-scene recordings**: skateboard (5 s) → rough terrain and high steps (5 s) → MimicX stair ascent (6 s) → ATHLETE tennis (15 s). Each action keeps its own original scene and actual recorded robot/object states. The film uses three direct cuts, original speed and no animated joins, scene relocation or prop edits. Only the native tennis task contains a racket. The same frozen public Actor is shared across task-specific experts/adapters; the film is not one continuous episode. Full recordings are linked separately in the expanded player. See [selection and provenance](docs/HIGHLIGHTS.md).

## Reproduce media

Run in the parent robotics checkout, using the existing environment (MuJoCo, NumPy, Pillow, ffmpeg); no training environment installation is required:

```bash
.venvs/instinct-supermimic/bin/python website/scripts/render_gallery.py --clip roundhouse --preview
.venvs/instinct-supermimic/bin/python website/scripts/render_gallery.py
.venvs/instinct-supermimic/bin/python website/scripts/render_terrain.py --preview
.venvs/instinct-supermimic/bin/python website/scripts/render_terrain.py
```

`highlights.json` controls the eight-skill experimental gallery and archived montage; `render_highlights.py` exports complete recordings, short excerpts and the montage. `studio_lighting.py` provides bright robot lighting with distinct studio, terrain, skate-ground and tennis-court presentation profiles. `render_config.json` controls the earlier baseline/comparison clip sources. The rendering scripts read actual recorded `robot_*` states / `qpos`; they never use `ref_*` as robot poses or query a policy. The paired kick camera follows the same reference trajectory in a translation-aligned world frame. `ref_*` is used only for the camera target. Terrain geometry is preserved. Sources and model bytes are not modified. These experimental-gallery renderers select recorded poses without interpolation. The separate `render_scene_preview.py` intentionally interpolates poses and authors transitions for the clearly labeled design preview.

Web files are 1920×1080 at 30 fps. Lossless FFV1 masters remain in `.render-work/` (ignored by Git), and original NPZ/video sources remain in `reports/`. The scripts refuse to overwrite an existing master or MP4: move previous generated outputs into a new archival directory before rendering again. Preview only replaces the generated poster; run it before production rendering.

To compress another independently rendered master:

```bash
ffmpeg -i input.master.mkv -an -c:v libx264 -preset slow -crf 22 \
  -pix_fmt yuv420p -movflags +faststart output.mp4
ffmpeg -ss 1 -i output.mp4 -frames:v 1 -quality 92 output.webp
```

Never alter speed or trim failed portions to imply successful execution. Caption excerpts and separate baseline/public/expert results clearly. After any media/data edits:

```bash
.venvs/instinct-supermimic/bin/python website/scripts/export_figures.py
python website/scripts/verify_assets.py
npm --prefix website run build
```

Every gallery skill now has a real ON/OFF switch at `#expert-comparison`. OFF plays the frozen HoloMotion public policy on that task. Fresh standalone paired recordings were added for roll, vault, skate, gaps and high steps, with identical initial qpos verified for each pair. Tennis uses the original ATHLETE launch protocol with the tennis plug-in bypassed for OFF. Moonwalk uses the recorded stage_500 public control. The earlier quantified roundhouse data remain separate from the refined gallery kick; there is only one interactive expert switch in the gallery. `public/data/comparisons.json` indexes every source, duration and complete recording.

Tennis uses a close following camera (distance 3.2 m), with no wide court inset. Both ON and OFF play 20 continuous seconds; full source recordings remain available. Moonwalk uses the selected model_499 (500 PPO iterations / 12,000 Actor Adam updates), including a six-second excerpt and full 17.8-second recording. `public/data/moonwalk.json` contains the exact single-seed metrics.

Failed roll/vault/gaps/steps OFF replays now include four seconds of real closed-loop continuation after the native failure, solely to show the fall and aftermath without resetting. Native scoring is unchanged. See `record_failure_continuations.py` and the v6 provenance. Comparison players and the hero highlights hold the last frame and offer Replay. The explanation before the sole gallery switch identifies the shared frozen public Actor and task-specific experts/adapters.

There are 48 web MP4s including the native-scene highlights, the complete native stair recording and all complete comparison sources. Media are lazy-loaded; only visible replay clips load automatically. The homepage is an edited four-scene highlight film. The earlier eight-skill montage remains available as `showreel.mp4`; the animated six-station previs is archived locally and is no longer served as the hero.

## GitHub Pages

Public URL: https://shangchaohu718.github.io/pera-project/

Editable source is on `main` in `shangchaohu718/pera-project`. GitHub Pages serves
prebuilt files from `gh-pages`; the training repository keeps its existing access.
The authenticated GitHub account currently has `repo` access without the
`workflow` scope, so publication uses the static branch rather than Actions.

After committing and pushing source changes to `main`, publish from this checkout:

```bash
cd website
npm run deploy
```

Requires Node 22, npm, Git and an authenticated GitHub CLI (`gh`) with repository
write access. The command builds with `/pera-project/` as its base path, stages
only `dist/` using a temporary Git index, pushes the static `gh-pages` branch and
requests a Pages build. It preserves the editable source branch and all media.
The original Actions workflow is retained as an optional template at
`docs/github-pages-actions.yml.example`; enabling it later requires appropriate
workflow authorization and changing the Pages source to Actions.

Web MP4s remain ordinary Git files as configured by `.gitattributes`. Local
`qa/`, `.render-work/`, `node_modules/`, ZIP exports and `dist/` are ignored.

The tennis demo is a fresh 40-second rollout of the existing frozen round_10 student, seed 97001, with zero teacher queries/actions and no training. `record_tennis.py` records the original simulation; `tennis_hands.py` restores only the upstream commented visual meshes in a separate replay model. `public/data/tennis-recording.json` and `tennis-hands.json` contain the audit.

## Attribution

G1 geometry comes from the actual local Unitree asset supplied with HoloMotion. Public-policy behavior is attributed to HoloMotion G1 v1.4. The climbing expert is distilled from the official InstinctMJ teacher. Tennis uses the original ATHLETE court, physics, ball launches and robot assets, with visual hands restored only in replay. Local snapshots and their limitations are listed in the site. Third-party license notices are preserved in `licenses/`; this website does not assign a new license to the underlying robot, motion data or model assets. No reference-site assets, AI-generated robot experiments or fabricated plots are used.

## Hero presentation — 2026-11-06

Paper and Code now appear directly below the main title/description. Paper remains Coming soon at the user's request; the local v2.2 draft is not published. Code points to the existing private GitHub repository, with its access status shown. No repository permission or visibility changes were made.

The hero has no Unitree specification badges, per-action annotations, duration counters or burned-in action labels. `render_highlights.py --montage-only` now encodes label-free excerpts from the actual lossless masters before concatenation; the labeled comparison-gallery clips stay intact. The current hero is explicitly labeled Recorded highlights. Its complete, independently audited physical source is linked separately.


The superseded design previsualization is archived in `.render-work/scene-preview-v1/`, including the original seven motion stages, six animated joins and source provenance. It is not the current homepage media.

## Current four native scenes

The homepage now uses separate native recordings at the user’s request. `scene-preview.mp4` retains its filename but now contains 31 seconds / 930 frames of independent task excerpts. `public/data/hero-highlights.json` records each source, native scene, report hash, interval and full-recording link. The prior shared-course highlight and complete course source are retained locally in `.render-work/native-four-scenes-v2/previous-hero/` and `retired-unified-course-public/`; they are no longer homepage sources.

`native_scene_highlights.py` reuses the original lossless skateboard, high-step and close-up tennis masters. The stair recording is `reports/mimicx_stairs_512_v1/run_20261106_083527/training/final_student_4004_trajectory.npz`, with its native fifteen-step scene. The selected stair interval is 2.8–8.8 seconds; its complete recorded trajectory is linked as `native-stairs-full.mp4`. The source MuJoCo 3.5 scene was exported with its original mesh assets into `reports/continuous_four_scene_v1/native_stairs_probe_02/scene-export/` for the compatible visual replay. Native and replay body positions agree exactly on the four checked source states. No simulation stepping, training or teacher queries occur during rendering.

```bash
.venvs/instinct-supermimic/bin/python website/scripts/native_scene_highlights.py --output <new-render-directory> --preview
.venvs/instinct-supermimic/bin/python website/scripts/native_scene_highlights.py --output <new-render-directory>
```

The native scenes keep their terrain, skateboard, court, ball and racket coordinates. Only camera, lighting and presentation materials change. The tennis excerpt plays its first 15 continuous seconds, including three incoming-ball cycles and an unsuccessful return. This is selected footage, not a success-rate measurement or an opponent rally. The gallery’s eight ON/OFF comparisons and prior quantitative evidence remain unchanged.

## Complete skill gallery — 2026-11-06

The interactive gallery now uses the 31-task roster in `public/data/expert-catalog.json`, with category filters, name search and one shared expert switch. Skill names and ON/OFF are the presentation labels; source brands and training-method labels are omitted from the page. The four-scene homepage film remains independent of this catalogue.

`src/ExpertGallery.jsx` renders the catalogue. New media live in `public/media/catalog31/`; the earlier media remain available to the homepage and evidence section. Original recording metadata and the source audit are retained for integrity checks. Historical batch metrics are explicitly separate from the illustrative videos.

Export with `python website/scripts/build_expert_catalog.py --existing`; replay newly captured actual states with `bash scripts/run_flex_supermimic.sh website/scripts/build_expert_catalog.py --native`. Recording helpers under `website/scripts/record_catalog_*.py` only run existing policies, verify frozen weights and preserve native outcomes; no training occurs.

Validate the complete gallery with `node website/scripts/check_catalog.mjs`, then run `npm --prefix website run build` and `npm --prefix website run portable`. Browser interaction requires a separately available browser-control surface; server rendering and all media/provenance are checked independently.

The public project name is **PERA**. The legacy workflow filename `pear-pages.yml` is retained for deployment compatibility; its visible Actions title uses PERA.

White-stage replacements live in `public/media/catalog31-white-v2/`, using new URLs so cached dark recordings are not reused. `render_catalog_white.py` replays archived native root/joint/object states at original speed, with white ground/sky and pale blue terrain; robot and object colors remain intact. Native MuJoCo 3.5 binary scenes require `.venvs/mimicx/bin/python`. Empty sky is set to white using the renderer's exact object-ID buffer, preserving every geometry pixel. Original dark videos remain archived and are no longer selected where a white replacement exists.

`record_catalog_pmt_states.py` captures the two existing policy modes at the same seed for replay of the native terrain mesh and actual articulation states. It does not use reference poses for the robot, query teacher actions, update weights or include reset frames. Its outputs and source hashes are retained in `.render-work/catalog31-white-v2/` and the public media sidecars.

## Curated display — 2026-11-06

The gallery now displays 20 tasks: 19 from the original roster after excluding all five wall variants and the seven trailing MimicX tasks, plus the existing continuous stair-ascent task. Exclusions are enforced by `build_expert_catalog.py`; original research files are retained. Continuous stairs uses the existing matched seed-4004 ON/OFF trajectories and the same frozen public digest, with native fifteen-step geometry intact.

The task selector is a single horizontal strip on desktop and mobile, with arrow buttons, native touch scrolling, a scrollbar and keyboard navigation. Selecting a skill scrolls only the strip. The overview replaces repeated text and the abstract box diagram with the main conversation's unmodified v14 concept figure, copied as the self-contained `public/concept-one-policy.svg`, followed by one sentence.

`render_catalog_white.py --task pmt_optimized_scene0 --bright` produces `public/media/gallery-bright-v3/`. A second native render at the exact same actual state increases camera fill on robot pixels selected by the renderer's geometry-ID buffer. Terrain and sky retain the standard render. Robot poses, materials, source trajectories, timing and task outcomes are unchanged. The continuous stair pair is rendered with `--task mimicx_stairs` and standard studio lighting.

## Paper figures and simplified page — 2026-11-13

The main page now contains the hero, the one-policy concept, the 20-task interactive gallery and two original paper figures. The previous Method, experimental-snapshot Results and bottom Resources sections are removed from the rendered page. Navigation points to Overview, Skills and Results; Paper (Coming soon) and Code stay in the hero. Historical evidence/media files remain available for provenance but are not used as the new results section.

`src/ResearchFigures.jsx` presents Figures 3 and 5 from the desktop `PERA.pdf` (2026-11-11 manuscript). Each image is an unchanged embedded PNG extracted from the PDF, rather than a recreated chart. Figure 3 covers unseen fast-dance transfer; Figure 5 compares the five tasks where both HoloMotion + expert and the task teacher complete all 16 episodes. Captions retain the sample sizes and the restricted comparison scope. Images open at full resolution; small screens support horizontal scrolling within each figure without overflowing the page.

Original images and source SHA-256 are in `public/figures/paper/` and `provenance.json`. The manuscript itself is not copied to the served website. To reproduce the import with pypdf installed:

```bash
python website/scripts/import_paper_figures.py /path/to/PERA.pdf
npm --prefix website run build
npm --prefix website run portable
```

The v2026-11-06 desktop ZIP is an older snapshot; the current `website/` source, `dist/` and `OPEN-WEBSITE.html` include these revisions.
