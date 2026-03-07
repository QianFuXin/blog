---
tags: ["qwen3", "vllm", "大模型"]
---

# 使用vllm部署qwen3

## 获取镜像

`vllm/vllm-openai:latest`

## 下载模型文件

`modelscope download --model 'Qwen/Qwen3-14b' --local_dir 'path/to/dir'`

## 启动容器

```shell
# 如果资源够，可以去掉--max-model-len 8192
docker run -d --gpus=all -v /data/hjh/Qwen3-14B-Int8-W8A16:/model/14b --network host --name qwen3 vllm/vllm-openai --model /model/14b --host 0.0.0.0 --port 11001 --max-model-len 8192
```

默认使用第一块卡，如果想使用其他卡：`-e CUDA_VISIBLE_DEVICES="3"`
