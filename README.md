# LazyAd ⌚🎬

> **AI-Powered Product Advertisement Generator**  
> Turn a product prompt or description into a finished, polished MP4 advertisement automatically.

LazyAd automates the commercial video pipeline:
**Product description / prompt → product visual generation → 3D reconstruction → automated cinematic camera & scene rendering → script generation → voiceover → composited MP4 ad.**

---

## Current Status: Phase 1 (Notebook-First MVP)

The project is currently executing **Phase 1 Validation Spikes** on Google Colab:
- **Scope:** One fictional luxury watch.
- **Workflow:** Sequential proof-of-concept Jupyter notebooks executed on Google Colab GPU runtimes.
- **Specification:** See [Docs/PHASE 0 FEASIBILITY & REQUIREMENTS.md](Docs/PHASE%200%20FEASIBILITY%20&%20REQUIREMENTS.md) for full architecture, research findings, and technical validation gates.

### Phase 1 Sequential Spikes
1. **Spike S1:** Colab Diagnostic & Hardware Profiling
2. **Spike S2:** Hero Image Generation & Memory Purge (FLUX.1 [schnell] / SDXL-Lightning)
3. **Spike S3:** Baseline 3D Reconstruction (TripoSR)
4. **Spike S4:** Headless Blender Programmatic Render
5. **Spike S5:** Audio Synthesis (Kokoro-82M) & Final Video Composition (FFmpeg)

After S1 through S5 pass individually, the unified end-to-end notebook (`LazyAd_MVP.ipynb`) will be assembled.

---

## License

This project evaluates and integrates open-source tools and models (TripoSR, FLUX.1 [schnell], Kokoro-82M, Blender, FFmpeg) under their respective permissive licenses (MIT, Apache-2.0, GPL).
