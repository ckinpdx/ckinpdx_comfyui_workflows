# ckinpdx ComfyUI Workflows

LTX Video 2.3 workflows in API format, built around a two-stage latent upsampling pipeline with RTX Video Super Resolution output.

## Naming Convention

```
[HQ_] LTX23 [T2V | I2V | AI2V] [HuMo] _API
```

| Segment | Meaning |
|---------|---------|
| `HQ_` | Higher quality / resolution variant |
| `LTX23` | LTX Video 2.3 |
| `T2V` | Text to Video |
| `I2V` | Image to Video (image as start frame) |
| `AI2V` | Audio + Image to Video (image start frame + audio file) |
| `HuMo` | Includes human motion module (see dependencies) |
| `_API` | ComfyUI API format |

## Workflows

| File | Description |
|------|-------------|
| `LTX23T2V_API.json` | Text-to-video. Prompts only, no image input. |
| `LTX23I2V_API.json` | Image-to-video. Uses a start frame image. |
| `LTX23AI2V_API.json` | Audio-driven image-to-video. Start frame + audio file guides generation. |
| `LTX23T2VHuMo_API.json` | Text-to-video with HuMo human motion. |
| `LTX23I2VHuMo_API.json` | Image-to-video with HuMo human motion. |
| `LTX23AI2V_HuMoAPI.json` | Audio + image-to-video with HuMo human motion. |
| `HQ_LTX23AI2V_HuMoAPI.json` | High quality variant of AI2V + HuMo. |

## Pipeline Features

All workflows share:
- **Two-stage generation** — low-res draft pass → latent upsample → full-res refinement
- **RTX Video Super Resolution** — final output upscale (requires RTX GPU)
- **ManualSigmas** — custom sigma schedules for both stages
- **LTXVChunkFeedForward** — chunked processing for longer clips
- **ClownSampler / ClownsharKSampler** — advanced sampling
- **FADE-labeled primitives** — exposed inputs for use with the FADE API layer

HuMo workflows additionally include:
- **MelBandRoFormer** — audio source separation
- **WanEx_HuMoImageToVideo** — human motion conditioning
- **ImageBatchChangeFPS** — downsamples 50 FPS generation to 25 FPS required by HuMo

## Dependencies

### All workflows
Standard ComfyUI with LTX Video 2.3 model support.

### HuMo workflows
Requires two additional custom node packs:

**WanExperiments** (HuMo nodes):
https://github.com/drozbay/WanExperiments

**ComfyUI-FPSChange** (FPS downsampling):
https://github.com/ckinpdx/ComfyUI-FPSChange

> The FPS node is required because these workflows generate at 50 FPS for quality, but HuMo's motion module expects 25 FPS input.
