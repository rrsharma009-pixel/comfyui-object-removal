# Inpainting — Object Removal

## Overview

This project demonstrates mask-guided object removal in ComfyUI. The workflow removes bread and its shadow from a source photograph and reconstructs the affected area to blend with the surrounding plate and scene.

## Objective

Remove the unwanted bread and bread shadow from the plate while keeping the rest of the photograph as consistent as possible.

## Model and Environment

| Component | Configuration |
|---|---|
| Base model | `flux2/flux-2-klein-9b-int8-convrot.safetensors` |
| Text encoder | `qwen_3_8b_fp8mixed.safetensors` |
| Text encoder type | `flux2` |
| VAE | `flux2-vae.safetensors` |
| ComfyUI version recorded in workflow | `0.38.1` |
| Inpainting crop and stitch | Pixaroma ComfyUI nodes |
| Image resizing | KJNodes `ImageResizeKJv2` |

The model weights are external dependencies and are not included in this repository.

## Input Image

The workflow loads the following source image:

`vecteezy_a-couple-holding-hands-at-a-table-with-food-and-wine_82682115.jpeg`

The workflow's crop/mask editor state records a source canvas of **5504 × 3072 pixels**.

## Prompt

**Positive prompt recorded in the workflow:**

> Remove bread and bread shaodw from plate

This is the exact prompt text saved in the supplied workflow.

The workflow does not contain a separate negative-prompt text. It uses `ConditioningZeroOut` for the negative conditioning branch.

## Mask Preparation and Image Processing

The workflow uses the Pixaroma inpaint editor and `PixaromaInpaintCrop` to define and prepare the region to be reconstructed. The mask is handled within the node editor; the workflow does not require a separately supplied mask file as a normal input to the graph.

Recorded crop and mask settings:

| Setting | Value |
|---|---:|
| Size mode | `keep shape (long side)` |
| Target long side | `1024` |
| Dimension multiple | `8` |
| Context around mask | `24 px` |
| Mask grow | `4 px` |
| Mask blur | `4 px` |
| Softness | `16` |
| Blend mode | `mask` |
| Invert mask | `false` |

The crop node prepares a focused region around the mask and retains nearby context to support reconstruction of the plate and surrounding scene.

## Generation Settings

The workflow's `KSamplerAdvanced` node records:

| Setting | Value |
|---|---|
| Steps | `32` |
| CFG | `1` |
| Sampler | `euler` |
| Scheduler | `simple` |
| Start step | `0` |
| End step | `10000` |
| Add noise | `enable` |
| Return with leftover noise | `disable` |

The seed is supplied by a Pixaroma seed node. The saved seed state is `6999662720501586`; the workflow also has the seed control configured to randomize after generation. The advanced sampler's saved noise seed is `234603585436515`, with its control-after-generate setting configured to randomize.

## Workflow

1. Load the source photograph with `LoadImage`.
2. Resize the image using `ImageResizeKJv2`, configured for 1920 × 1080 on the input path.
3. Define the removal region using the Pixaroma inpaint editor.
4. Prepare the masked crop with `PixaromaInpaintCrop`, using the crop and mask settings listed above.
5. Load the Flux.2 Klein model, Qwen 3 8B text encoder, and Flux.2 VAE.
6. Encode the removal prompt with `CLIPTextEncode`. The negative conditioning branch is created using `ConditioningZeroOut`.
7. Encode the cropped image with `VAEEncode` and provide image context through `ReferenceLatent` nodes.
8. Generate the reconstructed region with `KSamplerAdvanced`.
9. Decode the generated latent using `VAEDecode`.
10. Composite the generated crop into the original image with `PixaromaInpaintStitch`.
11. Resize the stitched output through the workflow's output resize node and save it using the `INpaint_` filename prefix.
12. Review the before-and-after images using the workflow's comparison node.

## Stitching and Integration

The `PixaromaInpaintStitch` node records:

- **Blend mode:** `from crop`
- **Color matching:** `strong`
- **Softness:** `-1` (automatic/default behavior in the node)

These settings are intended to blend the reconstructed region with the source photograph and reduce visible transitions.

## Evaluation

The result should be checked for:

- Removal of the targeted bread and its shadow.
- Plausible reconstruction of the plate and surrounding tabletop.
- Consistency of lighting, texture, and perspective.
- Clean transitions around the edited area.
- Preservation of unrelated objects and the wider composition.

The supplied workflow includes a comparison node for inspecting the source and processed images.

## Repository Contents

Suggested project layout:

```text
Assessment-2-Object-Removal/
├── README.md
├── workflows/
│   └── In-paint_remove_object.json
├── inputs/
│   └── vecteezy_a-couple-holding-hands-at-a-table-with-food-and-wine_82682115.jpeg
└── outputs/
    └── INpaint_final.png
```

The filenames above describe the intended layout; the final output filename should match the actual exported image.

## Installation and Reproduction

1. Install ComfyUI.
2. Install the custom node packages required by the workflow, including Pixaroma ComfyUI nodes and KJNodes.
3. Place the Flux.2 Klein checkpoint, Qwen 3 8B text encoder, and Flux.2 VAE in the model directories expected by ComfyUI.
4. Open `In-paint_remove_object.json` in ComfyUI.
5. Ensure the source image is available to the `LoadImage` node.
6. Open the Pixaroma inpaint editor and confirm the intended removal mask is present.
7. Queue the workflow and inspect the saved output.

## Training

This project uses pretrained-model inference. No model training or fine-tuning is performed.

## Known Limitations

- Results depend on mask placement, prompt wording, model versions, and installed custom nodes.
- The exact mask shape is managed by the Pixaroma editor and is not represented as a standalone mask input in the workflow graph.
- The workflow contains an output resize setting of **3887 × 4859**. This is a portrait-shaped configuration, so the final saved image dimensions should be confirmed from the exported output.
- The model weights are not included in the repository and must be installed separately.
