---
tags: ["python", "conda"]
---

## python离线环境以及独立环境的几种方式

最近遇到`小可爱`的机器，没有外网，所以安装独立的python极为别扭

## 方法

- conda离线安装
  首先在联网的机器上，准备安装包，然后把安装包拷贝到离网机器后安装

  ```shell
  # 设置安装包位置
  pkgs_dirs:
    - /home/up/myconda/pkgs
  ```

  ```shell
  # 安装环境到安装包
  conda create -n py38_pytorch pytorch python==3.8.5 --download-only
  # 将安装包拷贝到指定位置，在离网机器上安装
  conda create -n offlinepy38 pytorch python==3.8.5 --offline
  ```

- conda环境打包
  ```shell
  conda install conda-pack
  conda pack -n tf-1.15.0-py3.6
  tar -xzf tf-1.15.0-py3.6.tar.gz -C tf-1.15.0-py3.6
  source tf-1.15.0-py3.6/bin/activate
  ```
- python venv
  ```shell
  python3 -m venv venv
  ```
