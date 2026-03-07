---
tags: ["linux", "clash"]
---

# 大概流程

首先配置web页面，便于后续通过web去切换线路和节点。

然后配置clash本体，获取配置文件并修改，最后启动程序。

# 配置clash

```shell
mkdir /etc/clash/  && cd /etc/clash/
# https://w2.v2free.top/ssr-download/clash-dashboard.tar.gz
tar zxvf clash-dashboard.tar.gz

mkdir  ~/.config/
mkdir  ~/.config/mihomo/
cd     ~/.config/mihomo/
# https://w2.v2free.top/ssr-download/clash-linux.tar.gz
tar xvf clash-linux.tar.gz
chmod +x clash-linux
wget -U "Mozilla/6.0" -O ~/.config/mihomo/config.yaml "https://v1.v2ai.top/link/IVEMbb14xsh6zW87?clash=2"
vim config.yaml
# 在配置文件中修改或增加以下内容；
external-controller: 127.0.0.1:9090 # 如果你不是从本机访问，需要从其它机器访问这个Clash Dashboard ,则改为：0.0.0.0:9090
external-ui: /etc/clash/clash-dashboard # clash-dashboard的路径；
secret: 'PaaRwW3B1Kj9' # PaaRwW3B1Kj9 是登录web管理界面的密码，请自行设置你自己的,不要照抄教程中的密码；
# 启动代理
nohup /clash-linux > clash.log 2>&1 &
# 访问对应的端口号选择代理:127.0.0.1:9090/ui
```
