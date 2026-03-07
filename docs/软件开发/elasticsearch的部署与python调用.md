---
tags: ["elasticsearch"]
---

# elasticsearch的部署与python调用

## 部署

    version: '3'
    services:
      elasticsearch:
        image: docker.elastic.co/elasticsearch/elasticsearch:8.9.0
        container_name: elasticsearch
        environment:
          - discovery.type=single-node
          - ES_JAVA_OPTS=-Xms1g -Xmx1g  # 设置 JVM 堆内存为 1GB
          - xpack.security.enabled=true  # 启用安全功能
          - ELASTIC_PASSWORD=password  # 设置默认超级用户 elastic（用户名） 的密码
          - xpack.security.authc.api_key.enabled=true  # 启用 API 密钥认证
        ports:
          - "9200:9200"
          - "9300:9300"
        volumes:
          - /dc/es_data:/usr/share/elasticsearch/data

## python调用

    from elasticsearch import Elasticsearch

    # 使用用户名和密码连接到 Elasticsearch
    es = Elasticsearch(
        "http://localhost:9200",
        basic_auth=("elastic", "password")  # 替换为你的用户名和密码
    )

    # 创建索引
    index_name = "my_index"
    if not es.indices.exists(index=index_name):
        es.indices.create(index=index_name)

    # 添加文档
    doc = {
        "title": "Hello World",
        "content": "This is a test document in Elasticsearch",
        "timestamp": "2024-08-06"
    }
    es.index(index=index_name, id=1, body=doc)

    # 查询文档
    res = es.get(index=index_name, id=1)
    print(res['_source'])

    # 搜索文档
    search_query = {
        "query": {
            "match": {
                "content": "test"
            }
        }
    }
    search_results = es.search(index=index_name, body=search_query)
    print(search_results['hits']['hits'])

## 总结

最初看到es的文档，下意识想到了mongodb，然后查了一下，既然都是存储文档，那么为啥要用es，结论是mongo旨在数据更新，es旨在数据查询。其中es的索引类似sql中的表概念，文档类似行概念
