---
title: cloudstudio ide和cnb平台搭建comfyui
description: 
date: 2025-11-02 23:47:27 +0800
categories: []
tags: [comfyui,flux模型]
pin: false
math: false
mermaid: false
---
### <img src="https://cnb.cdn-go.cn/monorepo/latest/6bb3d146672db732/images/favicon.png"> **cnb平台：[https://cnb.cool](https://cnb.cool)**

### <img src="https://cs-res-1258344699.file.myqcloud.com/cloudstudio/new-dashboard/0d9632a5064b94e6e6b8cb2dd889f8789409759f/dist/assets/favicon.0ba056ba.ico"> **cloudstudio ide平台：[https://ide.cloud.tencent.com](https://ide.cloud.tencent.com)**

***

## CNB平台

首先创建一个空白的CNB公开仓库。

创建.cmb.yml 文件：

```yml
$:
  vscode:
    - runner:
        cpus: 16
        tags: cnb:arch:amd64:gpu:L40
      docker:
        image: docker.cnb.cool/binfen23/comfyui_bf/comfyui
      services:
        - vscode
        - docker
      stages:
        - name: 查看显卡
          script: /usr/bin/nvidia-smi
```

<span style="color: #ff0000">\*</span>后续需要将image替换成自己的docker镜像，后续会讲到。

直接提交到main分支。

点击分支-查看全部分支-创建新分支。分支名称随意命名，演示使用的分支名称为Dev，点击创建按钮。接着在Dev分支中，删除`.cnb.yml`文件，（main分支的不会被删除，因为后续需要用到）接着新建以下三个文件：

`Dockerfile` 、`build_push.sh` 这两个文件。

### Dockerfile:

```Dockerfile
FROM pytorch/pytorch:2.7.1-cuda12.8-cudnn9-devel


# 修改镜像源
RUN sed -i 's/archive.ubuntu.com/mirrors.cloud.tencent.com/g' /etc/apt/sources.list && \
    sed -i 's/security.ubuntu.com/mirrors.cloud.tencent.com/g' /etc/apt/sources.list && \
    sed -i 's/cn.archive.ubuntu.com/mirrors.cloud.tencent.com/g' /etc/apt/sources.list

# 设置时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

# 安装 ssh 服务，用于支持 VSCode 客户端通过 Remote-SSH 访问开发环境
RUN apt-get update && \
    apt-get install -y \
    git-lfs curl wget git axel nload libgl1-mesa-glx ffmpeg build-essential gcc g++ \
    htop unzip openssh-server lftp xdg-utils nano && \
    apt-get -y autoremove 

# 安装 code-server 和 vscode 常用插件
RUN curl -fsSL https://code-server.dev/install.sh | sh \
    && code-server --install-extension redhat.vscode-yaml \
    && code-server --install-extension dbaeumer.vscode-eslint \
    && code-server --install-extension eamodio.gitlens \
    && code-server --install-extension tencent-cloud.coding-copilot
# 指定字符集支持命令行输入中文（根据需要选择字符集）
ENV LANG C.UTF-8
ENV LANGUAGE C.UTF-8

# 安装 git-lfs
RUN git lfs install --force

# pip改全局清华源
RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# 更新 pip
RUN pip install --upgrade pip
RUN pip install -U "huggingface_hub[cli]"



# 清理缓存
RUN apt clean && rm -rf ~/.cache/pip && rm -rf /var/cache/apt/*
```

### build\_push.sh:

```shell
#!/bin/bash
# Docker构建和推送脚本

# 格式化时间显示函数
format_time() {
    local seconds=$1
    local hours=$((seconds / 3600))
    local minutes=$(( (seconds % 3600) / 60 ))
    local secs=$((seconds % 60))
    
    local time_str=""
    [ $hours -gt 0 ] && time_str="${hours}小时"
    [ $minutes -gt 0 ] && time_str="${time_str}${minutes}分钟"
    time_str="${time_str}${secs}秒"
    
    echo "$time_str"
}

# 开始总计时
start_time=$(date +%s)

# 检查变量是否设置
if [ -z "${CNB_DOCKER_REGISTRY}" ] || [ -z "${CNB_REPO_SLUG_LOWERCASE}" ]; then
    echo "错误：环境变量 CNB_DOCKER_REGISTRY 或 CNB_REPO_SLUG_LOWERCASE 未设置"
    exit 1
fi

# 构建镜像
echo "开始构建Docker镜像..."
build_start=$(date +%s)
docker build --no-cache -t ${CNB_DOCKER_REGISTRY}/${CNB_REPO_SLUG_LOWERCASE}/comfyui .
if [ $? -eq 0 ]; then
    build_end=$(date +%s)
    build_time=$((build_end - build_start))
    echo "Docker镜像构建完成，耗时: $(format_time $build_time)"
else
    echo "Docker镜像构建失败"
    exit 1
fi

# 推送镜像
echo "开始推送Docker镜像到仓库..."
push_start=$(date +%s)
docker push ${CNB_DOCKER_REGISTRY}/${CNB_REPO_SLUG_LOWERCASE}/comfyui
if [ $? -eq 0 ]; then
    push_end=$(date +%s)
    push_time=$((push_end - push_start))
    echo "Docker镜像推送完成，耗时: $(format_time $push_time)"
else
    echo "Docker镜像推送失败"
    exit 1
fi

# 计算总时间
end_time=$(date +%s)
total_time=$((end_time - start_time))

echo "所有操作执行完毕"
echo "构建时间: $(format_time $build_time)"
echo "推送时间: $(format_time $push_time)"
echo "总耗时: $(format_time $total_time)"
```

确认在Dev分支并且2个文件都创建完成之后，点击右上角的“云原生开发”，大约需要等待 2 分钟的创建时间，选 WebIDE 进入在线vscode编辑器在上面的导航栏选 Terminal - New Terminal 新建一个终端窗口。
赋予`build_push.sh` 这个文件执行权限

`chmod +x build_push.sh`

接着执行编译脚本，编译docker镜像并且上传到CNB平台以便后续调用`bash build_push.sh`

接着继续等待约11分钟的编译和推送镜像。
结束后，直接运行 `kill 1` 关闭本次开发环境，避免浪费时长

此时在仓库的 “制品” 中就可以看到编译好的docker镜像，点进去可以复制 “制品地址”，粘贴到main分支的`.cmb.yml`文件中的image里。

确定在main分支中的`.cmb.yml`文件已经修改了image的地址。

点击右上角的“原云生开发”选 WebIDE 进入在线vscode编辑器

输入 `nano .gitattributes` 添加以下的LFS跟踪

```
# 跟踪模型文件
*.ckpt filter=lfs diff=lfs merge=lfs -text
*.safetensors filter=lfs diff=lfs merge=lfs -text
*.gguf filter=lfs diff=lfs merge=lfs -text

# 跟踪图片文件
*.png filter=lfs diff=lfs merge=lfs -text
*.jpg filter=lfs diff=lfs merge=lfs -text
*.jpeg filter=lfs diff=lfs merge=lfs -text

# 跟踪视频文件
*.mp4 filter=lfs diff=lfs merge=lfs -text
*.mov filter=lfs diff=lfs merge=lfs -text

# 跟踪音频文件
*.mp3 filter=lfs diff=lfs merge=lfs -text
*.wav filter=lfs diff=lfs merge=lfs -text

# 跟踪压缩文件
*.zip filter=lfs diff=lfs merge=lfs -text
*.tar filter=lfs diff=lfs merge=lfs -text
*.7z filter=lfs diff=lfs merge=lfs -text
*.bz2 filter=lfs diff=lfs merge=lfs -text
*.gz filter=lfs diff=lfs merge=lfs -text
*.xz filter=lfs diff=lfs merge=lfs -text
*.zst filter=lfs diff=lfs merge=lfs -text

# 跟踪二进制文件
*.bin filter=lfs diff=lfs merge=lfs -text
*.so filter=lfs diff=lfs merge=lfs -text
*.dylib filter=lfs diff=lfs merge=lfs -text
*.dll filter=lfs diff=lfs merge=lfs -text

# 跟踪其他大文件
*.pt filter=lfs diff=lfs merge=lfs -text
*.pth filter=lfs diff=lfs merge=lfs -text
*.patch filter=lfs diff=lfs merge=lfs -text
*.onnx filter=lfs diff=lfs merge=lfs -text
*.whl filter=lfs diff=lfs merge=lfs -text
*.arrow filter=lfs diff=lfs merge=lfs -text
*.ftz filter=lfs diff=lfs merge=lfs -text
*.h5 filter=lfs diff=lfs merge=lfs -text
*.joblib filter=lfs diff=lfs merge=lfs -text
*.lfs.* filter=lfs diff=lfs merge=lfs -text
*.mlmodel filter=lfs diff=lfs merge=lfs -text
*.model filter=lfs diff=lfs merge=lfs -text
*.msgpack filter=lfs diff=lfs merge=lfs -text
*.npy filter=lfs diff=lfs merge=lfs -text
*.npz filter=lfs diff=lfs merge=lfs -text
*.ot filter=lfs diff=lfs merge=lfs -text
*.parquet filter=lfs diff=lfs merge=lfs -text
*.pb filter=lfs diff=lfs merge=lfs -text
*.pickle filter=lfs diff=lfs merge=lfs -text
*.pkl filter=lfs diff=lfs merge=lfs -text
*.rar filter=lfs diff=lfs merge=lfs -text
*.tar.* filter=lfs diff=lfs merge=lfs -text
*.tflite filter=lfs diff=lfs merge=lfs -text
*.tgz filter=lfs diff=lfs merge=lfs -text
*.wasm filter=lfs diff=lfs merge=lfs -text
*tfevents* filter=lfs diff=lfs merge=lfs -text
**/*.so filter=lfs diff=lfs merge=lfs -text
venv/**/*.so filter=lfs diff=lfs merge=lfs -text
venv/lib/python3.11/site-packages/torch/** filter=lfs diff=lfs merge=lfs -text
venv/lib/python3.11/site-packages/nvidia/** filter=lfs diff=lfs merge=lfs -text
venv/lib/python3.11/site-packages/** filter=lfs diff=lfs merge=lfs -text
```

ctrl + x，输入Y回车

git add .gitattributes
git commit -m "add .gitattributes”

在上面的导航栏选 Terminal - New Terminal 新建一个终端窗口。

创建虚拟环境`python -m venv venv`

激活虚拟环境
`source venv/bin/activate`

安装comfyui
`git clone https://github.com/comfyanonymous/ComfyUI.git && cd ComfyUI`
安装依赖
`pip install -r requirements.txt`

更新pip
`pip install --upgrade pip`

安装comfyui管理器
`git clone https://github.com/Comfy-Org/ComfyUI-Manager.git  /workspace/ComfyUI/custom_nodes/ComfyUI-Manager && cd /workspace/ComfyUI/custom_nodes/ComfyUI-Manager`
安装依赖
`pip install -r requirements.txt`

创建一键启动脚本

`nano /workspace/start2.py`

```python
import os
import shutil
import subprocess
import time
import socket
import sys

def main():
    # 处理ComfyUI的.git文件夹恢复
    comfyui_dir = "/workspace/ComfyUI"
    git_dir = os.path.join(comfyui_dir, ".git")
    gitkeep = os.path.join(comfyui_dir, "gitkeep")
    
    if not os.path.isdir(git_dir) and os.path.isdir(gitkeep):
        shutil.move(gitkeep, git_dir)
        print("\033[32m✓\033[0m ComfyUI (已恢复.git文件夹)")

    # 检查自定义节点目录
    custom_nodes_dir = os.path.join(comfyui_dir, "custom_nodes")
    if not os.path.isdir(custom_nodes_dir):
        print(f"错误: 目录 {custom_nodes_dir} 不存在!")
        sys.exit(1)

    # 获取所有子文件夹
    folders = []
    for item in os.listdir(custom_nodes_dir):
        item_path = os.path.join(custom_nodes_dir, item)
        if os.path.isdir(item_path) and item != "__pycache__":
            folders.append((item, item_path))

    # 如果没有子文件夹
    if not folders:
        print(f"在 {custom_nodes_dir} 目录下未发现子文件夹。(ComfyUI还未安装任何插件)")

    # 遍历所有子文件夹检查gitkeep
    print(f"检查 {custom_nodes_dir} 目录下的文件夹是否包含 gitkeep 文件夹:")
    print("-" * 70)

    for folder_name, folder_path in folders:
        folder_gitkeep = os.path.join(folder_path, "gitkeep")
        folder_git = os.path.join(folder_path, ".git")

        if os.path.isdir(folder_gitkeep):
            shutil.move(folder_gitkeep, folder_git)
            print(f"\033[32m✓\033[0m {folder_name} (已还原.git文件夹)")
        else:
            if os.path.isdir(folder_git):
                print(f"\033[32m✓\033[0m {folder_name} (已还原过.git文件夹)")
            else:
                print(f"\033[31m✗\033[0m {folder_name} (缺失.git文件夹，会导致无法进行正常git控制)")

    print("-" * 70)
    print()

    # 启动ComfyUI
    port = 8188
    venv_activate = "/workspace/venv/bin/activate"
    
    # 构建并执行启动命令
    cmd = (
        f"source {venv_activate} && "
        f"nohup python {os.path.join(comfyui_dir, 'main.py')} "
        f"--listen 0.0.0.0 --port {port} "
        f"> /workspace/ComfyUI.log 2>&1 &"
    )
    
    subprocess.run(cmd, shell=True, executable="/bin/bash")
    
    print("\033[32m》》ComfyUI正在启动\033[0m")
    port_opened = False
    max_attempts = 30
    
    # 检查端口是否打开
    for _ in range(max_attempts):
        try:
            with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
                s.settimeout(1)
                s.connect(("localhost", port))
            port_opened = True
            break
        except (ConnectionRefusedError, TimeoutError, OSError):
            time.sleep(1)
    
    # 输出结果
    if port_opened:
        print("\033[32m✓ ComfyUI正在运行\033[0m")
        proxy_uri = os.getenv("CNB_VSCODE_PROXY_URI", f"http://localhost:{{port}}")
        url = proxy_uri.replace("{{port}}", str(port))
        print(f"\033[32m》》按住 Ctrl + 鼠标左键 点击链接打开：【 {url} 】\033[0m")
    else:
        print("\033[31m ✗ ComfyUI启动失败，请查看 ComfyUI.log 日志文件\033[0m")

if __name__ == "__main__":
    main()
```

再新建个shell脚本，不用打命令直接可以拖动到终端执行。
`nano `

```shell
#!/bin/bash
python /workspace/start2.py
```

赋予执行权限`chmod +x /workspace/start.sh`

创建结束脚本`nano /workspace/stop.sh`

```
#!/bin/bash

# 定义要结束的进程名称
TARGET_PROCESSES=("python")

echo "正在查找并结束以下进程：${TARGET_PROCESSES[*]}"

for PROC in "${TARGET_PROCESSES[@]}"; do
  # 使用ps查找对应的进程ID
  PIDS=$(ps -e -o pid=,comm= | grep -w "$PROC" | awk '{print $1}')
  
  for PID in $PIDS; do
    if [ "$PID" != "$$" ]; then
      echo "正在结束进程 $PROC (PID: $PID)"
      kill -9 "$PID"
    fi
  done
done

echo "进程已结束。"




# 定义要检查的目录
CUSTOM_NODES_DIR="/workspace/ComfyUI/custom_nodes"

# 检查目录是否存在
if [ ! -d "$CUSTOM_NODES_DIR" ]; then
    echo "错误: 目录 $CUSTOM_NODES_DIR 不存在!"
    exit 1
fi

# 获取所有子文件夹
folders=$(find "$CUSTOM_NODES_DIR" -mindepth 1 -maxdepth 1 -type d)

# 如果没有子文件夹，直接退出
if [ -z "$folders" ]; then
    echo "在 $CUSTOM_NODES_DIR 目录下未发现子文件夹。"
    exit 0
fi

# 遍历所有子文件夹
echo "检查 $CUSTOM_NODES_DIR 目录下的文件夹是否包含 .git 文件夹:"
echo "------------------------------------------------------------------------"


for folder in $folders; do
    folder_name=$(basename "$folder")
    if [ "$folder_name" = "__pycache__" ]; then
        continue
    fi
    if [ -d "$folder/.git" ]; then
        mv "$folder/.git" "$folder/gitkeep"
        echo -e "\033[32m✓\033[0m $folder_name (已备份.git文件夹)"
    else
        if [ -d "$folder/gitkeep" ]; then
                echo -e "\033[32m✓\033[0m $folder_name (已备份过.git文件夹)"
            else
                echo -e "\033[31m✗\033[0m $folder_name (缺失.git备份文件夹，会导致无法进行还原.git文件夹)"
        fi
    fi
done

echo "------------------------------------------------------------------------"

# 检查目录是否存在
if [ -d "/workspace/ComfyUI/.git" ]; then
    mv /workspace/ComfyUI/.git /workspace/ComfyUI/gitkeep
    echo -e "\033[32m✓\033[0m ComfyUI (已备份.git文件夹)"
fi

# 判断文件是否存在
if [ -f "/workspace/ComfyUI/.gitignore" ]; then
  sed -i -e '/^\/models\/$/d' -e '/^\/input\/$/d' -e '/^\/custom_nodes\/$/d' -e '/^\/user\/$/d' -e '/^web_custom_versions\/$/d' /workspace/ComfyUI/.gitignore
  echo -e "\033[32m✓\033[0m ComfyUI (已经修改.gitignore文件)"
fi
```

赋予执行权限`chmod +x /workspace/stop.sh`

创建同步脚本`nano /workspace/sync.sh`

```shell
#!/bin/bash

# 定义要检查的目录
CUSTOM_NODES_DIR="/workspace/ComfyUI/custom_nodes"

# 检查目录是否存在
if [ ! -d "$CUSTOM_NODES_DIR" ]; then
    echo "错误: 目录 $CUSTOM_NODES_DIR 不存在!"
    exit 1
fi

# 获取所有子文件夹
folders=$(find "$CUSTOM_NODES_DIR" -mindepth 1 -maxdepth 1 -type d)

# 如果没有子文件夹，直接退出
if [ -z "$folders" ]; then
    echo "在 $CUSTOM_NODES_DIR 目录下未发现子文件夹。"
    exit 0
fi

# 遍历所有子文件夹
echo "检查 $CUSTOM_NODES_DIR 目录下的文件夹是否包含 .git 文件夹:"
echo "------------------------------------------------------------------------"


for folder in $folders; do
    folder_name=$(basename "$folder")
    if [ "$folder_name" = "__pycache__" ]; then
        continue
    fi
    if [ -d "$folder/.git" ]; then
        mv "$folder/.git" "$folder/gitkeep"
        echo -e "\033[32m✓\033[0m $folder_name (已备份.git文件夹)"
    else
        if [ -d "$folder/gitkeep" ]; then
                echo -e "\033[32m✓\033[0m $folder_name (已备份过.git文件夹)"
            else
                echo -e "\033[31m✗\033[0m $folder_name (缺失.git备份文件夹，会导致无法进行还原.git文件夹)"
        fi
    fi
done

echo "------------------------------------------------------------------------"

# 检查目录是否存在
if [ -d "/workspace/ComfyUI/.git" ]; then
    mv /workspace/ComfyUI/.git /workspace/ComfyUI/gitkeep
    echo -e "\033[32m✓\033[0m ComfyUI (已备份.git文件夹)"
fi

# 判断文件是否存在
if [ -f "/workspace/ComfyUI/.gitignore" ]; then
  sed -i -e '/^\/models\/$/d' -e '/^\/input\/$/d' -e '/^\/custom_nodes\/$/d' -e '/^\/user\/$/d' -e '/^web_custom_versions\/$/d' /workspace/ComfyUI/.gitignore
  echo -e "\033[32m✓\033[0m ComfyUI (已经修改.gitignore文件)"
fi

if [ -d "/workspace/ComfyUI/gitkeep" ]; then
    if ! git diff --quiet || ! git diff --staged --quiet; then
        echo -e "\033[32m✓\033[0m ComfyUI (开始推送到CNB仓库)"
        git add .
        git commit -m "$(date '+%Y-%m-%d %H:%M:%S')"
        # 指定推送到 origin 远程仓库的 main 分支
        if git push origin main; then
            echo -e "\033[32m✓\033[0m ComfyUI (成功推送到CNB仓库的 main 分支)"
        else
            echo -e "\033[31m✗\033[0m ComfyUI (推送到CNB仓库的 main 分支失败)"
            exit 1
        fi
    else
        echo -e "\033[33m! 没有文件更新，跳过提交和推送\033[0m"
    fi
fi
```

赋予执行权限`chmod +x /workspace/sync.sh`

将sync.sh从左侧拖动到下方的终端，并且回车就执行了同步到CNB仓库的操作。然后就可以关闭开发环境结束计算时长了。
---

待补充cloudstudio ide安装comfyui的方法