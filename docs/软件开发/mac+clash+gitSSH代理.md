---
tags: ["mac", "clash", "ssh"]
---

# 起因

有一天使用git ssh 拉取代码，发现网络卡住，密钥已经录入GitHub，所以只可能是网络不通畅的原因。

# 解放方案

`vim ~/.ssh/config`

```shell
Host github.com
  HostName ssh.github.com
  Port 443
  User git
```
