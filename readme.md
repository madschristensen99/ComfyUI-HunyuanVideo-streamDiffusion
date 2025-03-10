# ComfyUI + Hunyuan + Streaming

**A highly experimental method for developing livestreams using ComfyUI, Hunyuan, and Stream Diffusion.**

This project explores the combination of the **Hunyuan model** (effective for blocking and general world modeling) and **Stream Diffusion** (effective for actor likeness transformation) to create livestreams that adhere to actor likeness. However, due to the computational intensity of generating high-quality Hunyuan videos (5 seconds per generation, ~380 seconds per generation), running this concept live currently requires approximately **75 A100 GPUs**, making it impractical at this time.

---

## Overview
The Hunyuan model excels at blocking and general world modeling but struggles with actor likeness. Stream Diffusion, on the other hand, can transform videos that don't resemble the actor into videos that do. By combining these two approaches, this project aims to create livestreams that maintain actor likeness while leveraging the strengths of both models.

---

## Key Features
- **Hunyuan Model**: Effective for blocking and world modeling.
- **Stream Diffusion**: Transforms videos to adhere to actor likeness.
- **ComfyUI Integration**: Streamlines the workflow for experimental livestream development.
- **hunyuanWorkflow.json**: A custom workflow configuration for integrating Hunyuan and Stream Diffusion.

---

## Workflow
The core of this project is the `hunyuanWorkflow.json` file, which outlines the integration of Hunyuan and Stream Diffusion within ComfyUI. This workflow is designed to:
1. Generate base videos using the Hunyuan model. -- DONE
2. Apply Stream Diffusion to enhance actor likeness. -- WORKING ON IT - | [![StreamDiffusion Repository](https://img.shields.io/badge/GitHub-StreamDiffusion-blue?style=for-the-badge&logo=github)](https://github.com/cumulo-autumn/StreamDiffusion)
3. Combine the outputs for a cohesive livestream. -- TODO

### Video Demonstration
<!-- Replace the placeholder below with an embedded video or a link to a video -->
🎥 **[[Video Placeholder](https://www.youtube.com/watch?v=54RYwTB-THg)]**  
*Insert your video here. You can use a platform like YouTube, Vimeo, or Loom to host the video and embed it using Markdown or HTML.*

---

## Usage
In runpod, use the thelocallab/hunyuan-comfyui template. It takes a half our to load. Then in comfyUI, you can load the hunyuanWorkflow.json 


# Everything below this was original when I forked

# ComfyUI wrapper nodes for [HunyuanVideo](https://github.com/Tencent/HunyuanVideo)

# Update 3:

It's been hectic couple of weeks with this model, I've lost track of what has happened since the start, but I'll try to present some of the more important updates:

## Official scaled fp8 weights were released:

https://huggingface.co/tencent/HunyuanVideo/blob/main/hunyuan-video-t2v-720p/transformers/mp_rank_00_model_states_fp8.pt

Even if this file is .pt it's completely safe and it is loaded with weights_only, the scale map is included with the nodes. To use this model you have to use the `fp8_scaled` -quantization option in the model loader.
The quality of these weights is much closer to the original bf16, downside is that they do not currently support fp8 fast mode, or LoRAs.

## Almost free quality increase with [Enhance-A-Video](https://github.com/NUS-HPC-AI-Lab/Enhance-A-Video):

This has a very slight hit on inference speed and zero hit on memory use, initial tests indicate it's absolutely worth using.

![image](https://github.com/user-attachments/assets/68f0b5eb-aa23-49e1-a48f-fd3c4b1108ed)

https://github.com/user-attachments/assets/e19b30e1-5f67-4e75-9c73-716d4569c319

https://github.com/user-attachments/assets/083353a2-e9aa-43e9-a916-ff3af1d581c1



# Update 2: Experimental IP2V - Image Prompting to Video via VLM by @Dango233
## WORK IN PROGRESS - But it should work now!

Now you can feed image to the VLM as condition of generations! This is different from image2video where the image become the first frame of the video. IP2V uses image as a part of the prompt, to extract the concept and style of the image.
So - very much like IPAdapter - but VLM will do the heavy lifting for you!

Now this is a tuning free approach but with further task specific tuning we can expand the use scenarios.

## Guide to Using `xtuner/llava-llama-3-8b-v1_1-transformers` for Image-Text Tasks

## Step 1: Model Selection
Use the original `xtuner/llava-llama-3-8b-v1_1-transformers` model which includes the vision tower. You have two options:
- Download the model and place it in the `models/LLM` folder.
- Rely on the auto-download mechanism.

**Note:** It's recommended to offload the text encoder since the vision tower requires additional VRAM.

## Step 2: Load and Connect Image
- Use the comfy native node to load the image.
- Connect the loaded image to the `Hunyuan TextImageEncode` node.
  - You can connect up to 2 images to this node.

## Step 3: Prompting with Images
- Reference the image in your prompt by including `<image>`.
- The number of `<image>` tags should match the number of images provided to the sampler.
  - Example prompt: `Describe this <image> in great detail.`

You can also choose to give CLIP a prompt that does not reference the image separately.

## Step 4: Advanced Configuration - `image_token_selection_expression`
This expression is for advanced users and serves as a boolean mask to select which part of the image hidden state will be used for conditioning. Here are some details and recommendations:

- The hidden state sequence length (or number of tokens) per image in llava-llama-3 is 576.
- The default setting is `::4`, meaning every four tokens, one token goes into conditioning, interleaved, resulting in 144 tokens per image.
- Generally, more tokens lean more towards the conditional image.
- However, too many tokens (especially if the overall token count exceeds 256) will degrade generation quality. It's recommended not to use more than half the tokens (`::2`).
- Interleaved tokens generally perform better, but you might also want to try the following expressions:
  - `:128` - First 128 tokens.
  - `-128:` - Last 128 tokens.
  - `:128, -128:` - First 128 tokens and last 128 tokens.
- With a proper prompting strategy, even not passing in any image tokens (leaving the expression blank) can yield decent effects.

# Update

Scaled dot product attention (sdpa) should now be working (only tested on Windows, torch 2.5.1+cu124 on 4090), sageattention is still recommended for speed, but should not be necessary anymore making installation much easier.

Vid2vid test:
[source video](https://www.pexels.com/video/a-4x4-vehicle-speeding-on-a-dirt-road-during-a-competition-15604814/)

https://github.com/user-attachments/assets/12940721-4168-4e2b-8a71-31b4b0432314


text2vid (old test):

https://github.com/user-attachments/assets/3750da65-9753-4bd2-aae2-a688d2b86115


Transformer and VAE (single files, no autodownload):

https://huggingface.co/Kijai/HunyuanVideo_comfy/tree/main

Go to the usual ComfyUI folders (diffusion_models and vae)

LLM text encoder (has autodownload):

https://huggingface.co/Kijai/llava-llama-3-8b-text-encoder-tokenizer

Files go to `ComfyUI/models/LLM/llava-llama-3-8b-text-encoder-tokenizer`

Clip text encoder (has autodownload)

Either use any Clip_L model supported by ComfyUI by disabling the clip_model in the text encoder loader and plugging in ClipLoader to the text encoder node, or 
allow the autodownloader to fetch the original clip model from:

https://huggingface.co/openai/clip-vit-large-patch14, (only need the .safetensor from the weights, and all the config files) to:

`ComfyUI/models/clip/clip-vit-large-patch14`

Memory use is entirely dependant on resolution and frame count, don't expect to be able to go very high even on 24GB. 

Good news is that the model can do functional videos even at really low resolutions.
