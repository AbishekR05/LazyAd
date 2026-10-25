# LazyAd — Phase 1 Spike Results & Benchmarks

| Milestone | Target | Status | Assigned GPU | Result Summary |
| :--- | :--- | :--- | :--- | :--- |
| **Spike S1** | Colab Runtime Diagnostic | ✅ **PASS** | Tesla T4 (14.56 GB) | GPU verified; OS: Ubuntu 24.04, Python 3.13, CUDA 13.0, PyTorch 2.11. FFmpeg present. |
| **Spike S2** | User-Supplied Product Image Input | ✅ **PASS** | CPU / Colab | Validated 2752x1536 RGBA PNG (5.89 MB) with alpha transparency. Staged at `/content/lazyad_assets/product_input.png`. |
| **Spike S3** | Baseline 3D Reconstruction (TripoSR) | ⚠️ **TECHNICAL PASS / VISUAL REJECTION** | Tesla T4 (4.14 GB peak) | Valid GLB exported (53,571 verts, 6.05s recon). But output is a literal white ceramic brick/box with an embossed watch face. Inadequate for commercial video. |
| **Spike S4** | Headless Blender Programmatic Render | ⏳ *In Progress* | Tesla T4 | Automated camera orbit, studio softbox lighting, and EEVEE frame capture. |
| **Spike S5** | Audio Synthesis & Composition | ⏳ *Pending* | Tesla T4 | Kokoro-82M TTS + FFmpeg video/audio muxing. |

---

## Spike S1 Empirical Findings (Logged: 24 Oct 2026)

### Objective
Inspect and verify the Google Colab execution environment to confirm GPU capability, Python/CUDA versions, and disk space for hosting the pipeline.

### Execution & Measurements
- **GPU:** Tesla T4 (1 GPU)
  - VRAM: **14.56 GB** (15,637,086,208 bytes)
  - Compute Capability: **7.5**
  - CUDA: **13.0**, cuDNN: **92700**, PyTorch: **2.11.0+cu130**
- **System Memory:** Total RAM: **12.67 GB** (Available: 11.14 GB)
- **Disk Storage:** `/content` Total: **112.64 GB** (Free: **70.13 GB**)
- **OS & Tooling:**
  - OS: **Ubuntu 24.04.4 LTS** (Kernel 6.6.122+)
  - Python: **3.13.16**
  - Pre-installed system binaries: `ffmpeg`, `ffprobe`, `git`, `wget`, `curl`, `tar`, `gzip`.
  - Missing package identified: `trimesh` (pip installable).

---

## Spike S2 Empirical Findings (Logged: 24 Oct 2026)

### Objective
Stage and validate a high-resolution hero product image to serve as the visual anchor for 3D reconstruction and rendering.

### Attempt 1: Gemini API Nano Banana (`gemini-3.1-flash-image`)
- **Status:** **BLOCKED / PERMISSION_DENIED (403)**
- **Reported Error:** `403 PERMISSION_DENIED: Your project has been denied access.`
- **Root Cause:** Google AI Studio image-generation models require project allowlisting or enterprise billing accounts.
- **Architectural Pivot:** Dropped autonomous image generation. Adopted the user-supplied product image workflow (matching real-world e-commerce usage where users upload photos of their own inventory).

### Attempt 2: User-Supplied Product Image Input (Adopted Architecture)
- **Status:** ✅ **PASS**
- **Input File:** `Gemini_Generated_Image_bhz6u6bhz6u6bhz6.png` (5,894.59 KB)
- **Resolution:** **2752 x 1536 px**
- **Color Mode:** **RGBA** (Alpha / Transparency: `True`)
- **Aspect Ratio:** **1.792:1**
- **Output Asset:** Staged at `/content/lazyad_assets/product_input.png`
- **Assessment:**
  - Clean watch silhouette with alpha transparency.
  - Ultra-high resolution preserves crisp dial typography, gold casing, and emerald bezel details.

---

## Spike S3 Empirical Findings (Logged: 24-25 Oct 2026)

### Objective
Empirically test whether the user-supplied 2D product image can be reconstructed into a usable 3D mesh (`.glb`) using the TripoSR baseline on the Tesla T4 GPU.

### 1. Installation & Compatibility Engineering
Running TripoSR on Python 3.13 and PyTorch 2.11 required resolving multiple environment blockers:
- **`torchmcubes` Source Build:** Compiled cleanly in ~4 minutes against PyTorch 2.11 / Python 3.13.
- **`rembg` / `onnxruntime` Conflict:** TripoSR’s `tsr/utils.py` contains a hard `import rembg`. When `onnxruntime` was missing, it triggered `sys.exit(1)`. Neutralized using an in-memory stub (`sys.modules['rembg'] = MagicMock()`) since the S2 input already has an alpha mask.
- **Hugging Face Weight Resolution:** Stability AI repository hosts weights as `model.ckpt` (1.68 GB), not `model.safetensors` (404 error resolved).
- **Transformers 4.49+ Architecture Breaking Change:** Hugging Face recently refactored `ViTModel` keys from `encoder.layer.X.attention.attention.query` to `layers.X.attention.q_proj`. An in-memory key adapter mapped the checkpoint tensors dynamically without downgrading Python or PyTorch.
- **OmegaConf Interpolation:** Required `OmegaConf.resolve(cfg)` before instantiating `TSR(cfg)` to evaluate `${tokenizer.num_channels}`.

### 2. Empirical Execution Benchmarks
- **Model Checkpoint Download:** 1.68 GB in ~15.2s (Colab network ~88 MB/s).
- **Neural Inference Time:** **3.13 – 3.22 seconds**
- **Marching Cubes Mesh Extraction Time (Grid 256):** **2.83 seconds**
- **Total Reconstruction Latency:** **6.05 seconds**
- **Peak GPU VRAM Allocated:** **4.14 GB** (28.4% of Tesla T4 capacity; zero OOM risk).
- **Exported File:** `/content/lazyad_assets/watch_reconstruction.glb` (2.01 MB).

### 3. Mesh Integrity & Geometry Statistics
- **Vertices:** **53,571**
- **Triangle Faces:** **106,944**
- **Bounding Box Extents:** `X = 0.413`, `Y = 1.013`, `Z = 0.566`
- **Finite Geometry:** True
- **Vertex Color Channels:** True (RGB per vertex)

### 4. Visual Inspection & The "Ceramic Brick" Output Defect
When the exported `.glb` was opened in a 3D viewer, the visual output was inspected:
- **The Visual Reality:** The output is **literally a giant white rectangular block/brick with the watch face stamped onto the front like a wax seal**.
- **Root Cause Analysis:**
  1. **Marching Cubes Boundary Shell:** TripoSR predicts density across a normalized 3D grid volume $[-0.5, 0.5]^3$. Because the background pixels retained slight non-zero density values, Marching Cubes wrapped an outer shell around the entire sampling volume, forming a hollow box.
  2. **Single-Image Blind Spot:** A single 2D photo contains zero information about the back, thickness, or crown profile. TripoSR did not reconstruct a 360° watch; it extruded a 2.5D bas-relief stamp forward from the flat background plane.
  3. **Mesh Merging:** The watch bezel is geometrically connected to the outer boundary walls by intermediate density faces, making simple programmatic separation destructive.
- **User Verdict:** **"it was shite"** — Completely unusable for a commercial advertisement.

### 5. Architectural Conclusions & Pivot for User-Input Ad Generation
- **TripoSR Verdict:** **REJECTED FOR COMMERCIAL PRODUCT GENERATION.** While technically fast (6s) and memory-light (4.14 GB), its single-image geometry is too distorted and generates severe volumetric artifacts.
- **How to faithfully handle the user's uploaded product:**
  - Generating random stock 3D models defeats the user's intent to advertise *their own* product.
  - To preserve 100% of the user's original photograph (crisp dials, metallic reflections, legible branding) with zero AI hallucination or white boxes, the pipeline pivots to **2.5D Metric Depth Parallax (Depth-Anything-V2 + Blender Camera Choreography)**:
    - User's 4K photo is projected onto a high-density displaced mesh in Blender using a metric depth map.
    - Blender's automated camera executes smooth $\pm 30^\circ$ orbits, dolly-in, and macro pans.
    - Result: Genuine 3D parallax and lighting reflections on the user's *exact* watch without deformed geometry or melted backplates.

---

## Spike S4 Empirical Findings (In Progress)

- **Target Engine:** Headless Blender 3.6 LTS (Linux x64) via `blender -b -P render_script.py`
- **Asset Testing:** Khronos Group `ChronographWatch.glb` (testing studio lighting, camera keyframes, and EEVEE renderer).
- **Resolved Issues:** 
  - Blender 3.6 glTF importer threw `TypeError: 'NoneType' object is not subscriptable` due to unhandled `KHR_materials_variants` extension mappings.
  - Implemented automated binary GLB sanitizer in Step 1 to strip variant tags before import.
- **Current Run:** Executing 90-frame EEVEE render benchmark (1080x1080 @ 30fps).
