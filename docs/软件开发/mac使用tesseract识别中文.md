---
tags: ["mac", "ocr", "tesseract"]
---

# 安装

brew install tesseract

# 添加中文支持

下载包

<https://github.com/tesseract-ocr/tessdata/blob/main/chi_sim.traineddata>

![](/images/mac使用tesseract识别中文_1.png)

# 使用

![](/images/mac使用tesseract识别中文_2.png)

    tesseract picture_path out -l chi_sim
