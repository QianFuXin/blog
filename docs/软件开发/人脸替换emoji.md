---
tags: ["人脸"]
---

# 人脸替换emoji

## 起源

偶然看到一个AI的项目：上传一张照片，用emoji来替换人脸，保护隐私。然后想到了一个古老的face_recognition包，好像也能实现。

## 环境

```shell
conda create -n  face_recognition_env python=3.11  -y
conda activate face_recognition_env
brew install cmake
pip install face_recognition
```

## 源码

[源码](https://gist.github.com/QianFuXin/8904a4ebeef974b2567d84baacfa514a)
