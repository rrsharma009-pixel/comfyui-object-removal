# Object Removal — ComfyUI Assessment

## Project Overview
A ComfyUI inpainting workflow designed to remove an unwanted object from an image while reconstructing the affected area to blend with the surrounding scene.

## Files
- `vecteezy_a-couple-holding-hands-at-a-table-with-food-and-wine_82682115.jpeg` — original source image
- `Inpaint_Final.png` — final object removal result
- `In-paint_remove_object_v01.json` — ComfyUI workflow
- `README_Object_Removal.md` — detailed workflow documentation

## Workflow
The workflow uses inpainting to reconstruct the selected image region after object removal, aiming to preserve the surrounding scene and maintain visual consistency.

## Usage
1. Open ComfyUI.
2. Load `In-paint_remove_object_v01.json`.
3. Ensure the required models and custom nodes are installed.
4. Load the source image and run the workflow.

