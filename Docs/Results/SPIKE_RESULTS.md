# LazyAd — Phase 1 Spike Results & Benchmarks

| Milestone | Target | Status | Assigned GPU | Result Summary |
| :--- | :--- | :--- | :--- | :--- |
| **Spike S1** | Colab Runtime Diagnostic | ✅ **PASS** | Tesla T4 (14.56 GB) | GPU verified; OS: Ubuntu 24.04, Python 3.13, CUDA 13.0, PyTorch 2.11. FFmpeg present. |
| **Spike S2** | Nano Banana Hero Image Generation | ❌ **BLOCKED (403)** | API (Cloud) | `gemini-3.1-flash-image` returned `403 PERMISSION_DENIED: Your project has been denied access`. |
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

- **Model Candidate Tested:** `gemini-3.1-flash-image` (Nano Banana 2 via `google-genai` SDK)
- **Status:** **BLOCKED / PERMISSION_DENIED (403)**
- **Reported Error:**
  `403 PERMISSION_DENIED: Your project has been denied access. Please contact support.`
- **Root Cause & Technical Assessment:**
  1. In Google AI Studio, image-generation endpoints require either an account with verified billing (pay-as-you-go) enabled in Google Cloud Console, or an approved/allowlisted project tier. Standard unbilled or freshly generated AI Studio API keys return `403 PERMISSION_DENIED` on multimodal image generation calls.
  2. This confirms the open risk flagged in Phase 0 (§7.3, §14.2, U-5): API-based image generation cannot be relied upon as a guaranteed zero-cost path without active billing.
- **Strict Scope Limit Triggered:**
  Per `Docs/Phase 1, Spike S2.md`: *"If the model is unavailable to the account or requires billing that has not been enabled, report the limitation and stop. Do not silently switch to another paid model or incur unapproved costs."*
  $\rightarrow$ S2 is halted pending user review.
