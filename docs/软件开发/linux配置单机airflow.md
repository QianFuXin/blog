---
tags: ["airflow", "linux"]
---

# 安装airflow

```shell
AIRFLOW_VERSION=3.0.2

# Extract the version of Python you have installed. If you're currently using a Python version that is not supported by Airflow, you may want to set this manually.
# See above for supported versions.
PYTHON_VERSION="$(python -c 'import sys; print(f"{sys.version_info.major}.{sys.version_info.minor}")')"

CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
# For example this would install 3.0.0 with python 3.9: https://raw.githubusercontent.com/apache/airflow/constraints-3.0.2/constraints-3.9.txt

uv pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

> 如果报错libcst==1.8.0没安装成功，且需要rust环境，则执行以下内容

```shell
# 国内环境
export RUSTUP_DIST_SERVER=https://mirrors.tuna.tsinghua.edu.cn/rustup
export RUSTUP_UPDATE_ROOT=https://mirrors.tuna.tsinghua.edu.cn/rustup/rustup
# 安装
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# 刷新
source $HOME/.cargo/env
# 验证
rustc --version
cargo --version
```

> 如果出现 could not compile pyo3` (lib)

```shell
sudo yum install python3-devel
```

# 配置

```shell
# 指定目录 export AIRFLOW_HOME=~/airflow（默认~/airflow）
# 启动单机服务（自动生成配置文件）
airflow standalone
# 修改配置文件 load_examples = False
# 删除除了cfg之外所有文件，然后重新执行airflow standalone
# 密码在同级目录的一个文件，账号是admin
```

# 第一个定时任务

在~/airflow/dags文件夹下面新建py文件，系统会自动扫描。

或者手动扫描：`airflow dags list`

[代码](https://gist.github.com/QianFuXin/15ab4a6ab2673721c55d88a4e981f1e4)
