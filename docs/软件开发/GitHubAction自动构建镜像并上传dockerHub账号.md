---
tags: ["GitHub", "docker"]
---

# GitHubAction自动构建镜像并上传dockerHub账号

## action文件

```bash
name: Docker Image CI

on:
push.sh:
branches: [ "main" ]
pull_request:
branches: [ "main" ]

jobs:

build:

runs-on: ubuntu-latest

steps:
- uses: actions/checkout@v3
- name: Build the Docker image
  run: docker build -t ${{ secrets.IMAGE_NAME }} .

- name: docker login
  run: docker login -u qianfuxin -p '${{ secrets.DOCKER_TOKEN }}'
- name: change tag
  run: docker tag ${{ secrets.IMAGE_NAME }}:latest qianfuxin/${{ secrets.IMAGE_NAME }}:latest
- name: push.sh image
  run: docker push.sh qianfuxin/${{ secrets.IMAGE_NAME }}:latest
```

## 配置页面

![](/images/GitHubAction自动构建镜像并上传dockerHub账号.png)
