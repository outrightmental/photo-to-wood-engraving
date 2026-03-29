# Required Models

This document lists all models referenced by the workflows, with download sources and installation paths.

## Base Checkpoint

| Model | Source | ComfyUI Path |
|---|---|---|
| SDXL Base 1.0 | [stabilityai/stable-diffusion-xl-base-1.0](https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0) | `models/checkpoints/sd_xl_base_1.0.safetensors` |

## Style LoRA + Embedding

| Model | Source | ComfyUI Path |
|---|---|---|
| Engraving SDXL LoRA | [MaxNoichl/engraving-sdxl-lora-001](https://huggingface.co/MaxNoichl/engraving-sdxl-lora-001) | `models/loras/engraving-sdxl-lora-001.safetensors` |
| Engraving Embedding (optional) | Same repository | `models/embeddings/` (follow repo instructions) |

## ControlNet Models

| Model | Purpose | Source | ComfyUI Path |
|---|---|---|---|
| Canny ControlNet SDXL | Edge/contour preservation | [diffusers/controlnet-canny-sdxl-1.0](https://huggingface.co/diffusers/controlnet-canny-sdxl-1.0) | `models/controlnet/controlnet-canny-sdxl-1.0.safetensors` |
| Depth ControlNet SDXL | 3D structure for hatching alignment | [diffusers/controlnet-depth-sdxl-1.0](https://huggingface.co/diffusers/controlnet-depth-sdxl-1.0) | `models/controlnet/controlnet-depth-sdxl-1.0.safetensors` |

### Optional ControlNet Models

| Model | Purpose | Source |
|---|---|---|
| Union ControlNet SDXL | Multi-control type support | [xinsir/controlnet-union-sdxl-1.0](https://huggingface.co/xinsir/controlnet-union-sdxl-1.0) |
| Lineart ControlNet SDXL | Stroke-based guidance | [ShermanG/ControlNet-Standard-Lineart-for-SDXL](https://huggingface.co/ShermanG/ControlNet-Standard-Lineart-for-SDXL) |
| T2I-Adapter Lineart SDXL | Alternative line conditioning | [TencentARC/t2i-adapter-lineart-sdxl-1.0](https://huggingface.co/TencentARC/t2i-adapter-lineart-sdxl-1.0) |

## Upscale Models (Optional)

| Model | Purpose | ComfyUI Path |
|---|---|---|
| Real-ESRGAN 4x | Neural upscaling | `models/upscale_models/` |
| SwinIR | Neural upscaling | `models/upscale_models/` |

Upscale model filenames vary by distribution. ComfyUI's `UpscaleModelLoader` enumerates whatever is in the upscale models folder.

## Identity Preservation (Optional, Portraits)

| Model | Purpose | Source |
|---|---|---|
| InstantID | Single-photo identity retention | [InstantX/InstantID](https://huggingface.co/InstantX/InstantID) |

Requires a ComfyUI extension (e.g., ComfyUI_InstantID). Follow the extension's installation instructions.

## Style Reference (Optional)

| Model | Purpose | Source |
|---|---|---|
| IP-Adapter SDXL | Image prompt style conditioning | Various (e.g., h94/IP-Adapter) |

Requires a ComfyUI extension (e.g., ComfyUI_IPAdapter_plus). Supply 1–3 authentic engraving prints as style anchors.

## Depth & Preprocessor Models

The landscape workflow uses a depth preprocessor node (`DepthAnythingPreprocessor`) that expects the **Depth Anything ViT-L 14** weights:

| Model file | Purpose | Source | Typical Path |
|---|---|---|---|
| `depth_anything_vitl14.pth` | Monocular depth estimation for landscape workflow | [LiheYoung/depth-anything-large-hf](https://huggingface.co/LiheYoung/depth-anything-large-hf) or auto-downloaded by `comfyui_controlnet_aux` | `models/annotators/depth_anything_vitl14.pth` |

When using the `comfyui_controlnet_aux` extension, the Depth Anything weights are typically auto-downloaded to `models/annotators/depth_anything_vitl14.pth` on first use of the Depth Anything nodes. If your environment has no internet access, download the compatible Depth Anything ViT-L 14 weights manually from the source above and place them at the configured path.

## Required Custom Node Packs

ComfyUI core does **not** include ControlNet preprocessors. Install one of:

- [comfyui_controlnet_aux](https://github.com/Fannovel16/comfyui_controlnet_aux) — provides Canny, HED, Depth (including Depth Anything), Lineart preprocessors, and auto-downloads required annotator models such as `depth_anything_vitl14.pth` into `models/annotators/` on first use
- [ComfyUI_UltimateSDUpscale](https://github.com/ssitu/ComfyUI_UltimateSDUpscale) — tiled diffusion upscaling (optional)
- [ComfyUI_IPAdapter_plus](https://github.com/cubiq/ComfyUI_IPAdapter_plus) — IP-Adapter support (optional)

Install via ComfyUI Manager or by cloning into `custom_nodes/`.
