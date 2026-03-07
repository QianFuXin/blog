---
tags: ["定时任务"]
---

# 配置gocron

## 安装

[下载地址](https://github.com/ouqiang/gocron/releases)

需要gocron-node和gocron两个包

## systemd管理

`vim /etc/systemd/system/gocron-node.service`

```shell
[Unit]
Description=GoCron Node Service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/root/gocron/gocron-node-linux-amd64/gocron-node -allow-root -s 127.0.0.1:5921
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

`vim /etc/systemd/system/gocron.service`

```shell
[Unit]
Description=GoCron Node Service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/root/gocron/gocron-linux-amd64/gocron web --host 127.0.0.1
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

开机自启动

```shell
systemctl daemon-reload
systemctl enable gocron-node
systemctl enable gocron
systemctl start gocron-node
systemctl start gocron
```
