# ckinpdx ComfyUI Workflows

LTX Video 2.3 workflows built in the ComfyUI UI, structured for clean export to API format. No Get/Set nodes or subgraphs are used, as these are not needed for the UI→API conversion.

These workflows prioritize quality over efficiency and are built for high-end consumer hardware (RTX 5090, 96GB DDR5 RAM). They reflect the creator's ongoing research into techniques and approaches gathered from community workflows across the internet.

Core pipeline: two-stage latent upsampling with RTX Video Super Resolution output.

## Naming Convention

```
[1Stage_] LTX23 [_A-] [TorI2V | _FLF-I2V | _FLF-A-I2V | _Continuation] [_IDLora] [_HuMo | _WAN22] _API
```

| Segment | Meaning |
|---------|---------|
| `1Stage_` | Single-stage LTX generation (no two-stage latent upsampling) |
| `LTX23` | LTX Video 2.3 |
| `A-` | Audio input |
| `TorI2V` | T2V or I2V — mode toggled via boolean input (`T2V True` / `I2V False`) |
| `FLF` | First and Last Frame — conditions generation on both a start and end frame |
| `I2V` | Image to Video (start frame) |
| `A-I2V` | Audio + Image to Video |
| `_Continuation` | True video continuation — loads an existing video and generates new frames from a specified start point forward |
| `_IDLora` | Identity LoRA — voice cloning via reference audio |
| `_HuMo` | Includes human motion module (see dependencies) |
| `_WAN22` | Wan 2.2 low-noise refinement pass instead of HuMo (see dependencies) |
| `_API` | Structured for API export (no Get/Set nodes or subgraphs) |

## Workflows

### LTX23/

Base workflows — LTX 2.3 only, no motion refinement.

| File | Description |
|------|-------------|
| `LTX23_TorI2V_API.json` | T2V or I2V toggle. |
| `LTX23_A-TorI2V_API.json` | T2V or I2V toggle with audio input. |
| `LTX23_TorI2V_IDLora_API.json` | T2V or I2V toggle with identity LoRA. |
| `LTX23_FLF-I2V_API.json` | First and last frame I2V — generates content between a start and end frame. |
| `LTX23_FLF-A-I2V_API.json` | First and last frame A-I2V with audio. |
| `LTX23_Continuation_API.json` | True video continuation — loads a video and generates new frames from a specified start point. |

### LTX23_HuMo/

LTX 2.3 + HuMo human motion conditioning. Requires additional models (see below).

| File | Description |
|------|-------------|
| `LTX23_TorI2V_Humo_API.json` | T2V or I2V toggle with HuMo human motion. |
| `LTX23_A-TorI2V_Humo_API.json` | T2V or I2V toggle with audio + HuMo human motion. |
| `LTX23_TorI2V_IDLora_HuMo_API.json` | T2V or I2V toggle with identity LoRA + HuMo human motion. |
| `LTX23_FLF-I2V_HuMo_API.json` | First and last frame I2V with HuMo human motion. |
| `LTX23_FLF-A-I2V_HuMo_API.json` | First and last frame A-I2V with HuMo human motion. |
| `1Stage_LTX23_A-I2V_HuMo_API.json` | Single-stage audio + image-to-video with HuMo. |
| `LTX23_Continuation_HuMo_API.json` | Video continuation with HuMo human motion. |

### LTX23_WAN22/

LTX 2.3 + Wan 2.2 low-noise refinement. A simpler alternative to HuMo — no audio separation or motion model required.

| File | Description |
|------|-------------|
| `LTX23_TorI2V_WAN22_API.json` | T2V or I2V toggle with Wan 2.2 refinement. |
| `LTX23_TorI2V_IDLora_WAN22_API.json` | T2V or I2V toggle with identity LoRA + Wan 2.2 refinement. |
| `LTX23_A-TorI2V_WAN22_API.json` | T2V or I2V toggle with audio + Wan 2.2 refinement. |
| `LTX23_FLF-I2V_WAN22_API.json` | First and last frame I2V with Wan 2.2 refinement. |
| `LTX23_FLF-A-I2V_WAN22_API.json` | First and last frame A-I2V with Wan 2.2 refinement. |
| `LTX23_Continuation_WAN22_API.json` | Video continuation with Wan 2.2 refinement. |
| `1Stage_LTX23_TorI2V_WAN22_API.json` | Single-stage T2V or I2V toggle with Wan 2.2 refinement. |
| `1Stage_LTX23_A-TorI2V_WAN22_API.json` | Single-stage T2V or I2V toggle with audio + Wan 2.2 refinement. |

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
- **RTX Video Super Resolution** — final output upscale (requires RTX GPU)
- **LTXVChunkFeedForward** — chunked processing for longer clips
- **ClownSampler / ClownsharKSampler** — advanced sampling
- **LTXDimensionCalculator / LTXFrameCalculator** — dimension and frame count validation
- **FADE-labeled primitives** — exposed inputs for use with the FADE API layer

Most workflows use a **two-stage generation** pipeline — low-res draft pass → latent upsample → full-res refinement. `1Stage_` workflows skip the first stage.

`TorI2V` workflows use a **boolean toggle** (`T2V True` / `I2V False`) to switch between text-to-video and image-to-video mode in a single workflow.

`FLF` workflows condition on **both a start and end frame**, with the model generating the content in between.

HuMo workflows additionally include:
- **MelBandRoFormer** — audio source separation
- **WanEx_HuMoImageToVideo** — human motion conditioning
- **VHS_SelectEveryNthImage** — downsamples 50 FPS generation to 25 FPS required by HuMo

The **HuMo Long Edge** input controls the resolution fed to the HuMo model. Recommended values: `1280`, `1536`, `1920`, `2560`.

**Choosing compatible dimensions is non-trivial.** LTX, HuMo, and the rescale step each impose constraints, and the interaction between them — particularly across different workflow types — means a combination that works in one workflow may fail in another. Expect to test combinations rather than derive them from first principles. Known working baseline: LTX `2560×1440`, HuMo long edge `1920`.

WAN22 workflows use **Wan 2.2 low-noise refinement** in place of HuMo — no MelBandRoFormer, no Whisper, no HuMo model.

## Dependencies

### All workflows

| Node Pack | Nodes Used | Repo |
|-----------|-----------|------|
| **ComfyUI-LTXVideo** | All `LTXV*` / `LTX2*` nodes | https://github.com/Lightricks/ComfyUI-LTXVideo |
| **ComfyUI-KJNodes** | `VAELoaderKJ`, `DiffusionModelLoaderKJ`, `ManualSigmas`, `GuiderParameters`, `GetImageSize`, `GetImageSizeAndCount`, `VRAM_Debug`, `LoadAndResizeImage`, `LazySwitchKJ`, `LTXVImgToVideoInplaceKJ`, `BatchImagesNode` | https://github.com/kijai/ComfyUI-KJNodes |
| **RES4LYF** | `ClownSampler_Beta`, `ClownsharKSampler_Beta`, `ClownOptions_ExtraOptions_Beta`, `Sigmas Resample`, `Sigmas Rescale`, `Sigmas Split Value`, `Linear Quadratic Advanced`, `FloatConstant` | https://github.com/ClownsharkBatwing/RES4LYF |
| **ComfyUI_essentials** | `SimpleMath+`, `ImageFromBatch` | https://github.com/cubiq/ComfyUI_essentials |
| **ComfyUI-VideoHelperSuite** | `VHS_LoadAudioUpload`, `VHS_LoadVideo`, `VHS_VideoInfoSource`, `VHS_SelectEveryNthImage`, `VHS_VideoCombine` | https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite |
| **ComfyUI-WanVideoWrapper** | `NormalizeAudioLoudness` | https://github.com/kijai/ComfyUI-WanVideoWrapper |
| **ComfyUI-LTXDimensionCalculator** | `LTXDimensionCalculator`, `LTXFrameCalculator` | https://github.com/ckinpdx/ComfyUI-LTXDimensionCalculator |
| **Nvidia RTX Nodes** | `RTXVideoSuperResolution` | https://github.com/Comfy-Org/Nvidia_RTX_Nodes_ComfyUI |



### HuMo workflows (additional)

| Node Pack | Nodes Used | Repo |
|-----------|-----------|------|
| **WanExperiments** | `WanEx_HuMoImageToVideo`, `WanEx_ContextWindowsAdvanced` | https://github.com/drozbay/WanExperiments |
| **ComfyUI-MelBandRoFormer** | `MelBandRoFormerModelLoader`, `MelBandRoFormerSampler` | https://github.com/kijai/ComfyUI-MelBandRoFormer |

> `VHS_SelectEveryNthImage` (VideoHelperSuite) is used to downsample 50 FPS generation to 25 FPS required by HuMo.

### WAN22 workflows (additional)

| Node Pack | Nodes Used | Repo |
|-----------|-----------|------|
| **ComfyUI-KJNodes** | `ColorMatchV2` (additional node — pack already required above) | https://github.com/kijai/ComfyUI-KJNodes |

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
| `ltx-2.3-22b-distilled-lora-384.safetensors` | LTX 2.3 distilled LoRA (in two-stage workflows, loaded twice at different weights per stage) |
| `ltx-2.3-id-lora-celebvhq-3k.safetensors` | Identity LoRA for character/face consistency (`IDLora` workflows) |

### HuMo workflows (additional)

| Model | Role |
|-------|------|
| `humo_17B_fp16.safetensors` | HuMo 17B human motion model |
| `Wan2_1_VAE_bf16.safetensors` | Wan 2.1 VAE (used by HuMo) |
| `umt5_xxl_fp8_e4m3fn_scaled.safetensors` | UMT5-XXL text encoder (used by HuMo) |
| `lightx2v_T2V_14B_cfg_step_distill_v2_lora_rank128_bf16.safetensors` | LightX2V distilled LoRA |
| `MelBandRoformer_fp16.safetensors` | MelBand RoFormer audio separator |
| `whisper_large_v3_encoder_fp16.safetensors` | Whisper large-v3 audio encoder |

### WAN22 workflows (additional)

| Model | Role |
|-------|------|
| `wan2.2_t2v_low_noise_14B_fp8_scaled.safetensors` | Wan 2.2 low-noise 14B refinement model (fp8) |
| `Wan2_1_VAE_bf16.safetensors` | Wan 2.1 VAE |
| `umt5_xxl_fp8_e4m3fn_scaled.safetensors` | UMT5-XXL text encoder |
| `lightx2v_T2V_14B_cfg_step_distill_v2_lora_rank128_bf16.safetensors` | LightX2V distilled LoRA |

