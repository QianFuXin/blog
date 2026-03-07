---
tags: ["docker"]
---

# docker添加proxy

虽然添加了国内镜像，但有些时候，依然访问错误，避免小众问题发生，设置代理

## pull代理

    mkdir /etc/systemd/system/docker.service.d
    vim /etc/systemd/system/docker.service.d/http-proxy.conf

    [Service]
    Environment="HTTP_PROXY=http://127.0.0.1:7890"
    Environment="HTTPS_PROXY=http://127.0.0.1:7890"

    sudo systemctl daemon-reload
    sudo systemctl restart docker

## build代理

待定，暂时没有需求
