---
tags: ["python", "PyPI"]
---

# 分享自己py代码到PyPI

### 准备条件

PyPI的账号

### 代码格式

```shell
filtered_flask/
|-- filtered_flask/
|   |-- __init__.py
|   |-- app.py
|-- tests/
|-- setup.py
|-- README.md
|-- MANIFEST.in
```

- filtered_flask/app.py

```python
from flask import Flask, Response

app = Flask(__name__)

# 装饰器用于全局信息过滤
@app.after_request
def global_filter(response):
    # 检查响应内容是否包含 "test"
    if b'test' in response.data:
        # 如果包含 "test"，可以采取过滤、修改或其他操作
        filtered_content = response.data.replace(b'test', b'filtered')
        response.set_data(filtered_content)

    return response

# 一个简单的路由返回包含 "test" 的内容
@app.route('/')
def home():
    return 'This is a test message.'

if __name__ == '__main__':
    app.run(debug=True)
```

- filtered_flask/**init**.py

```python
from .app import app
```

- setup.py

```python
from setuptools import setup, find_packages

setup(
name='filtered_flask',
version='0.1.0',
packages=find_packages(),
install_requires=[
'Flask',
],
classifiers=[
'Programming Language :: Python :: 3',
],
)
```

- MANIFEST.in

```python
include README.md
recursive-include filtered_flask/templates *
recursive-include filtered_flask/static *
```

- README.md

```
# filtered_flask
```

### 打包

在项目根目录下执行

```
python setup.py sdist bdist_whee
```

这将在 dist 目录下生成源码发行包（.tar.gz 文件）和 Wheel 包（.whl 文件）

### 上传到pypi

```python
pip install twine
# 输入你的账号密码
twine upload dist/*
```

### 安装和使用

`pip install filtered_flask`

```python
from filtered_flask import app
app.run()
```
