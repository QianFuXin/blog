---
tags: ["jupyter"]
---

# jupyter简单使用

## docker安装

```bash
version: '3'

services:
  jupyter-notebook:
    image: jupyter/minimal-notebook
    ports:
      - "8888:8888"

    volumes:
      - /Users/qianfuxin/dc/jupyter1:/home/jovyan/work
```

```bash
!pip install jupyterlab-language-pack-zh-CN
```

## 本机安装

### 使用 pip 安装

```bash
pip install jupyterlab
```

### 使用 conda 安装

```bash
conda install -c conda-forge jupyterlab notebook
```

---

### 2. 生成配置文件

```bash
jupyter notebook --generate-config
```

配置文件默认生成在：

```
~/.jupyter/jupyter_notebook_config.py
```

---

### 3. 设置密码

#### 进入 Python 环境

```python
from jupyter_server.auth import passwd
passwd()
```

系统会提示输入密码，并返回一串加密后的字符串，例如：

```
'sha1:abcd1234...'
```

---

### 4. 修改配置文件

编辑配置文件：

```bash
vim ~/.jupyter/jupyter_notebook_config.py
```

添加或修改以下内容：

```python
c.ServerApp.open_browser = False           # 启动时不自动打开浏览器
c.ServerApp.password = 'sha1:abcd1234...'  # 使用上一步生成的加密密码
c.ServerApp.ip = '0.0.0.0'                 # 允许外部访问
c.ServerApp.port = 8888                    # 指定端口
```

---

### 5. 安装中文语言包（可选）

```bash
pip install jupyterlab-language-pack-zh-CN
```

---

### 6. 后台运行 JupyterLab

使用 `nohup` 后台运行：

```bash
nohup jupyter lab --allow-root > jupyter.log 2>&1 &
```

日志会输出到 `jupyter.log` 文件。
