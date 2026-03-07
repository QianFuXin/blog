---
tags: ["electron"]
---

# 使用electron开发窗体并打包

## 环境准备

```shell
#设置代理
set ELECTRON_GET_USE_PROXY=1
set GLOBAL_AGENT_HTTP_PROXY=http://proxy.example.com:7890
set GLOBAL_AGENT_HTTPS_PROXY=https://proxy.example.com:7890
# Clone this repository
git clone https://github.com/electron/electron-quick-start
# Go into the repository
cd electron-quick-start
# Install dependencies
npm install
```

## 可能会卡住（探究一下为什么）

    `npm install --save-dev @electron-forge/cli`

## 运行

    `npm start`

## 打包

```shell
npx electron-forge import
npm run make
```
