# Photo-to-Wood-Engraving

Convert photographs into convincing black-and-white wood engraving illustrations using ComfyUI, SDXL diffusion, ControlNet structure conditioning, and print-aware postprocessing.

## Overview

A convincing wood engraving is defined less by "making it monochrome" and more by reproducing **printmaking logic**: tonal gradients expressed through line density, line width, and cross-hatching, with crisp ink (near-binary blacks) on paper texture, and with structure (contours) remaining legible under aggressive stylization.

This project provides ready-to-use ComfyUI workflow files and documentation for three use cases:

- **Portrait** — identity-preserving engraving with Canny ControlNet
- **Landscape** — tone-driven hatching with Depth + Canny dual ControlNet
- **Architecture** — straight-edge preservation with strong Canny ControlNet

### Approach

The workflows use a hybrid approach:

1. **Photo preprocessing** — clean tone and structure maps
2. **Diffusion-based stylization** — SDXL base with engraving LoRA + ControlNet structure control
3. **Print-aware postprocessing** — binarization via threshold masking
4. **High-resolution finishing** — optional tiled VAE decode for large outputs and upscaling

## Quick Start

### Prerequisites

- [ComfyUI](https://github.com/comfyanonymous/ComfyUI) installed and running
- A ControlNet preprocessor node pack (e.g., [comfyui_controlnet_aux](https://github.com/Fannovel16/comfyui_controlnet_aux))

### 1. Download Required Models

See [docs/models.md](docs/models.md) for the complete model list. At minimum you need:

| Model | Download | Place in |
|---|---|---|
| SDXL Base 1.0 | [HuggingFace](https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0) | `models/checkpoints/` |
| Engraving LoRA | [MaxNoichl/engraving-sdxl-lora-001](https://huggingface.co/MaxNoichl/engraving-sdxl-lora-001) | `models/loras/` |
| Canny ControlNet SDXL | [diffusers/controlnet-canny-sdxl-1.0](https://huggingface.co/diffusers/controlnet-canny-sdxl-1.0) | `models/controlnet/` |
| Depth ControlNet SDXL | [diffusers/controlnet-depth-sdxl-1.0](https://huggingface.co/diffusers/controlnet-depth-sdxl-1.0) | `models/controlnet/` |

### 2. Install Custom Nodes

ComfyUI core does **not** ship with ControlNet preprocessors. Install a preprocessor pack:

```bash
cd ComfyUI/custom_nodes
git clone https://github.com/Fannovel16/comfyui_controlnet_aux
```

### 3. Load a Workflow

1. Open ComfyUI in your browser
2. Drag and drop one of the workflow JSON files onto the canvas:
   - [`workflows/portrait_engraving.json`](workflows/portrait_engraving.json) — portraits
   - [`workflows/landscape_engraving.json`](workflows/landscape_engraving.json) — landscapes
   - [`workflows/architecture_engraving.json`](workflows/architecture_engraving.json) — architecture
3. Load your input photo into the `LoadImage` node
4. Click **Queue Prompt**

## Workflow Details

### Portrait Engraving

Optimized for identity preservation under aggressive line stylization.

```
LoadImage → ImageScale (1024×1280)
  ├→ CannyEdgePreprocessor → ControlNetApplyAdvanced (strength 0.9, end 0.75)
  └→ VAEEncode ─────────────────────────────────────────┐
CheckpointLoaderSimple → LoraLoader (0.8/0.6)            │
  ├→ CLIPTextEncodeSDXL (positive) ──→ ControlNet ──┐   │
  └→ CLIPTextEncodeSDXL (negative) ──→ ControlNet ──┤   │
                                         KSampler ←─┘←──┘
                                    (seed 135791113, steps 30,
                                     cfg 5.0, denoise 0.48)
                                            │
                                       VAEDecode
                                            │
                              ImageToMask → ThresholdMask (0.55)
                                            │
                                    MaskToImage → SaveImage
```

**Key parameters:**
- **KSampler**: seed `135791113`, steps `30`, cfg `5.0`, sampler `dpmpp_2m_sde`, scheduler `karras`, denoise `0.48`
- **LoRA**: strength_model `0.8`, strength_clip `0.6`
- **ControlNet (Canny)**: strength `0.9`, start `0.0`, end `0.75`
- **ThresholdMask**: value `0.55`

### Landscape Engraving

Dual ControlNet (Depth primary + Canny secondary) prevents busy noise in skies and water.

```
LoadImage → ImageScale (1216×832)
  ├→ DepthAnythingPreprocessor → ControlNetApplyAdvanced (strength 0.75, start 0.1, end 1.0)
  ├→ CannyEdgePreprocessor → ControlNetApplyAdvanced (strength 0.45, end 0.5)
  └→ VAEEncode ──────────────────────────────────────────────┐
CheckpointLoaderSimple → LoraLoader (0.8/0.6)                │
  ├→ CLIPTextEncodeSDXL (positive) → Depth CN → Canny CN ┐  │
  └→ CLIPTextEncodeSDXL (negative) → Depth CN → Canny CN ┤  │
                                           KSampler ←─────┘←─┘
                                      (seed 246802468, steps 32,
                                       cfg 4.5, denoise 0.58)
                                              │
                                         VAEDecode
                                              │
                                ImageToMask → ThresholdMask (0.55)
                                              │
                                      MaskToImage → SaveImage
```

**Key parameters:**
- **KSampler**: seed `246802468`, steps `32`, cfg `4.5`, sampler `dpmpp_2m_sde`, scheduler `karras`, denoise `0.58`
- **Depth ControlNet**: strength `0.75`, start `0.1`, end `1.0`
- **Canny ControlNet**: strength `0.45`, start `0.0`, end `0.5`
- **ThresholdMask**: value `0.55`

**Tip:** If the sky becomes noisy hatch, reduce denoise and/or reduce LoRA strength. Consider dithering (Floyd–Steinberg) instead of hard threshold for smooth regions.

### Architecture Engraving

Prioritizes edge truthfulness with strong Canny ControlNet and conservative denoise.

```
LoadImage → ImageScale (1344×768)
  ├→ CannyEdgePreprocessor → ControlNetApplyAdvanced (strength 1.05, end 0.85)
  └→ VAEEncode ─────────────────────────────────────────┐
CheckpointLoaderSimple → LoraLoader (0.8/0.6)            │
  ├→ CLIPTextEncodeSDXL (positive) ──→ ControlNet ──┐   │
  └→ CLIPTextEncodeSDXL (negative) ──→ ControlNet ──┤   │
                                         KSampler ←─┘←──┘
                                    (seed 112358132, steps 32,
                                     cfg 4.2, denoise 0.35)
                                            │
                                       VAEDecode
                                            │
                              ImageToMask → ThresholdMask (0.58)
                                            │
                                    MaskToImage → SaveImage
```

**Key parameters:**
- **KSampler**: seed `112358132`, steps `32`, cfg `4.2`, sampler `dpmpp_2m_sde`, scheduler `karras`, denoise `0.35`
- **ControlNet (Canny)**: strength `1.05`, start `0.0`, end `0.85`
- **ThresholdMask**: value `0.58`

**Caution:** Lower denoise (0.25–0.45) prevents geometry drift. Avoid high diffusion-upscale denoise for architectural subjects.

## Parameter Reference

### KSampler Ranges

| Parameter | Portrait | Landscape | Architecture | Notes |
|---|---|---|---|---|
| seed | 135791113 | 246802468 | 112358132 | Fixed for reproducibility |
| steps | 30 | 32 | 32 | 24–40 range |
| cfg | 5.0 | 4.5 | 4.2 | 3.5–7.0 range |
| sampler | dpmpp_2m_sde | dpmpp_2m_sde | dpmpp_2m_sde | Recommended starting point |
| scheduler | karras | karras | karras | Recommended starting point |
| denoise | 0.48 | 0.58 | 0.35 | Lower = more photo preservation |

### ControlNet Ranges

| Parameter | Portrait (Canny) | Landscape (Depth) | Landscape (Canny) | Architecture (Canny) |
|---|---|---|---|---|
| strength | 0.9 | 0.75 | 0.45 | 1.05 |
| start_percent | 0.0 | 0.1 | 0.0 | 0.0 |
| end_percent | 0.75 | 1.0 | 0.5 | 0.85 |

### LoRA Strength

| Parameter | Default | Range |
|---|---|---|
| strength_model | 0.8 | 0.5–1.1 |
| strength_clip | 0.6 | 0.3–1.0 |

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Gray haze / muddy midtones | Insufficient contrast | Increase LoRA strength; lower ThresholdMask value (0.45–0.55); add binarization stage |
| "Wormy" circular hatching | Non-engraver-like line flow | Increase structure controls; tighten denoise; add Depth conditioning; reduce CFG |
| Portrait identity drift | Style overwhelms identity | Use InstantID; reduce denoise (0.30–0.45); keep Canny end_percent ~0.75–0.85 |
| Architecture lines bending | Geometry distortion | Lower denoise (0.25–0.35); increase Canny strength (1.0–1.2); avoid high upscale denoise |
| Seams in tiled upscaling | Tile boundary artifacts | Reduce tile denoise; increase overlap; use Ultimate SD Upscale |
| Noisy sky hatching (landscape) | Over-stylization of smooth areas | Reduce denoise and LoRA strength; use dithering instead of hard threshold |

## Documentation

- [docs/models.md](docs/models.md) — Complete model list with download links and installation paths
- [docs/prompts.md](docs/prompts.md) — Prompt templates for all use cases with SDXL dual-encoder strategy
- [docs/postprocessing.md](docs/postprocessing.md) — Binarization, dithering, morphological cleanup, emboss, and upscaling guide

## Project Structure

```
photo-to-wood-engraving/
├── README.md                              # This file
├── workflows/
│   ├── portrait_engraving.json            # Portrait workflow (Canny ControlNet)
│   ├── landscape_engraving.json           # Landscape workflow (Depth + Canny ControlNet)
│   └── architecture_engraving.json        # Architecture workflow (Strong Canny ControlNet)
└── docs/
    ├── models.md                          # Model download reference
    ├── prompts.md                         # Prompt templates
    └── postprocessing.md                  # Postprocessing guide
```

## License

This project provides workflow configurations and documentation. Model weights are subject to their respective licenses (see individual model cards on HuggingFace).
