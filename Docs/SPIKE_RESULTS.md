# LazyAd — Phase 1 Spike Results & Benchmarks

| Milestone | Target | Status | Assigned GPU | Result Summary |
| :--- | :--- | :--- | :--- | :--- |
| **Spike S1** | Colab Runtime Diagnostic | ✅ **PASS** | Tesla T4 (14.56 GB) | GPU verified; OS: Ubuntu 24.04, Python 3.13, CUDA 13.0, PyTorch 2.11. FFmpeg present. |
| **Spike S2** | Hero Image Generation & Memory Purge | ⏳ *Pending* | Tesla T4 | FLUX.1 [schnell] with CPU offload (SDXL-Lightning fallback). |
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

### Practical Engineering Impact for Subsequent Spikes:
1. **TripoSR Baseline 3D (S3):** 14.56 GB VRAM provides ample headroom for TripoSR's ~6 GB requirement.
2. **Hero Image Generation (S2):** 14.56 GB VRAM on a T4 means FLUX.1 [schnell] (12B parameters) must use sequential CPU offloading (`enable_model_cpu_offload()`). If FLUX encounters system RAM pressure on the 12.67 GB host RAM, `SDXL-Lightning` (~6 GB VRAM, pure GPU inference) is prepared as the fast fallback.
3. **Sequential Cleanup is Critical:** Because system RAM (12.67 GB) and VRAM (14.56 GB) are finite, deleting pipelines and calling `torch.cuda.empty_cache()` between S2 and S3 is strictly verified as necessary.
