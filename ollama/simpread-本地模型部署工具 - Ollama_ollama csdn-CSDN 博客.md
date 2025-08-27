> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u012365780/article/details/144987115)

## 目录

- [一、基于 Ollama 部署本地开源大模型](#一基于-ollama-部署本地开源大模型)
  - [1. Ollama 本地安装](#1-ollama-本地安装)
  - [2. Ollama 终端交互](#2-ollama-终端交互)
  - [3. 使用 API 服务访问](#3-使用-api-服务访问)
  - [4. 常用命令](#4-常用命令)
  - [5. 常见问题](#5-常见问题)
    - [5.1 设置本地存储模型路径（MacOS）](#51-设置本地存储模型路径macos)
    - [5.2 设置 API 请求 HOST](#52-设置-api-请求-host)
    - [5.3 如何指定上下文窗口大小？](#53-如何指定上下文窗口大小)

# 本地模型部署工具 - Ollama

### 一、基于 Ollama 部署本地开源大模型

> 如果需要部署本地开源大模型并将其用于提供推理服务，那么推荐使用 Ollama 这款工具来作为大模型运行与推理的管理工具。
> 
> 好处：
> 
> *   安装、使用简单。
> *   可以利用普通 CPU 进行推理（也支持 GPU 推理）。

**官网地址：** https://ollama.com/

#### 1. Ollama 本地安装

![](https://i-blog.csdnimg.cn/img_convert/4e615381437ed415e7e5a62d0bdf5c8a.png)

安装完成。进入终端，使用如下命令，查看是否安装成功。

```
ollama --version

# 如果出现
# Warning: could not connect to a running Ollama instance
# Warning: client version is 0.1.48
# 安装的 ollama 未启动 点击启动即可

# 正常输出
# ollama version is 0.1.48
```

#### 2. Ollama 终端交互

**模型查询入口：**

![](https://i-blog.csdnimg.cn/img_convert/d5a3f64c944ca2f1d7655eddbd221606.png)

![](https://i-blog.csdnimg.cn/img_convert/021f5e44487cd37f38d2bf1a8c040790.png)

**拉取模型并运行：**

![](https://i-blog.csdnimg.cn/img_convert/36a9d0ee03518f09252c00ed3e42cd72.png)

**退出本地模型终端交互：**

```
/bye
```

#### 3. 使用 API 服务访问

> Ollama 默认[服务器](https://so.csdn.net/so/search?q=%E6%9C%8D%E5%8A%A1%E5%99%A8&spm=1001.2101.3001.7020)端口 `11434`， 可以通过[设置环境变量](https://so.csdn.net/so/search?q=%E8%AE%BE%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F&spm=1001.2101.3001.7020)的方式更改默认的服务端口。
> 
> ```
> OLLAMA_HOST=:11435 ollama serve
> ```

**验证 Ollama 的 API 服务器能否正常工作：**

```
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{'
    "model": "qwen2:0.5b",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "你好呀!"
      }
    ]
  }'
```

```
# 返回
{"id":"chatcmpl-19","object":"chat.completion","created":1736223721,"model":"qwen2:0.5b","system_fingerprint":"fp_ollama","choices":[{"index":0,"message":{"role":"assistant","content":"你好，很高兴为你提供帮助。有什么我可以帮到您的？"},"finish_reason":"stop"}],"usage":{"prompt_tokens":22,"completion_tokens":14,"total_tokens":36}}
```

#### 4. 常用命令

```
# 拉取模型 用于更新本地模型。只会拉取 diff
ollama pull llama3.2

# 删除模型
ollama rm llama3.2

# 复制模型
ollama cp llama3.2 my-model

# 显示模型信息
ollama show llama3.2

# 列出计算机上的模型
ollama list

# 列出当前加载的模型
# 100% GPU means the model was loaded entirely into the GPU
# 100% CPU means the model was loaded entirely in system memory
# 48%/52% CPU/GPU means the model was loaded partially onto both the GPU and into system memory
ollama ps

# 停止当前正在运行的模型
ollama stop llama3.2

# 启动 Ollama  当您想在不运行桌面应用程序的情况下启动 ollama 时，可以使用 ollama serve。
ollama serve
```

##### 4.1 Ollama常用命令介绍

Ollama常用命令包括3类，分别是模型管理类、服务管理类、信息查询类

##### 4.1.1 模型管理类

###### `ollama pull <model-name>`
从在线仓库拉取模型。

示例：
```shell
ollama pull deepseek-r1:1.5b
```
意思是拉取 `deepseek-r1:1.5b` 模型。

###### `ollama create <custom-model-name> -f <Modelfile>`
根据 Modelfile 创建模型。

该命令通常用于离线安装模型。例如，当你在 Hugging Face 下载了一个 `deepseek-r1:8b` 的模型文件，可以通过以下命令进行离线安装：

```shell
echo "From ./deepseek-r1-8b.gguf" > modelfile-deepseek-r1-8b
ollama create deepseek-r1:8b -f modelfile-deepseek-r1-8b
```

###### `ollama list`
查看已下载的模型列表。

示例：
```shell
ollama list
```

###### `ollama run <model-name>`
运行指定模型。

示例：
```shell
ollama run deepseek-r1:1.5b
```
意思是运行 `deepseek-r1:1.5b` 模型。


######  `ollama cp <source-model-name> <new-model-name>` 
复制模型。

示例：
```shell
ollama cp deepseek-r1:1.5b my_model
```
意思是复制 `deepseek-r1:1.5b`，新模型名称为 `my_model`。


###### `ollama rm <model-name>`
删除模型。

示例：
```shell
ollama rm deepseek-r1:1.5b
```
意思是删除 `deepseek-r1:1.5b` 模型。


###### `ollama ps`
查看正在运行的模型。

示例：
```shell
ollama ps
```


###### `ollama stop <model-name>`
停止正在运行的模型。

示例：
```shell
ollama stop deepseek-r1:1.5b
```
意思是停止 `deepseek-r1:1.5b` 模型。

##### 4.1.2 服务管理类

### `ollama serve`
启动 Ollama 服务。

示例：
```shell
ollama serve
```
对于 Windows 来说通常会报错，提示端口占用，因为 Ollama 已经在运行。通常是开机自动运行。对于 Windows，想启动 Ollama 也无需使用命令行，像打开其他软件一样直接双击即可启动。


##### 4.1.3 信息查询类

###### `ollama -v`
查询 Ollama 版本信息。

示例：
```shell
ollama -v
```


###### `ollama show <model-name>`
查看模型的详细信息。

示例：
```shell
ollama show deepseek-r1:1.5b
```
表示查看 `deepseek-r1:1.5b` 模型的详细信息。


#### 5. 常见问题

参考：https://github.com/ollama/ollama/blob/main/docs/faq.md

##### 5.1 设置本地存储模型路径（MacOS）

```
# 设置
launchctl setenv OLLAMA_MODELS /Users/yangdong/tools/llm/models/ollama-models
# 读取
launchctl getenv OLLAMA_MODELS
# 卸载
launchctl unsetenv OLLAMA_MODELS

# 配置完成重启 ollama app
# 参考：https://dev.to/hamed0406/how-to-change-place-of-saving-models-on-ollama-4ko8
```

##### 5.2 设置 API 请求 HOST

```
# 设置
launchctl setenv OLLAMA_HOST "0.0.0.0:11434"
# 读取
launchctl getenv OLLAMA_HOST
```

##### 5.3 如何指定上下文窗口大小？

```
# 默认情况下，Ollama 使用的上下文窗口大小为 2048 个令牌。

# api 接口 参数配置
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{'
    "model": "qwen2:0.5b",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "你好呀!"
      }
    ],
    "options": {
      "num_ctx": 4096
    }
  }'
	
# ollama run /set parameter num_ctx 4096
# 参考：https://zhuanlan.zhihu.com/p/719200177 有详细原理讲述 
# 参考：https://blog.csdn.net/YiHanXii/article/details/143176687 实操相对简单
```