---
tags: ["llm", "shell"]
---

# 用大模型对shell赋能

## 起因

由于最近shell输入命令比较多，经常忘记命令的细节。每次都要去问大模型，然后拷贝命令再执行，就想着能不能将两个步骤合二为一。

## 实现

- 采用了本地大模型

[源码](https://gist.github.com/QianFuXin/0b561de9a86a654e571550f6f7147f51)

## 使用方式（alias）

`vim ~/.zshrc`

`alias aishell="python /xx/main.py"`

`source ~/.zshrc`

`aishell "创建一个a文件" --run`
