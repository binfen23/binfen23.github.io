---
title: cloudstudio IDE装双截棍flux
description: 
date: 2025-11-24 17:06:02 +0800
categories: []
tags: []
pin: false
math: false
mermaid: false
---
```shell


git config --global url."https://ghfast.top/https://github.com/".insteadof "https://github.com/"


mkdir -p ~/.pip && echo -e "[global]\nindex-url = https://mirrors.aliyun.com/pypi/simple/" > ~/.pip/pip.conf

pip uninstall torch torchvision torchaudio

pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu128

wget https://mirrors.aliyun.com/pytorch-wheels/cu128/torchaudio-2.7.1+cu128-cp310-cp310-manylinux_2_28_x86_64.whl
wget https://mirrors.aliyun.com/pytorch-wheels/cu128/torchvision-0.22.1+cu128-cp310-cp310-manylinux_2_28_x86_64.whl
wget https://mirrors.aliyun.com/pytorch-wheels/cu128/torch-2.7.1+cu128-cp310-cp310-manylinux_2_28_x86_64.whl


pip install torch-2.7.1+cu128-cp310-cp310-manylinux_2_28_x86_64.whl torchvision-0.22.1+cu128-cp310-cp310-manylinux_2_28_x86_64.whl torchaudio-2.7.1+cu128-cp310-cp310-manylinux_2_28_x86_64.whl


git clone "https://github.com/comfyanonymous/ComfyUI.git"


pip install -r ComfyUI/requirements.txt


curl -sSL https://ngrok-agent.s3.amazonaws.com/ngrok.asc \
  | tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null \
  && echo "deb https://ngrok-agent.s3.amazonaws.com buster main" \
  | tee /etc/apt/sources.list.d/ngrok.list \
  && apt update \
  && apt install ngrok



git clone https://github.com/Comfy-Org/ComfyUI-Manager.git ComfyUI/custom_nodes/ComfyUI-Manager
pip install -r "ComfyUI/custom_nodes/ComfyUI-Manager/requirements.txt"


wget "https://ghfast.top/https://github.com/mit-han-lab/nunchaku/releases/download/v0.3.1/nunchaku-0.3.1+torch2.7-cp310-cp310-linux_x86_64.whl"
pip install nunchaku-0.3.1+torch2.7-cp310-cp310-linux_x86_64.whl


apt install -y libgl1


pip install xformers

pip install apex

wget "https://hf-mirror.com/comfyanonymous/flux_text_encoders/resolve/main/clip_l.safetensors"  -O ComfyUI/models/clip/clip_l.safetensors
wget "https://hf-mirror.com/comfyanonymous/flux_text_encoders/resolve/main/t5xxl_fp8_e4m3fn.safetensors"  -O ComfyUI/models/clip/t5xxl_fp8_e4m3fn.safetensors
wget "https://hf-mirror.com/black-forest-labs/FLUX.1-dev/resolve/main/ae.safetensors" -O ComfyUI/models/vae/ae.safetensors
wget "https://www.modelscope.cn/models/AI-ModelScope/FLUX.1-dev/resolve/master/ae.safetensors" -O ComfyUI/models/vae/ae.safetensors
wget "https://hf-mirror.com/alimama-creative/FLUX.1-Turbo-Alpha/resolve/main/diffusion_pytorch_model.safetensors"  -O ComfyUI/models/loras/FLUX.1-Turbo-Alpha.safetensors


git clone https://github.com/mit-han-lab/ComfyUI-nunchaku.git ComfyUI/custom_nodes/nunchaku_nodes
pip install -r "ComfyUI/custom_nodes/nunchaku_nodes/requirements.txt"

git clone https://github.com/AIGODLIKE/AIGODLIKE-ComfyUI-Translation.git ComfyUI/custom_nodes/AIGODLIKE-ComfyUI-Translation

mkdir ComfyUI/models/diffusion_models/svdq-int4-flux.1-dev && 
wget "https://modelscope.cn/models/Lmxyy1999/svdq-int4-flux.1-dev/resolve/master/comfy_config.json" -O ComfyUI/models/diffusion_models/svdq-int4-flux.1-dev/comfy_config.json && 
wget "https://modelscope.cn/models/Lmxyy1999/svdq-int4-flux.1-dev/resolve/master/config.json" -O ComfyUI/models/diffusion_models/svdq-int4-flux.1-dev/config.json && 
wget "https://modelscope.cn/models/Lmxyy1999/svdq-int4-flux.1-dev/resolve/master/transformer_blocks.safetensors" -O ComfyUI/models/diffusion_models/svdq-int4-flux.1-dev/transformer_blocks.safetensors && 
wget "https://modelscope.cn/models/Lmxyy1999/svdq-int4-flux.1-dev/resolve/master/unquantized_layers.safetensors" -O ComfyUI/models/diffusion_models/svdq-int4-flux.1-dev/unquantized_layers.safetensors



ngrok authtoken 2yzdHnkQEEvgMQDGbgWX0JRnul5_yAq3iDGkPuu45wp31uDs



ngrok http --url=optimal-pup-funky.ngrok-free.app 8188



python ComfyUI/main.py --listen 0.0.0.0 --port 8188



mkdir ComfyUI/models/diffusion_models/svdq-int4-flux.1-fill-dev && 
wget "https://hf-mirror.com/mit-han-lab/svdq-int4-flux.1-fill-dev/resolve/main/transformer_blocks.safetensors" -O ComfyUI/models/diffusion_models/svdq-int4-flux.1-fill-dev/transformer_blocks.safetensors && 
wget "https://hf-mirror.com/mit-han-lab/svdq-int4-flux.1-fill-dev/resolve/main/comfy_config.json" -O ComfyUI/models/diffusion_models/svdq-int4-flux.1-fill-dev/comfy_config.json && 
wget "https://hf-mirror.com/mit-han-lab/svdq-int4-flux.1-fill-dev/resolve/main/config.json" -O ComfyUI/models/diffusion_models/svdq-int4-flux.1-fill-dev/config.json && 
wget "https://hf-mirror.com/mit-han-lab/svdq-int4-flux.1-fill-dev/resolve/main/unquantized_layers.safetensors" -O ComfyUI/models/diffusion_models/svdq-int4-flux.1-fill-dev/unquantized_layers.safetensors


wget "https://www.modelscope.cn/models/black-forest-labs/FLUX.1-Redux-dev/resolve/master/flux1-redux-dev.safetensors" -O ComfyUI/models/style_models/flux1-redux-dev.safetensors

mkdir ComfyUI/models/clip/siglip-so400m-patch14-384
wget "https://cnb.cool/binfen23/comfyui_flux1/-/lfs/adc04928d8fd19a61822584fe0cf2e813e5ebac17f3e49fb1ea096860ae6457b?name=config.json" -O ComfyUI/models/clip/siglip-so400m-patch14-384/config.json
wget "https://cnb.cool/binfen23/comfyui_flux1/-/lfs/ea2abad2b7f8a9c1aa5e49a244d5d57ffa71c56f720c94bc5d240ef4d6e1d94a?name=model.safetensors" -O ComfyUI/models/clip/siglip-so400m-patch14-384/model.safetensors
wget "https://cnb.cool/binfen23/comfyui_flux1/-/lfs/f59da2f87c3cd079bd4f8f3037e81b277c60c498e279a8020331f67a5a3157e8?name=preprocessor_config.json" -O ComfyUI/models/clip/siglip-so400m-patch14-384/preprocessor_config.json
wget "https://cnb.cool/binfen23/comfyui_flux1/-/lfs/2b6a1ff67a27e0df9ac0c7d93250fc0d87431c7b366b3d5669217104f9088a26?name=special_tokens_map.json" -O ComfyUI/models/clip/siglip-so400m-patch14-384/special_tokens_map.json
wget "https://cnb.cool/binfen23/comfyui_flux1/-/lfs/1e5036bed065526c3c212dfbe288752391797c4bb1a284aa18c9a0b23fcaf8ec?name=spiece.model" -O ComfyUI/models/clip/siglip-so400m-patch14-384/spiece.model
wget "https://cnb.cool/binfen23/comfyui_flux1/-/lfs/c6e405cb7c670d56636a9402c81023a55bc6c3c53d89cf02b92f5c5005bfe920?name=tokenizer.json" -O ComfyUI/models/clip/siglip-so400m-patch14-384/tokenizer.json
wget "https://cnb.cool/binfen23/comfyui_flux1/-/lfs/d6423dae508cc3a129d22ea443841c111832a1a73125b8f25ea8736951698bcb?name=tokenizer_config.json" -O ComfyUI/models/clip/siglip-so400m-patch14-384/tokenizer_config.json```