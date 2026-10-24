# LazyAd — Phase 1 Spike Results & Benchmarks

| Milestone | Target | Status | Assigned GPU | Result Summary |
| :--- | :--- | :--- | :--- | :--- |
| **Spike S1** | Colab Runtime Diagnostic | ✅ **PASS** | Tesla T4 (14.56 GB) | GPU verified; OS: Ubuntu 24.04, Python 3.13, CUDA 13.0, PyTorch 2.11. FFmpeg present. |
| **Spike S2** | User-Supplied Product Image Input | ✅ **PASS** | CPU / Colab | Validated 2752x1536 RGBA PNG (5.89 MB) with alpha transparency. Staged at `/content/lazyad_assets/product_input.png`. |
| **Spike S3** | Baseline 3D Reconstruction (TripoSR) | ⏳ *Pending* | Tesla T4 | Single-image GLB generation with TripoSR. |
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
