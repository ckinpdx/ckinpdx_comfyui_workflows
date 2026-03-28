# ckinpdx ComfyUI Workflows

LTX Video 2.3 workflows built in the ComfyUI UI, structured for clean export to API format. No Get/Set nodes or subgraphs are used, as these do not survive the UI→API conversion.

These workflows prioritize quality over efficiency and are built for high-end hardware (RTX 5090, 96GB DDR5 RAM). They reflect the creator's ongoing research into techniques and approaches gathered from community workflows across the internet.

Core pipeline: two-stage latent upsampling with RTX Video Super Resolution output.

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
| `continuation` | Video temporal inpainting — loads an existing video and regenerates a user-defined time window within it |
| `HuMo` | Includes human motion module (see dependencies) |
| `_API` | Structured for API export (no Get/Set nodes or subgraphs) |

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
| `LTX23continuation_API.json` | Temporal inpainting — loads a video and regenerates a specified time range. |
| `LTX23continuationHuMo_API.json` | Temporal inpainting with HuMo human motion. |

## Node Color Coding

Nodes are color coded by role for easier navigation:

| Color | Role |
|-------|------|
| 🔴 Red | Model loading |
| 🟢 Green | User inputs (prompts, images, audio, parameters) |
| 🟡 Yellow | Generation (sampling, scheduling, conditioning) |
| 🟣 Purple | Saving / output |

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

The **HuMo Long Edge** input controls the resolution fed to the HuMo model. Recommended values: `1280`, `1536`, `1920`, `2560`.

**Choosing compatible dimensions is non-trivial.** LTX, HuMo, and the rescale step each impose constraints, and the interaction between them — particularly across different workflow types — means a combination that works in one workflow may fail in another. Expect to test combinations rather than derive them from first principles. Known working baseline: LTX `2560×1440`, HuMo long edge `1920`.

## Dependencies

### All workflows

| Node Pack | Nodes Used | Repo |
|-----------|-----------|------|
| **ComfyUI-LTXVideo** | All `LTXV*` / `LTX2*` nodes | https://github.com/Lightricks/ComfyUI-LTXVideo |
| **ComfyUI-KJNodes** | `VAELoaderKJ`, `DiffusionModelLoaderKJ`, `ManualSigmas`, `GuiderParameters`, `GetImageSize`, `GetImageSizeAndCount`, `GetImageRangeFromBatch`, `VRAM_Debug` | https://github.com/kijai/ComfyUI-KJNodes |
| **RES4LYF** | `ClownSampler_Beta`, `ClownsharKSampler_Beta`, `ClownOptions_ExtraOptions_Beta`, `Sigmas Resample`, `Sigmas Rescale`, `Sigmas Split Value`, `Linear Quadratic Advanced`, `FloatConstant` | https://github.com/ClownsharkBatwing/RES4LYF |
| **ComfyUI_essentials** | `SimpleMath+`, `ImageFromBatch` | https://github.com/cubiq/ComfyUI_essentials |
| **ComfyUI-VideoHelperSuite** | `VHS_LoadAudioUpload`, `VHS_LoadVideo`, `VHS_VideoInfoSource` | https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite |
| **ComfyUI-WanVideoWrapper** | `NormalizeAudioLoudness` | https://github.com/kijai/ComfyUI-WanVideoWrapper |
| **comfyui-various** | `JWImageResizeByLongerSide` | https://github.com/jamesWalker55/comfyui-various |
| **Nvidia RTX Nodes** | `RTXVideoSuperResolution` | https://github.com/Comfy-Org/Nvidia_RTX_Nodes_ComfyUI |

> **NormalizeAudioLoudness bug:** If the audio input is very short or near-silence, the BS.1770 loudness measurement returns NaN/inf, producing a poisoned tensor that silently passes through and crashes the video save node with `[Errno 22]` during AAC encoding. Fix in `ComfyUI-WanVideoWrapper/nodes_utility.py`:
> ```python
> # before
> if abs(loudness) > 100:
> # after
> if not np.isfinite(loudness) or abs(loudness) > 100:
> ```
> The node will then return audio unchanged rather than passing a NaN tensor downstream.

### HuMo workflows (additional)

| Node Pack | Nodes Used | Repo |
|-----------|-----------|------|
| **WanExperiments** | `WanEx_HuMoImageToVideo`, `WanEx_ContextWindowsAdvanced` | https://github.com/drozbay/WanExperiments |
| **ComfyUI-MelBandRoFormer** | `MelBandRoFormerModelLoader`, `MelBandRoFormerSampler` | https://github.com/kijai/ComfyUI-MelBandRoFormer |
| **ComfyUI-FPSChange** | `ImageBatchChangeFPS` | https://github.com/ckinpdx/ComfyUI-FPSChange |

> The FPS node is required because these workflows generate at 50 FPS for quality, but HuMo's motion module expects 25 FPS input.

## Required Models

### All workflows

| Model | Role |
|-------|------|
| `ltx-2.3-22b-dev_transformer_only_fp8_scaled.safetensors` | LTX 2.3 22B main diffusion model (fp8) |
| `gemma_3_12B_it_heretic_fp8_e4m3fn.safetensors` | Gemma 3 12B text encoder |
| `ltx-2.3-22b-dev_embeddings_connectors.safetensors` | LTX 2.3 embeddings / connector weights |
| `LTX23_video_vae_bf16.safetensors` | LTX 2.3 video VAE |
| `LTX23_audio_vae_bf16.safetensors` | LTX 2.3 audio VAE |
| `ltx-2.3-spatial-upscaler-x2-1.1.safetensors` | LTX 2.3 spatial upscaler (2x) |
| `ltx-2.3-22b-distilled-lora-384.safetensors` | LTX 2.3 distilled LoRA (loaded twice at different weights for each generation stage) |

### HuMo workflows (additional)

| Model | Role |
|-------|------|
| `humo_17B_fp16.safetensors` | HuMo 17B human motion model |
| `Wan2_1_VAE_bf16.safetensors` | Wan 2.1 VAE (used by HuMo) |
| `umt5_xxl_fp8_e4m3fn_scaled.safetensors` | UMT5-XXL text encoder (used by HuMo) |
| `lightx2v_T2V_14B_cfg_step_distill_v2_lora_rank128_bf16.safetensors` | LightX2V distilled LoRA |
| `FaceDetailerV1.safetensors` | Face detail LoRA |
| `MelBandRoformer_fp16.safetensors` | MelBand RoFormer audio separator |
| `whisper_large_v3_encoder_fp16.safetensors` | Whisper large-v3 audio encoder |
