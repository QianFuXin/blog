---
tags: ["rag", "LangChain", "ollama", "embeddings"]
---

# 实现rag的几种方式

## ollama提供embeddings、llm

- llm可以是ollama，或者接口为openai格式即可
- embeddings服务由ollama提供
- 向量数据库容器提供（cpu）

[源码](https://gist.github.com/QianFuXin/39ba41e29a58e7b9b1636db06d774237)

## ollama提供llm，容器提供embeddings

- llm可以是ollama，或者接口为openai格式即可
- embeddings服务由HuggingFaceEmbeddings提供
- 向量数据库容器提供（cpu）

[源码](https://gist.github.com/QianFuXin/0e94c3875574f0315af9fe58a7b9b04e)

## llm、embeddings、向量数据库均由第三方提供

[源码](https://gist.github.com/QianFuXin/78945087cab1e8321138025f307d570d)

## llm、embeddings、向量数据库、关键字搜索（es）、重排序、均由第三方提供

[源码](https://gist.github.com/QianFuXin/31549fb7cdef55c956de0aeff817c19c)
