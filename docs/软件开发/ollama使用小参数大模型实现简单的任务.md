---
tags: ["ollama", "大模型"]
---

# 实际意义

很多场景下都是简单的任务，如果都用大参数大模型去处理，并发、资源、耗时都是问题。

所以对于某些简单的任务，可以让小参数大模型去做，不要精确率多高，只要有效，哪怕百分之十，配上程序的逻辑判断，都可以为生产做贡献。

# 构建

## modelfile

```
FROM qwen3:0.6b
# 设置温度，较低的温度使输出更具确定性和一致性，适合分类任务
# 对于小模型，较低的温度有助于减少不相关或错误的输出
PARAMETER temperature 0.1

# 系统提示：明确告知模型其角色和任务，以及预期的输出类别
# 对于小模型，指令需要非常直接和清晰
SYSTEM 你是一个文本情绪分析助手。请将用户输入的文本分类为“积极”、“消极”或“中性”。

# 少样本提示：提供清晰的输入输出样例，帮助模型理解任务
# 样例要简单直接，符合小模型的理解能力
MESSAGE user 今天阳光明媚，我心情很好！
MESSAGE assistant 积极
MESSAGE user 我的项目失败了，感觉很难过。
MESSAGE assistant 消极
MESSAGE user 会议将在下午三点开始。
MESSAGE assistant 中性
MESSAGE user 这家餐厅的食物味道一般。
MESSAGE assistant 中性
```

## 构建

`ollama create emotion_cls -f ./Modelfile`

## 使用

```shell
curl http://localhost:11434/api/generate -d '{ "model": "emotion_cls","prompt":"哈哈哈哈", "stream": false}' | jq .
```

不使用思考模式

```shell
curl http://localhost:11434/api/generate -d '{ "model": "emotion_cls","prompt":"哈哈哈哈/no_think", "stream": false}' | jq .
```
