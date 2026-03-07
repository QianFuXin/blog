---
tags: ["pinot", "mcp"]
---

# 部署

[链接](https://gist.github.com/QianFuXin/c6c5c1f9c67dc2866be863f8be06a8f6)

# 验证

打开 `http://localhost:9000/#/`

测试broker是否连接上 `curl http://localhost:9000/v2/brokers`
如果返回为空，有可能是因为开启容器的先后顺序导致。则 `docker restart pinot-broker`

# 创建表

`curl -X POST -H "Content-Type: application/json"   -d @schema.json   http://localhost:9000/schemas`

```
{
  "schemaName": "my_table",
  "dimensionFieldSpecs": [
    {"name": "id", "dataType": "STRING"},
    {"name": "name", "dataType": "STRING"}
  ],
  "metricFieldSpecs": [
    {"name": "age", "dataType": "INT"}
  ],
  "dateTimeFieldSpecs": [
    {
      "name": "ts",
      "dataType": "LONG",
      "format": "1:MILLISECONDS:EPOCH",
      "granularity": "1:MILLISECONDS"
    }
  ]
}
```

`curl -X POST -H "Content-Type: application/json"  -d @table.json http://localhost:9000/tables`

```
{
"tableName": "my_table",
"tableType": "OFFLINE",
"segmentsConfig": {
"timeColumnName": "ts",
"schemaName": "my_table",
"replication": "1"
},
"tableIndexConfig": {
"loadMode": "MMAP"
},
"tenants": {},
"metadata": {}
}
```

# 测试

[源码](https://github.com/QianFuXin/mcps/blob/main/mcp_pinot.py)
