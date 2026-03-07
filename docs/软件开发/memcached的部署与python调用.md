---
tags: ["memcached", "python", "缓存"]
---

# 部署

```bash
version: '3.8'

services:
memcached:
image: memcached
container_name: memcached
ports:

- "11211:11211"
```

# python调用

    from pymemcache.client import base

    # 连接到 Memcached 服务
    client = base.Client(('localhost', 11211))

    # 设置一个键值对
    client.set('my_key', 'Hello, Memcached!')

    # 获取存储的值
    value = client.get('my_key')
    print(f'Value for my_key: {value.decode("utf-8")}')

    # 删除键值对
    client.delete('my_key')

    # 关闭连接
    client.close()

# 总结

memcached是一个很单纯的缓存服务（减少数据库压力），数据存放在内存，所以没必要本地映射，
而且不支持身份验证，所以需要在容器互联时使用，不应该让外部访问。
