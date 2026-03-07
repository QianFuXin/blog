---
tags: ["LLaMA", "微调"]
---

# 使用LLaMA_factory进行模型微调

## 环境安装

> [参考](https://github.com/hiyouga/LLaMA-Factory/blob/main/README_zh.md)

```shell
conda create -n llama_factory python=3.10 -y
conda activate llama_factory
pip install --upgrade pip
# 设置清华源，可选
# pip config set global.index-url https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
git clone --depth 1 https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
# 遇到包冲突时，可使用 pip install --no-deps -e . 解决。
pip install -e ".[torch,metrics]"
```

## 环境验证

```shell
llamafactory-cli -h
```

## 启动web环境

```shell
USE_MODELSCOPE_HUB=1 llamafactory-cli webui
```

## 命令行

> [参考](https://llamafactory.readthedocs.io/zh-cn/latest/getting_started/merge_lora.html)
