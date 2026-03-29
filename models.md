# Загрузка необходимых моделей

```sh
export HF_AUTH_TOKEN='<secret>'

# clip модельки
mkdir clip/
# https://huggingface.co/comfyanonymous/flux_text_encoders/tree/main
wget https://huggingface.co/comfyanonymous/flux_text_encoders/resolve/main/clip_l.safetensors --directory-prefix clip/
# https://huggingface.co/comfyanonymous/flux_text_encoders/tree/main
wget https://huggingface.co/comfyanonymous/flux_text_encoders/resolve/main/t5xxl_fp8_e4m3fn.safetensors --directory-prefix clip/

# vae модельки
mkdir vae/
# https://huggingface.co/OreX/Models/blob/main/Flux-Main/Flux-vae.safetensors
wget https://huggingface.co/OreX/Models/resolve/main/Flux-Main/Flux-vae.safetensors --directory-prefix vae/

# controlnet модельки
mkdir controlnet/
# https://huggingface.co/Shakker-Labs/FLUX.1-dev-ControlNet-Union-Pro-2.0/tree/main
wget https://huggingface.co/Shakker-Labs/FLUX.1-dev-ControlNet-Union-Pro-2.0/resolve/main/diffusion_pytorch_model.safetensors --directory-prefix controlnet/

# diffusion_models модельки
mkdir diffusion_models/
# https://huggingface.co/black-forest-labs/FLUX.1-dev
wget --header="Authorization: Bearer $HF_AUTH_TOKEN" https://huggingface.co/black-forest-labs/FLUX.1-dev/resolve/main/flux1-dev.safetensors --directory-prefix diffusion_models/
# https://huggingface.co/boricuapab/flux1-fill-dev-fp8
wget --header="Authorization: Bearer $HF_AUTH_TOKEN" https://huggingface.co/boricuapab/flux1-fill-dev-fp8/resolve/main/flux1-fill-dev-fp8.safetensors --directory-prefix diffusion_models/

# upscale_models модельки
mkdir upscale_models/
# https://huggingface.co/Kim2091/UltraSharp
wget --header="Authorization: Bearer $HF_AUTH_TOKEN" https://huggingface.co/Kim2091/UltraSharp/resolve/main/4x-UltraSharp.safetensors --directory-prefix upscale_models/

# loras модельки
mkdir loras/
# https://civitai.com/models/1685169/casual-game-art
wget -O SBG_quality_2_big.safetensors 'https://civitai.com/api/download/models/1907270?type=Model&format=SafeTensor' --directory-prefix loras/
```
