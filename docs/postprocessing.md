# Postprocessing Guide

This document covers the print-aware postprocessing steps that make diffusion output look like authentic ink-on-paper wood engraving.

## Binarization / Thresholding

Wood engravings are near-binary (black ink on white paper). After diffusion, apply a threshold to push the output toward this behavior.

### Using ComfyUI Core Nodes

```
ImageToMask (channel: "red") → ThresholdMask (value: 0.55) → MaskToImage
```

- **ThresholdMask value** range: 0.0–1.0 (default 0.5)
- **Lower values** (0.45–0.50): more ink / darker output
- **Higher values** (0.55–0.65): more paper / lighter, more open linework
- Adjust based on desired "ink coverage"

### Otsu's Method (External)

For automatic threshold selection, Otsu's histogram-based method finds the optimal threshold without manual tuning. This can be applied via ImageMagick or Python preprocessing outside of ComfyUI.

## Dithering (Optional)

When hard thresholding causes banding (especially in smooth regions like skies or skin), error-diffusion dithering preserves perceived tone in near-binary images.

- **Floyd–Steinberg dithering** is the canonical "print halftoning" approach
- Particularly useful for landscape workflows where sky gradients would otherwise become flat
- Can be applied via ImageMagick: `convert input.png -dither FloydSteinberg -monochrome output.png`

## Morphological Cleanup (Optional)

After binarization, broken hatch lines or ink dropouts can be cleaned with morphological operations:

- **Dilate**: thickens lines, fills small gaps in hatching
- **Erode**: thins lines, cleans up ink blobs
- Available via mask/image filter nodes or ImageMagick

## Emboss / Relief Cue (Optional)

A subtle emboss effect dramatically increases "printed" believability by simulating the physical impression of an engraving plate.

### Using ImageMagick (MagickWand nodes)

If you have MagickWand integration in ComfyUI:
```
-shade 135x45 -normalize
```

This adds directional lighting that simulates plate embossing. Use very subtle settings to avoid an overtly 3D look.

## Paper Texture (Optional)

Adding slight paper texture increases print authenticity:

1. Load a paper texture image
2. Blend with the binarized output using composite/multiply operations
3. Keep the blend very subtle (5–15% opacity) to avoid obscuring line detail

## High-Resolution Finishing

### Two-Stage Upscaling (Recommended)

For print-quality output, use a two-stage approach:

1. **Stage 1**: `ImageUpscaleWithModel` (2× neural upscale) — preserves line crispness
2. **Stage 2** (optional): `Ultimate SD Upscale` (tiled diffusion at low denoise 0.15–0.30) — enhances detail without drifting style

### Tiled VAE Decode

For outputs ≥ 2048px, use `VAEDecodeTiled` instead of `VAEDecode` to prevent VRAM spikes during decoding.

### Architecture-Specific Caution

For architectural subjects, keep diffusion-based tile enhancement denoise very conservative (0.10–0.25) to prevent:
- Tile seam inconsistencies
- Geometry drift / line warping
- Invented ornamentation

## Parameter Quick Reference

| Step | Parameter | Portrait | Landscape | Architecture |
|---|---|---|---|---|
| ThresholdMask | value | 0.55 | 0.55 | 0.58 |
| Upscale denoise | denoise | 0.15–0.30 | 0.15–0.30 | 0.10–0.25 |
| Dithering | method | Optional | Recommended for skies | Optional |
| Emboss | intensity | Subtle | Subtle | Subtle |
