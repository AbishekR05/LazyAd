# LazyAd — Phase 1, Spike S2: User-Supplied Product Image Input

Use the approved `PHASE 0 FEASIBILITY & REQUIREMENTS.md` v2.2 and the completed S1 diagnostic results as the source of truth.

The LazyAd MVP accepts a **user-supplied product image** rather than relying on external image generation models or third-party APIs.

## Objective

Verify that a user-supplied product image can be safely loaded, validated, previewed, and prepared as a standard input for the later TripoSR reconstruction spike (Spike S3).

This is an **image-input validation stage**, not an image-generation stage.

## Input workflow

Supports two input methods:
1. Upload an image directly through Google Colab using `files.upload()`.
2. Optionally select an image from a path in `/content` or Google Drive.

Accepts:
- PNG
- JPEG / JPG
- WebP

Zero API keys or external billing required.

## Validation requirements

Validate and report:
- File exists and is readable.
- File size (in KB / MB).
- Image format (PNG, JPEG, WebP).
- Image width and height (pixels).
- Aspect ratio.
- Whether the image contains transparency (Alpha channel).
- Advisory warnings on resolution (<512px) or extreme aspect ratios.
- Display the uploaded image inline for visual inspection.

## Image preparation

Saves the standardized image to:
`/content/lazyad_assets/product_input.png`

Preserves the original uploaded file separately as `/content/lazyad_assets/original_<filename>`.

No destructive cropping, AI hallucinations, or geometry changes are applied.

## Acceptance criteria

S2 passes when:
1. A user can upload a supported image.
2. The notebook validates and displays the image inline.
3. The image is saved to `/content/lazyad_assets/product_input.png`.
4. The saved image can be reopened and verified non-empty.
5. The report includes its dimensions, format, file size, and aspect ratio.
6. The image is staged and ready for Spike S3.
7. Errors are handled clearly.

## Strict scope limits

- Do not implement S3, S4, or S5.
- Do not install TripoSR.
- Do not generate a GLB.
- Do not render video.
- Do not create `LazyAd_MVP.ipynb`.
- Do not download large AI models.
