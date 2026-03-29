# Prompt Templates

Prompt engineering examples for wood engraving stylization in ComfyUI with SDXL.

## General Engraving

**Positive:**
```
traditional wood engraving print, black ink on textured paper, intricate burin lines,
dense cross-hatching, high contrast, crisp contours, 19th century illustration,
no color, printmaking, museum-quality
```

**Negative:**
```
photograph, grayscale gradient, smooth shading, airbrush, watercolor, charcoal smudge,
pencil sketch, comic screentone dots, halftone dots, blurry, soft focus, low contrast,
plastic texture, CGI, glossy
```

## Portrait Engraving

**Positive:**
```
portrait, traditional wood engraving print, intricate cross-hatching, burin lines,
high contrast, black ink on paper, fine linework, realistic facial proportions,
sharp silhouette
```

**Negative:**
```
photo, smooth skin shading, airbrush, grayscale haze, blur, watercolor, pencil,
charcoal, halftone dots, noisy speckle
```

## Landscape Engraving

**Positive:**
```
landscape, traditional wood engraving, dense hatching in shadows, sparse hatching
in highlights, crisp contour lines, black ink on paper, vintage illustration
```

**Negative:**
```
photorealistic lighting gradients, smooth shading, foggy grayscale wash, blur,
watercolor
```

## Architecture Engraving

**Positive:**
```
architectural illustration, traditional wood engraving, precise linework,
cross-hatching for shadow planes, high contrast ink print, straight edges,
crisp perspective lines
```

**Negative:**
```
bent lines, warped geometry, fisheye, painterly brushstrokes, watercolor, blurry,
soft shading
```

## SDXL Dual-Encoder Strategy

SDXL uses two text encoders (CLIP-L and CLIP-G). The `CLIPTextEncodeSDXL` node exposes two text inputs (`text_g` and `text_l`). A useful strategy:

- **text_g** (CLIP-G): Place material/medium tokens — `"wood engraving, cross-hatching, etched lines, ink on paper"`
- **text_l** (CLIP-L): Place subject/scene tokens — `"portrait of a woman, detailed face, sharp contours"`

Alternatively, duplicate the same prompt in both fields for simplicity. Experiment to find what works best for your specific image.
