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

| Node Pack | Nodes Used | Repo |
|-----------|-----------|------|
| **ComfyUI-LTXVideo** | All `LTXV*` / `LTX2*` nodes | https://github.com/Lightricks/ComfyUI-LTXVideo |
| **ComfyUI-KJNodes** | `VAELoaderKJ`, `DiffusionModelLoaderKJ`, `ManualSigmas`, `GuiderParameters`, `GetImageSize`, `GetImageSizeAndCount`, `GetImageRangeFromBatch`, `VRAM_Debug` | https://github.com/kijai/ComfyUI-KJNodes |
| **RES4LYF** | `ClownSampler_Beta`, `ClownsharKSampler_Beta`, `ClownOptions_ExtraOptions_Beta`, `Sigmas Resample`, `Sigmas Rescale`, `Sigmas Split Value`, `Linear Quadratic Advanced`, `FloatConstant` | https://github.com/ClownsharkBatwing/RES4LYF |
| **ComfyUI_essentials** | `SimpleMath+` | https://github.com/cubiq/ComfyUI_essentials |
| **ComfyUI-VideoHelperSuite** | `VHS_LoadAudioUpload` | https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite |
| **comfyui-various** | `JWImageResizeByLongerSide` | https://github.com/jamesWalker55/comfyui-various |
| **Nvidia RTX Nodes** | `RTXVideoSuperResolution` | https://github.com/Comfy-Org/Nvidia_RTX_Nodes_ComfyUI |

### HuMo workflows (additional)

| Node Pack | Nodes Used | Repo |
|-----------|-----------|------|
| **WanExperiments** | `WanEx_HuMoImageToVideo`, `WanEx_ContextWindowsAdvanced` | https://github.com/drozbay/WanExperiments |
| **ComfyUI-MelBandRoFormer** | `MelBandRoFormerModelLoader`, `MelBandRoFormerSampler` | https://github.com/kijai/ComfyUI-MelBandRoFormer |
| **ComfyUI-FPSChange** | `ImageBatchChangeFPS` | https://github.com/ckinpdx/ComfyUI-FPSChange |

> The FPS node is required because these workflows generate at 50 FPS for quality, but HuMo's motion module expects 25 FPS input.
