# Stable Diffusion WebUI 安装指南

## 前提条件
在开始安装之前，请确保您的系统满足以下要求：
- 操作系统：macOS
- Python 3.10 或更高版本
- Git

## 安装步骤

### 1. 克隆仓库
首先，克隆 Stable Diffusion WebUI 的 GitHub 仓库：
```bash
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
cd stable-diffusion-webui
```

### 2. 创建虚拟环境
创建并激活一个新的 Python 虚拟环境：
```bash
python -m venv venv
source venv/bin/activate  # 对于 Windows 系统，使用 `venv\Scripts\activate`
```

### 3. 安装依赖
使用 pip 安装所需的 Python 依赖：
```bash
pip install -r requirements.txt
```

### 4. 下载模型
下载 Stable Diffusion 模型并将其放置在 `models/ldm/stable-diffusion-v1/` 目录下。
克隆完 stable diffusion webui 之后，就可以下载模型，这里以 stable diffusion 2.0 训练模型为例。复制下面链接在浏览器打开：

```bash
https://huggingface.co/stabilityai/stable-diffusion-2-1/tree/main
```
也可以下载其他模型，具体可以在这里下载：https://huggingface.co/stabilityai

#### 4.1 downloading-stable-diffusion-models

If you don't have any models to use, Stable Diffusion models can be downloaded from [Hugging Face](https://huggingface.co/models?pipeline_tag=text-to-image&sort=downloads). To download, click on a model and then click on the `Files and versions` header. Look for files listed with the ".ckpt" or ".safetensors" extensions, and then click the down arrow to the right of the file size to download them.

Some popular official Stable Diffusion models are:

*   [Stable DIffusion 1.4](https://huggingface.co/CompVis/stable-diffusion-v-1-4-original) ([sd-v1-4.ckpt](https://huggingface.co/CompVis/stable-diffusion-v-1-4-original/resolve/main/sd-v1-4.ckpt))
*   [Stable Diffusion 1.5](https://huggingface.co/stable-diffusion-v1-5/stable-diffusion-v1-5) ([v1-5-pruned-emaonly.safetensors](https://huggingface.co/stable-diffusion-v1-5/stable-diffusion-v1-5/resolve/main/v1-5-pruned-emaonly.safetensors))
*   [Stable Diffusion 1.5 Inpainting](https://huggingface.co/stable-diffusion-v1-5/stable-diffusion-inpainting) ([sd-v1-5-inpainting.ckpt](https://huggingface.co/stable-diffusion-v1-5/stable-diffusion-inpainting/resolve/main/sd-v1-5-inpainting.ckpt))

Stable Diffusion 2.0 and 2.1 require both a model and a configuration file, and image width & height will need to be set to 768 or higher when generating images:

*   [Stable Diffusion 2.0](https://huggingface.co/stabilityai/stable-diffusion-2) ([768-v-ema.ckpt](https://huggingface.co/stabilityai/stable-diffusion-2/resolve/main/768-v-ema.ckpt))
*   [Stable Diffusion 2.1](https://huggingface.co/stabilityai/stable-diffusion-2-1) ([v2-1_768-ema-pruned.ckpt](https://huggingface.co/stabilityai/stable-diffusion-2-1/resolve/main/v2-1_768-ema-pruned.ckpt))

For the configuration file, hold down the option key on the keyboard and click [here](https://github.com/Stability-AI/stablediffusion/raw/main/configs/stable-diffusion/v2-inference-v.yaml) to download `v2-inference-v.yaml` (it may download as `v2-inference-v.yaml.yml`). In Finder select that file then go to the menu and select `File` > `Get Info`. In the window that appears select the filename and change it to the filename of the model, except with the file extension `.yaml` instead of `.ckpt`, press return on the keyboard (confirm changing the file extension if prompted), and place it in the same folder as the model (e.g. if you downloaded the `768-v-ema.ckpt` model, rename it to `768-v-ema.yaml` and put it in `stable-diffusion-webui/models/Stable-diffusion` along with the model).

Also available is a [Stable Diffusion 2.0 depth model](https://huggingface.co/stabilityai/stable-diffusion-2-depth) ([512-depth-ema.ckpt](https://huggingface.co/stabilityai/stable-diffusion-2-depth/resolve/main/512-depth-ema.ckpt)). Download the `v2-midas-inference.yaml` configuration file by holding down option on the keyboard and clicking [here](https://github.com/Stability-AI/stablediffusion/raw/main/configs/stable-diffusion/v2-midas-inference.yaml), then rename it with the `.yaml` extension in the same way as mentioned above and put it in `stable-diffusion-webui/models/Stable-diffusion` along with the model. Note that this model works at image dimensions of 512 width/height or higher instead of 768.

### 5. 运行 WebUI
cd stable-diffusion-webui and then ./webui.sh to run the web UI. A Python virtual environment will be created and activated using venv and any remaining missing dependencies will be automatically downloaded and installed.
To relaunch the web UI process later, run ./webui.sh again. Note that it doesn't auto update the web UI; to update, run git pull before running ./webui.sh.
运行以下命令启动 WebUI：
```bash
python scripts/webui.py
```
### 6. 打开 stable-diffusion-webui 网页版

** 注意生成图片的过程中不要关闭终端（terminal）窗口，** 打开浏览器 (safari 或者 chrome) 后输入

http://127.0.0.1:7860，即可访问本地 stable diffusion webui 了。

### 7. 安装辅助
⚪ install from Official Market

Open Automatic1111 WebUI -> Click Tab "Extensions" -> Click Tab "Available" -> Find "[TiledDiffusion with Tiled VAE]" -> Click "Install"
⚪ install from git URL

Open Automatic1111 WebUI -> Click Tab "Extensions" -> Click Tab "Install from URL" -> type in https://github.com/pkuliyi2015/multidiffusion-upscaler-for-automatic1111.git -> Click "Install"
** 汉化插件（未更新了，可能不适用新版本了）
```bash
https://github.com/VinsonLaro/stable-diffusion-webui-chinese
```

## 常见问题
如果在安装过程中遇到问题，请参考以下资源：
- [官方文档](https://github.com/CompVis/stable-diffusion-webui/wiki)
- [GitHub Issues](https://github.com/CompVis/stable-diffusion-webui/issues)

## 结论
按照上述步骤，您应该能够成功安装并运行 Stable Diffusion WebUI。如果有任何问题，请随时在 GitHub 上提问。
