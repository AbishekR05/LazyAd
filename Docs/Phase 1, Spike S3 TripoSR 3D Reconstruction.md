# LazyAd — Phase 1, Spike S3: TripoSR 3D Reconstruction

Use the approved `PHASE 0 FEASIBILITY & REQUIREMENTS.md` v2.2, S1 diagnostic results, and S2 image-input results as the source of truth.

Implement **Spike S3 only** in:

`notebooks/S3_triposr_reconstruction.ipynb`

## Objective

Empirically test whether the user-supplied product image can be reconstructed into a usable 3D mesh using the TripoSR baseline in the active Google Colab runtime.

The required output is a valid GLB model that can later be imported into Blender for scripted camera movement and rendering.

## Active runtime

S1 recorded:

- GPU: Tesla T4
- Reported VRAM: 14.56 GB
- System RAM: 12.67 GB
- OS: Ubuntu 24.04.4 LTS
- Python: 3.13.16
- PyTorch: 2.11.0+cu130
- CUDA: 13.0

Do not assume these versions are compatible with every TripoSR dependency. Inspect the current environment before installing anything.

## Input

Use the S2 asset:

`/content/lazyad_assets/product_input.png`

The S2 image is a 2752 × 1536 RGBA PNG with transparency.

At the start of S3:

1. Check whether the image exists.
2. If it does not exist, provide a clear upload fallback.
3. Preserve the original image unchanged.
4. Inspect the alpha channel and determine whether it provides a useful foreground mask.
5. Display the original image and the prepared reconstruction input.

## Installation and compatibility

Use the official TripoSR repository:

`https://github.com/VAST-AI-Research/TripoSR`

Before installation:

1. Inspect the repository's current requirements and setup instructions.
2. Inspect the active Python, PyTorch, CUDA, and GPU environment.
3. Identify dependencies that conflict with the existing Colab packages.
4. Avoid blindly reinstalling or downgrading PyTorch, CUDA, NumPy, or other core packages.
5. Do not use runtime CUDA compilation unless the official implementation genuinely requires it and the requirement is documented.

If the current Python 3.13 environment cannot run TripoSR reliably, report the exact compatibility blocker and stop. Do not silently create a fragile installation or change the entire Colab environment.

Do not switch to TRELLIS or another reconstruction model without approval.

## Image preprocessing

Use a transparent-background-aware preprocessing workflow.

- Preserve the original input.
- Use the alpha channel as a foreground mask if appropriate.
- Composite the foreground onto a neutral background compatible with TripoSR.
- Preserve the product's proportions.
- Center the product.
- Resize the foreground to a suitable ratio without cropping the watch.
- Display the final prepared image before inference.

Do not run a redundant background-removal model on an already isolated product unless testing shows it is necessary.

Document the selected foreground ratio and preprocessing decisions.

## Reconstruction

Load the official TripoSR pretrained model.

Run inference on the Tesla T4 GPU when available.

Use a configurable marching-cubes resolution. Start with a reasonable baseline such as 128 or 256, and document the selected value.

Use a configurable renderer chunk size to manage VRAM usage.

The notebook must:

1. Report model initialization time.
2. Report inference time.
3. Report mesh extraction time.
4. Report total reconstruction time.
5. Record peak GPU memory usage when available.
6. Handle CUDA out-of-memory errors clearly.
7. Avoid rerunning expensive stages automatically after failure.

Do not claim that the reconstructed watch is geometrically accurate merely because inference completed.

## GLB export and validation

Export the mesh to:

`/content/lazyad_assets/watch_reconstruction.glb`

Validate the output:

- The file exists and is nonempty.
- The GLB can be reopened using a suitable library such as `trimesh`.
- The scene contains valid geometry.
- Vertex and face counts are recorded.
- Bounding-box dimensions are recorded.
- Mesh bounds and geometry are finite.
- The model is displayed or previewed if the notebook environment supports it.

If the official TripoSR output requires a conversion step for GLB export, document and test that conversion.

Do not assume that a file extension change creates a valid GLB.

## S3 acceptance criteria

S3 passes only if:

1. TripoSR installs or loads successfully in the active Colab environment.
2. The prepared S2 image is accepted by the model.
3. Reconstruction completes without an unresolved runtime error.
4. A valid GLB is exported.
5. The GLB can be reopened and contains nonempty geometry.
6. The model is visually inspected.
7. Runtime, memory, and mesh statistics are recorded.
8. Any reconstruction artifacts or limitations are documented.

A successful export alone does not prove that the watch is suitable for the final advertisement. The visual quality must be assessed.

## Strict scope limits

- Do not implement S4 or S5.
- Do not install Blender.
- Do not generate a camera animation.
- Do not render an advertisement.
- Do not generate audio.
- Do not create `LazyAd_MVP.ipynb`.
- Do not assume S3 has passed before actual execution and validation.
- Do not silently switch reconstruction models.
- Do not overwrite the original S2 image.

## Deliverables

Create the notebook and provide:

- Exact Colab execution instructions.
- Installation and compatibility findings.
- The prepared image preview.
- The GLB output path.
- Reconstruction timing and memory measurements.
- Mesh validation results.
- A clear PASS, FAIL, or BLOCKED status.
- Any known visual reconstruction artifacts.

Stop after S3. Wait for the actual GLB and execution results to be reviewed before proceeding to S4.
