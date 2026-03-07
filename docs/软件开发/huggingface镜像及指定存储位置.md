---
tags: ["HuggingFace"]
---

# huggingface镜像及指定存储位置

## 设置镜像

`HF_ENDPOINT=https://hf-mirror.com`

## 下载模型

`huggingface-cli download Qwen/Qwen2.5-1.5B-Instruct `

or

`huggingface-cli download Qwen/Qwen2.5-1.5B-Instruct --local-dir checkpoints/fish-speech-1.5`

## 设置模型根目录存储位置

`HF_HOME="xxx"`
