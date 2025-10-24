---
title: 腾讯云cloud studio linux安装comfyui和flux模型
description: 
date: 2025-10-24 12:59:55 +0800
categories: []
tags: [cloudstudio,linux,comfyui,flux模型]
pin: false
math: false
mermaid: false
---


# cloudstudio安装comfyui步骤

## 安装依赖

执行 `apt update`

`apt install curl wget nano sudo python3 python3-pip git -y`

## 添加主机名到host
例如创建后的主机名是以下所示：
root@VM-0-10-ubuntu:/workspace#

然后执行sudo命令的时候提示:
sudo: unable to resolve host VM-0-10-ubuntu: Temporary failure in name resolution

需要在/etc/hosts 文件添加主机名的映射，地址是127.0.0.1

127.0.0.1 VM-0-10-ubuntu



## 设置git镜像
执行 `git config --global url."https://mirror.ghproxy.com/https://github.com/".insteadOf "https://github.com/"`

## 更改torch版本

执行 `pip uninstall -y torch && pip install torch torchvision torchaudio -f https://mirrors.aliyun.com/pytorch/wheels/cu124/torch_stable.html`

## 安装comfyui
执行 `git clone https://github.com/comfyanonymous/ComfyUI.git && cd ComfyUI`
执行 `pip install -r requirements.txt`

## 更新torch版本  
如果不更新，会提示 
> module 'torch' has no attribute 'float8_e4m3fn'

执行 `pip install --upgrade torch torchvision torchaudio`

## 下载 flux fp8模型
【flux1-dev-fp8.safetensors】

抱脸网：
https://huggingface.co/Kijai/flux-fp8/tree/main

抱脸网镜像站（国内下载）：
https://hf-mirror.com/Kijai/flux-fp8/tree/main

魔搭（国内下载）：
https://modelscope.cn/models/Kijai/flux-fp8/files

下载 flux1-dev-fp8.safetensors 到 ComfyUI/models/unet/目录内

curl -L --output ComfyUI/models/unet/flux1-dev-fp8.safetensors "https://modelscope.cn/models/Kijai/flux-fp8/resolve/master/flux1-dev-fp8.safetensors"


## 下载必要的 2个 clip 模型
【t5xxl_fp8_e4m3fn.safetensors】
【clip_l.safetensors】


抱脸网：
https://huggingface.co/comfyanonymous/flux_text_encoders/tree/main

抱脸网镜像站（国内下载）：
https://hf-mirror.com/comfyanonymous/flux_text_encoders/tree/main

魔搭（国内下载）：
https://modelscope.cn/models/comfyanonymous/flux_text_encoders/files

下载到 ComfyUI/models/clip/目录内

curl -L --output ComfyUI/models/clip/t5xxl_fp8_e4m3fn.safetensors "https://modelscope.cn/models/comfyanonymous/flux_text_encoders/resolve/master/t5xxl_fp8_e4m3fn.safetensors" && curl -L --output ComfyUI/models/clip/clip_l.safetensors "https://modelscope.cn/models/comfyanonymous/flux_text_encoders/resolve/master/clip_l.safetensors"  

## 下载必要的 vae 模型
【ae.safetensors】

抱脸网：
https://huggingface.co/black-forest-labs/FLUX.1-schnell/blob/main/ae.safetensors

抱脸网镜像站（国内下载）：
https://hf-mirror.com/black-forest-labs/FLUX.1-schnell/blob/main/ae.safetensors

魔搭（国内下载）：
https://modelscope.cn/models/black-forest-labs/FLUX.1-schnell/files

下载到 ComfyUI/models/vae/目录内
curl -L --output ComfyUI/models/vae/ae.safetensors "https://modelscope.cn/models/black-forest-labs/FLUX.1-schnell/resolve/master/ae.safetensors"

### 以下json复制到剪贴板，直接在comfyui的web界面粘贴，可以快速搭建文生图工作流

```json  

{
  "last_node_id": 17,
  "last_link_id": 18,
  "nodes": [
    {
      "id": 1,
      "type": "DualCLIPLoader",
      "pos": [
        60,
        400
      ],
      "size": [
        315,
        106
      ],
      "flags": {},
      "order": 0,
      "mode": 0,
      "inputs": [],
      "outputs": [
        {
          "name": "CLIP",
          "type": "CLIP",
          "links": [
            1
          ],
          "slot_index": 0
        }
      ],
      "properties": {
        "Node name for S&R": "DualCLIPLoader"
      },
      "widgets_values": [
        "t5xxl_fp8_e4m3fn.safetensors",
        "clip_l.safetensors",
        "flux"
      ]
    },
    {
      "id": 4,
      "type": "UNETLoader",
      "pos": [
        60,
        250
      ],
      "size": [
        315,
        82
      ],
      "flags": {},
      "order": 1,
      "mode": 0,
      "inputs": [],
      "outputs": [
        {
          "name": "MODEL",
          "type": "MODEL",
          "links": [
            3
          ]
        }
      ],
      "properties": {
        "Node name for S&R": "UNETLoader"
      },
      "widgets_values": [
        "flux1-dev-fp8.safetensors",
        "fp8_e4m3fn"
      ]
    },
    {
      "id": 3,
      "type": "KSampler",
      "pos": [
        1080,
        230
      ],
      "size": [
        315,
        262
      ],
      "flags": {},
      "order": 6,
      "mode": 0,
      "inputs": [
        {
          "name": "model",
          "type": "MODEL",
          "link": 3
        },
        {
          "name": "positive",
          "type": "CONDITIONING",
          "link": 15
        },
        {
          "name": "negative",
          "type": "CONDITIONING",
          "link": 5
        },
        {
          "name": "latent_image",
          "type": "LATENT",
          "link": 10
        }
      ],
      "outputs": [
        {
          "name": "LATENT",
          "type": "LATENT",
          "links": [
            11
          ],
          "slot_index": 0
        }
      ],
      "properties": {
        "Node name for S&R": "KSampler"
      },
      "widgets_values": [
        809490034853903,
        "randomize",
        20,
        1,
        "euler",
        "beta",
        1
      ]
    },
    {
      "id": 12,
      "type": "VAELoader",
      "pos": [
        1060,
        110
      ],
      "size": [
        315,
        58
      ],
      "flags": {},
      "order": 2,
      "mode": 0,
      "inputs": [],
      "outputs": [
        {
          "name": "VAE",
          "type": "VAE",
          "links": [
            12
          ]
        }
      ],
      "properties": {
        "Node name for S&R": "VAELoader"
      },
      "widgets_values": [
        "ae.safetensors"
      ]
    },
    {
      "id": 11,
      "type": "VAEDecode",
      "pos": [
        1440,
        130
      ],
      "size": [
        210,
        46
      ],
      "flags": {},
      "order": 7,
      "mode": 0,
      "inputs": [
        {
          "name": "samples",
          "type": "LATENT",
          "link": 11
        },
        {
          "name": "vae",
          "type": "VAE",
          "link": 12
        }
      ],
      "outputs": [
        {
          "name": "IMAGE",
          "type": "IMAGE",
          "links": [
            13
          ],
          "slot_index": 0
        }
      ],
      "properties": {
        "Node name for S&R": "VAEDecode"
      },
      "widgets_values": []
    },
    {
      "id": 5,
      "type": "ConditioningZeroOut",
      "pos": [
        480,
        600
      ],
      "size": [
        210,
        26
      ],
      "flags": {},
      "order": 5,
      "mode": 0,
      "inputs": [
        {
          "name": "conditioning",
          "type": "CONDITIONING",
          "link": 4
        }
      ],
      "outputs": [
        {
          "name": "CONDITIONING",
          "type": "CONDITIONING",
          "links": [
            5
          ],
          "slot_index": 0
        }
      ],
      "properties": {
        "Node name for S&R": "ConditioningZeroOut"
      },
      "widgets_values": []
    },
    {
      "id": 2,
      "type": "CLIPTextEncode",
      "pos": [
        440,
        320
      ],
      "size": [
        400,
        200
      ],
      "flags": {},
      "order": 4,
      "mode": 0,
      "inputs": [
        {
          "name": "clip",
          "type": "CLIP",
          "link": 1
        }
      ],
      "outputs": [
        {
          "name": "CONDITIONING",
          "type": "CONDITIONING",
          "links": [
            4,
            15
          ],
          "slot_index": 0
        }
      ],
      "properties": {
        "Node name for S&R": "CLIPTextEncode"
      },
      "widgets_values": [
        "a cat"
      ],
      "color": "#232",
      "bgcolor": "#353"
    },
    {
      "id": 10,
      "type": "EmptyLatentImage",
      "pos": [
        730,
        570
      ],
      "size": [
        315,
        106
      ],
      "flags": {},
      "order": 3,
      "mode": 0,
      "inputs": [],
      "outputs": [
        {
          "name": "LATENT",
          "type": "LATENT",
          "links": [
            10
          ]
        }
      ],
      "properties": {
        "Node name for S&R": "EmptyLatentImage"
      },
      "widgets_values": [
        1024,
        1024,
        1
      ],
      "color": "#432",
      "bgcolor": "#653"
    },
    {
      "id": 13,
      "type": "SaveImage",
      "pos": [
        1440,
        250
      ],
      "size": [
        340,
        390
      ],
      "flags": {},
      "order": 8,
      "mode": 0,
      "inputs": [
        {
          "name": "images",
          "type": "IMAGE",
          "link": 13
        }
      ],
      "outputs": [],
      "properties": {},
      "widgets_values": [
        "ComfyUI"
      ]
    }
  ],
  "links": [
    [
      1,
      1,
      0,
      2,
      0,
      "CLIP"
    ],
    [
      3,
      4,
      0,
      3,
      0,
      "MODEL"
    ],
    [
      4,
      2,
      0,
      5,
      0,
      "CONDITIONING"
    ],
    [
      5,
      5,
      0,
      3,
      2,
      "CONDITIONING"
    ],
    [
      10,
      10,
      0,
      3,
      3,
      "LATENT"
    ],
    [
      11,
      3,
      0,
      11,
      0,
      "LATENT"
    ],
    [
      12,
      12,
      0,
      11,
      1,
      "VAE"
    ],
    [
      13,
      11,
      0,
      13,
      0,
      "IMAGE"
    ],
    [
      15,
      2,
      0,
      3,
      1,
      "CONDITIONING"
    ]
  ],
  "groups": [
    {
      "id": 1,
      "title": "文生图",
      "bounding": [
        50,
        40,
        1740,
        649.5999755859375
      ],
      "color": "#3f789e",
      "font_size": 24,
      "flags": {}
    }
  ],
  "config": {},
  "extra": {
    "ds": {
      "scale": 1,
      "offset": [
        154.72444490907253,
        131.80834405120413
      ]
    }
  },
  "version": 0.4
}
```


## 安装 GGUF 节点 
pip install --upgrade gguf

cd ComfyUI/custom_nodes

git clone https://github.com/city96/ComfyUI-GGUF



## flux fill 模型
【flux1-fill-dev-Q8_0.gguf】  （可以根据显卡的显存大小，下载接近的文件大小版本）

抱脸网：
https://huggingface.co/YarvixPA/FLUX.1-Fill-dev-gguf/tree/main

抱脸网镜像站（国内下载）：
https://hf-mirror.com/YarvixPA/FLUX.1-Fill-dev-gguf/tree/main

魔搭（国内下载）：
https://modelscope.cn/models/AI-ModelScope/FLUX.1-Fill-dev-GGUF/files

下载 flux1-fill-dev-Q8_0.gguf 到 ComfyUI/models/unet/目录内

curl -L --output ComfyUI/models/unet/flux1-fill-dev-Q8_0.gguf "https://modelscope.cn/models/AI-ModelScope/FLUX.1-Fill-dev-GGUF/resolve/master/flux1-fill-dev-fp16-Q8_0-GGUF.gguf"

### 待补充安装管理器等插件..