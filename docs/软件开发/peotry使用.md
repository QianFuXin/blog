---
tags: ["peotry"]
---

# Python 包管理与发布工具笔记

## 一、Poetry 基础操作

### 1. 创建项目

```bash
poetry new hello-poetry
cd hello-poetry
```

### 2. 添加逻辑（示例）

在 `hello_poetry/__init__.py` 中写入：

```python
def say_hello(name: str) -> str:
    return f"Hello, {name}!"
```

### 3. 打包

```bash
poetry build
```

### 4. 发布到 PyPI

```bash
poetry publish --build -u token -p pypi-xxx
```

---

## 二、Poetry 的定位

- **一体化工具**：依赖管理 + 打包发布。
- 类比前端的 **npm/yarn**：
  - 从 **创建项目 → 管理依赖 → 打包 → 发布** 一条龙。
- **目标**：替代 `pip + setuptools + virtualenv + twine` 的组合。

---

## 三、uv 的定位

- **开发者**：Astral。
- **特点**：
  - 超快（Rust 实现，比 pip/poetry/pip-tools 快几十倍）。
  - 关注 **依赖解析、安装、虚拟环境**。
- **不负责**：打包与发布。
- **类比**：pip/pipenv 的高速替代品。

---

## 四、对比总结

| 工具       | 核心定位                  | 覆盖范围                          | 类比       |
| ---------- | ------------------------- | --------------------------------- | ---------- |
| **Poetry** | 依赖管理 + 打包发布       | 项目创建 → 依赖管理 → 构建 → 发布 | npm / yarn |
| **uv**     | 超高速依赖安装 + 虚拟环境 | 依赖解析、安装、环境管理          | pip 极速版 |
