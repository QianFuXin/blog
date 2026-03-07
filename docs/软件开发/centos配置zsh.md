---
tags: ["centos", "zsh"]
---

# centos配置zsh

## 安装zsh

```shell
yum -y install zsh git
```

## 切换为zsh

```shell
sudo chsh -s /bin/zsh
```

## 安装ohmyzsh

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## 使用插件

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
vim ~/.zshrc
```

```shell
plugins=(git web-search jsontools z vi-mode zsh-syntax-highlighting zsh-autosuggestions)
```

```shell
source ~/.zshrc
```
