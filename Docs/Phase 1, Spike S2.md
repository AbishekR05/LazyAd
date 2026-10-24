# LazyAd — Phase 1, Spike S2: Nano Banana Hero Image Generation

Use the approved `PHASE 0 FEASIBILITY & REQUIREMENTS.md` v2.2 and the completed S1 diagnostic results as the source of truth.

## Objective

Implement **Spike S2 only** in a separate Google Colab notebook:

`notebooks/S2_nano_banana_hero_image.ipynb`

S2 must empirically verify that Nano Banana, accessed through the Google Gemini API, can generate and save a suitable luxury-watch hero image for the later TripoSR reconstruction spike.

This spike replaces the planned FLUX.1 [schnell] workflow. Do not implement FLUX or SDXL in this notebook.

## Model selection

Use the current recommended Nano Banana image-generation model available through the Gemini API.

The expected candidate is:

`gemini-3.1-flash-image`

Before implementation, verify the current official Google Gemini API documentation and supported Python SDK interface. Do not assume that a model ID or SDK method from an older tutorial is still valid.

If the model is unavailable to the account or requires billing that has not been enabled, report the limitation and stop. Do not silently switch to another paid model or incur unapproved costs.

## API key handling

- Use a Gemini API key supplied by the user at runtime.
- Use a secure input mechanism such as `getpass`.
- Never hardcode the key.
- Never print or expose the key in notebook output.
- Do not save the key in the notebook or generated files.
- Do not use browser automation or manually interact with Google AI Studio.
- Use the API programmatically from Colab.

S1 reported that `google.genai` is already installed. Check the installed SDK version and install or update only what is necessary for the documented API call.

## Hero image requirements

Generate one fictional luxury wristwatch product image.

The image must be optimized for **single-image 3D reconstruction**, not merely for a finished advertisement.

Prompt requirements:

- One complete wristwatch, clearly visible.
- Three-quarter front view, with the dial, case, bezel, and strap visible.
- Distinct case silhouette and dial color.
- Realistic materials and reflections.
- Watch centered and occupying most of the image.
- Plain, neutral background with clear separation from the product.
- No hands, people, extra watches, packaging, text, logos, watermarks, or decorative objects.
- Avoid extreme perspective, excessive glare, and dramatic shadows that obscure geometry.
- Use a fictional brand/product; do not reproduce a real brand's logo.

Use a deterministic, editable prompt template so later stages can generate different fictional products without rewriting the notebook.

## Notebook workflow

1. Display the selected model ID and explain any relevant API quota or billing caveat.
2. Accept the API key securely.
3. Initialize the documented Gemini Python client.
4. Submit the hero-image prompt.
5. Extract the actual image output from the API response.
6. Save the image to:

   `/content/lazyad_assets/hero_watch.png`

7. Verify:
   - The file exists.
   - The file is nonempty.
   - The image can be opened.
   - The image dimensions and format are recorded.
8. Display the generated image inline.
9. Record the model ID, prompt, image dimensions, file size, and generation duration.
10. Provide a concise pass/fail report.

Do not fabricate a successful response or image if the API call fails.

## S2 acceptance criteria

S2 passes only if:

- The selected Nano Banana model is successfully called through the API.
- A valid image is returned.
- The image is saved to the required path.
- The saved image can be reopened and displayed.
- The image is suitable for the intended TripoSR input, based on visual inspection.
- Any API errors, quota restrictions, billing requirements, or image-quality limitations are documented.

A successful API call alone does not prove the image is suitable for 3D reconstruction.

## Strict scope limits

- Do not implement S3, S4, or S5.
- Do not install or load TripoSR.
- Do not generate a GLB.
- Do not render video.
- Do not create `LazyAd_MVP.ipynb`.
- Do not assume S2 has passed before the notebook is actually executed in Colab.
- Do not add unnecessary dependencies or download large local models.

## Deliverables

Create the notebook and provide:

- The exact Google Colab execution steps.
- Any required API setup instructions.
- The notebook's pass/fail criteria.
- A clear explanation of any cost or quota dependency.

Stop after S2. Wait for the actual generated image and execution results to be reviewed before proceeding to S3.
