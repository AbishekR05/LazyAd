# LazyAd — Phase 1, Spike S4: Headless Blender Programmatic Render

Use the approved `PHASE 0 FEASIBILITY & REQUIREMENTS.md` v2.2 and the empirical results from Spikes S1, S2, and S3 as the source of truth.

Implement **Spike S4 only** in:

`notebooks/S4_headless_blender_render.ipynb`

## Objective

Empirically test whether a headless portable Linux Blender instance running in Google Colab (Tesla T4 GPU, Ubuntu 24.04 LTS) can programmatically:
1. Automatically download and import a photorealistic 3D luxury watch asset (`ChronographWatch.glb` from official Khronos glTF-Sample-Assets, CC-BY 4.0).
2. Set up a professional studio environment (three-point softbox studio lighting, neutral ground plane).
3. Execute an automated multi-shot cinematic camera orbit (reveal dolly, 35° orbit, dial macro close-up).
4. Render 90 frames (3 seconds @ 30 fps, 1080x1080 or 1280x720) using the EEVEE engine directly to disk (`/content/rendered_frames/frame_####.png`).
5. Measure render latency per frame, peak VRAM consumption, and display inline video/frame previews.

The output must be completely programmatic — **zero manual Blender UI interaction**.

## Input Asset

- **Photorealistic Watch Model:**
  Official Khronos Group `ChronographWatch.glb`
  - URL: `https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Assets/main/Models/ChronographWatch/glTF-Binary/ChronographWatch.glb`
  - Features: Multi-material PBR (metallic bracelet, polished bezel, sapphire glass transmission, detailed chronometer subdials).
  - License: CC BY 4.0 (Commercial & personal use permitted).
  - Staged automatically to: `/content/lazyad_assets/photorealistic_watch.glb`

## Headless Blender Setup

- **Portable Binary:** Blender 3.6 LTS Linux x64
  - URL: `https://download.blender.org/release/Blender3.6/blender-3.6.0-linux-x64.tar.xz`
  - Extraction Target: `/content/blender-3.6.0-linux-x64/`
  - Execution Pattern:
    ```bash
    !/content/blender-3.6.0-linux-x64/blender -b -P render_watch_ad.py
    ```

## Automated Blender Script (`render_watch_ad.py`)

The script must execute the following procedures:
1. **Scene Initialization:** Purge default cube, camera, and point light.
2. **GLTF Import:** Import `/content/lazyad_assets/photorealistic_watch.glb`, center object at origin `(0, 0, 0)`, and auto-scale to bounding extent ~1.0 unit.
3. **Studio Lighting (Three-Point Softbox Setup):**
   - **Key Light:** Warm-white area light (Power ~300W, angled at 45° from front-right).
   - **Fill Light:** Soft cool-white area light (Power ~120W, angled from front-left).
   - **Rim/Back Light:** Crisp highlight area light (Power ~250W, angled from high-rear to highlight watch chamfers and bezel).
   - **Studio Cyc Ground:** Subtle dark studio backdrop (`roughness=0.3`, metallic=0.0) catching soft contact shadows.
4. **Cinematic Camera Choreography (90 frames @ 30fps):**
   - **Frames 1–30 (Shot 1: Reveal Dolly-In):** Camera moves from distance `D = 2.4` to `D = 1.6`, focusing directly on the watch face.
   - **Frames 31–60 (Shot 2: Three-Quarter Hero Orbit):** Smooth orbital sweep from azimuth `-25°` to `+25°` around the front hemisphere.
   - **Frames 61–90 (Shot 3: Macro Close-up Pan):** Low-angle macro pan catching metallic specular glints across the dial markers and winding crown.
5. **Render Settings:**
   - Engine: `BLENDER_EEVEE`
   - Samples: `32` – `64` (optimized for rapid T4 GPU execution)
   - Resolution: `1080 x 1080` (Square format for social media ad) or `1280 x 720`
   - Output Path: `/content/rendered_frames/frame_####.png`

## Deliverables & Acceptance Criteria

Spike S4 passes only if:
1. Blender 3.6 portable binary downloads and runs cleanly in headless mode (`-b`).
2. `ChronographWatch.glb` downloads automatically and imports without missing textures.
3. EEVEE renders all 90 frames without crashing, GPU hang, or CUDA OOM.
4. Render timing per frame and total render time are measured and recorded.
5. At least 3 sample rendered frames are displayed in the notebook.
6. The frames are verified to show crisp metallic reflections, transparent sapphire glass, and dynamic camera motion.

## Strict Scope Limits

- Do not implement Spike S5 (TTS audio synthesis or FFmpeg muxing).
- Do not create the final monolithic `LazyAd_MVP.ipynb`.
- Keep S4 strictly isolated to headless Blender rendering, camera choreography, and visual validation.
