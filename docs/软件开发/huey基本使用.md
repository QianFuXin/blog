---
tags: ["huey", "异步"]
---

# huey基本使用

- 异步任务队列

- 定时任务（类似 cron）

- 结果存储（可选）

- 重试、任务优先级

[源码](https://gist.github.com/QianFuXin/924d8de689c57911b8d609ab16516709)

开启消费者：`huey_consumer task.huey -w 2`
