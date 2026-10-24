# LazyAd — Phase 1 Spike Results & Benchmarks

| Milestone | Target | Status | Assigned GPU | Result Summary |
| :--- | :--- | :--- | :--- | :--- |
| **Spike S1** | Colab Runtime Diagnostic | ✅ **PASS** | Tesla T4 (14.56 GB) | GPU verified; OS: Ubuntu 24.04, Python 3.13, CUDA 13.0, PyTorch 2.11. FFmpeg present. |
| **Spike S2** | User-Supplied Product Image Input | ✅ **PASS** | CPU / Colab | Validated 2752x1536 RGBA PNG (5.89 MB) with alpha transparency. Staged at `/content/lazyad_assets/product_input.png`. |
| **Spike S3** | Baseline 3D Reconstruction (TripoSR) | ⚠️ **TECHNICAL PASS / VISUAL REJECTION** | Tesla T4 (4.14 GB peak) | Valid GLB exported (53,571 verts, 6.05s recon). But visual quality is low: severe boundary cube extrusion and 2.5D bas-relief geometry. |
| **Spike S4** | Headless Blender Programmatic Render | ⏳ *Pending* | Tesla T4 | Automated camera orbit and EEVEE frame capture. |
| **Spike S5** | Audio Synthesis & Composition | ⏳ *Pending* | Tesla T4 | Kokoro-82M TTS + FFmpeg video/audio muxing. |

---

## Spike S1 Empirical Findings (Logged: 24 Oct 2026)

- **GPU:** Tesla T4 (1 GPU)
  - VRAM: **14.56 GB** (15,637,086,208 bytes)
  - Compute Capability: **7.5**
  - CUDA: **13.0**, cuDNN: **92700**, PyTorch: **2.11.0+cu130**
- **System Memory:**
  - Total RAM: **12.67 GB** (Available: 11.14 GB)
- **Disk Storage:**
  - `/content` Total: **112.64 GB** (Free: **70.13 GB**)
- **OS & Tooling:**
  - OS: **Ubuntu 24.04.4 LTS** (Kernel 6.6.122+)
  - Python: **3.13.16**
  - System binaries verified: `ffmpeg`, `ffprobe`, `git`, `wget`, `curl`, `tar`, `gzip` all pre-installed in `/usr/bin/`.
  - Python libraries verified: `torch`, `torchvision`, `diffusers` (0.41.0), `transformers` (5.20.2), `soundfile` (0.14.0), `google.genai` pre-installed. `trimesh` not pre-installed (must be installed via pip in S3).

---

## Spike S2 Empirical Findings (Logged: 24 Oct 2026)

### Attempt 1: Gemini API Nano Banana (`gemini-3.1-flash-image`)
- **Status:** **BLOCKED / PERMISSION_DENIED (403)**
- **Reported Error:** `403 PERMISSION_DENIED: Your project has been denied access.`
- **Finding:** Google AI Studio image-generation endpoints require an attached billing account or project allowlisting.

### Attempt 2: User-Supplied Product Image Input (Adopted Architecture)
- **Status:** ✅ **PASS**
- **Uploaded File:** `Gemini_Generated_Image_bhz6u6bhz6u6bhz6.png` (5,894.59 KB)
- **Dimensions:** **2752 x 1536 px**
- **Color Mode:** **RGBA** (Alpha / Transparency: `True`)
- **Aspect Ratio:** **1.792:1**
- **Output Asset:** Staged at `/content/lazyad_assets/product_input.png`
- **Assessment:**
  - High resolution (2752x1536) provides exceptional input clarity for depth estimation.
  - Presence of alpha transparency means the watch silhouette is already isolated, which will significantly improve TripoSR's background removal and marching cubes segmentation in Spike S3.
  - Zero API keys or external billing required.

---

## Spike S3 Empirical Findings (Logged: 24 Oct 2026)

- **Target Model:** TripoSR Baseline (`stabilityai/TripoSR`)
- **Runtime Environment:** Tesla T4 GPU (14.56 GB), Python 3.13, PyTorch 2.11.0+cu130, CUDA 13.0
- **Status:** ⚠️ **TECHNICAL PIPELINE PASS / VISUAL REJECTION**

### 1. Compatibility & Installation Blocker Resolution
- **`torchmcubes` Compilation:** Successfully compiled from source against PyTorch 2.11 / Python 3.13 in ~4 minutes.
- **`rembg` & `onnxruntime` Conflict:** TripoSR's `tsr/utils.py` has an unconditional top-level `import rembg`. `rembg` calls `sys.exit(1)` when `onnxruntime` is missing. Neutralized with a mock stub (`sys.modules['rembg'] = MagicMock()`).
- **Hugging Face Weight Name:** Weights repository uses `weight_name="model.ckpt"` (1.68 GB), not `model.safetensors` or `model.fp16.safetensors`.
- **Transformers 4.49+ Key Mismatch:** Hugging Face refactored `ViTModel` keys from `encoder.layer.X.attention.attention.query` to `layers.X.attention.q_proj`. An in-memory state dictionary adapter mapped the checkpoint keys dynamically.
- **OmegaConf Interpolation:** Required `OmegaConf.resolve(cfg)` before `TSR(cfg)` instantiation to evaluate `${tokenizer.num_channels}`.

### 2. Empirical Execution Benchmarks
- **Model Checkpoint Download:** 1.68 GB in ~15.2s (Colab network ~88 MB/s).
- **Neural Inference Time:** **3.13 – 3.22 seconds**
- **Marching Cubes Mesh Extraction Time (Grid 256):** **2.83 seconds**
- **Total Reconstruction Latency:** **6.05 seconds**
- **Peak GPU VRAM Allocated:** **4.14 GB** (Well within Tesla T4's 14.56 GB envelope; 28.4% utilization).
- **Exported File:** `/content/lazyad_assets/watch_reconstruction.glb` (2.01 MB).

### 3. Mesh Integrity & Geometry Statistics
- **Vertices:** **53,571**
- **Triangle Faces:** **106,944**
- **Bounding Box Extents:** `X = 0.413`, `Y = 1.013`, `Z = 0.566`
- **Finite Geometry:** True
- **Vertex Color Channels:** True (RGB per vertex)

### 4. Visual Inspection & Quality Assessment (User Review)
- **Front & Three-Quarter Views (0° – 45°):** The watch face, emerald dial, gold bezel, and winding crown are recognizable in textured relief.
- **Side & Top Views (90°):** Severe limitations inherent to single-image feed-forward reconstruction:
  1. **Marching Cubes Bounding Box Extrusion:** TripoSR's NeRF predicts residual non-zero density at the $[-0.5, 0.5]^3$ grid boundaries, causing Marching Cubes to generate a hollow rectangular outer shell/box around the model.
  2. **2.5D Bas-Relief Geometry:** The model is not a true watertight 3D watch in the round; it is an extruded bas-relief shell protruding from a flat backplate.
- **User Verdict:** **"it was shite"** — Visual quality is inadequate for a premium luxury watch commercial requiring realistic 3D camera orbits.
- **Engineering Recommendation:** 
  - TripoSR has proven that the pipeline (Colab T4 + PyTorch + Marching Cubes + GLB export) runs fast (6s) and stable (4.14 GB).
  - However, for high-end product visualization, either:
    1. **TRELLIS / SF3D:** Benchmark a higher-fidelity 3D model (e.g. TRELLIS on A100 or Stable Fast 3D with UV unwrapping).
    2. **Blender Camera Trickery (Defensive Orbit):** Restrict camera strictly to front $\pm 30^\circ$ dolly and isolate the watch geometry with a tight foreground bounding mask in Blender.
    3. **Procedural Watch Model / Asset Swap:** Load a photorealistic 3D watch GLB into Blender while testing the automated camera choreography in Spike S4.
