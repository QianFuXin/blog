# Claw Code 完整使用指南

> 合并去重自以下4篇文章：使用指南、资料研究清单、官方文档中文翻译、REPL帮助说明。

---

## 一、项目概述

**Claw Code** 是一个用 Rust 实现的 CLI Agent / Coding Agent 框架，命令名是 **`claw`**。

核心能力：

- 终端里的 AI agent，支持交互式会话和单次 prompt
- 能调用工具：读写文件、grep、glob、bash、web 搜索等
- 支持 session、权限控制、skills、sub-agent、MCP 等能力
- 主实现位于仓库的 `rust/` 目录下

> **重要**：研究的是 `ultraworkers/claw-code` 当前主仓库。补充站点 `claw-code.codes` 内容多处指向 `instructkr/claw-code`，描述的是 Python + Rust 双层架构，与当前主仓库的现实不完全一致。实际跑起来应优先看官方仓库文档。

---

## 二、安装与构建

### 关键警告

> 不要用 `cargo install claw-code`！crates.io 上该名字对应的是一个废弃占位包，不会安装出正确的 `claw`。

### 正确方式：从源码构建

```bash
git clone https://github.com/ultraworkers/claw-code
cd claw-code/rust
cargo build --workspace
```

构建后可执行文件：`./target/debug/claw`（Windows：`.\target\debug\claw.exe`）

### 第一次运行：先跑健康检查

```bash
cd rust
./target/debug/claw doctor
```

或在交互模式下：

```bash
./target/debug/claw
/doctor
```

这一步会检查：API key 是否生效、provider 是否正确识别、基本运行环境是否正常。

---

## 三、模型与 Provider

### 支持的 Provider

| Provider | 环境变量 | 说明 |
|----------|---------|------|
| Anthropic | `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN` | 原生 Messages API |
| OpenAI-compatible | `OPENAI_API_KEY` + `OPENAI_BASE_URL` | 兼容 `/v1/chat/completions` |
| xAI | `XAI_API_KEY` | Grok 系列 |
| DashScope/Qwen | `DASHSCOPE_API_KEY` | `qwen/` 或 `qwen-` 前缀自动路由 |

### 内置模型别名

- `opus` → `claude-opus-4-6`
- `sonnet` → `claude-sonnet-4-6`
- `haiku` → `claude-haiku-4-5-20251213`

### Provider 自动路由规则

1. 模型以 `claude` 开头 → Anthropic
2. 模型以 `grok` 开头 → xAI
3. 模型以 `openai/` 或 `gpt-` 开头 → OpenAI-compatible
4. 模型以 `qwen/` 或 `qwen-` 开头 → DashScope
5. 模型名不明显 → 按环境变量推断（Anthropic → OpenAI → xAI → 默认 Anthropic）

> 多 provider 环境下**务必显式写模型前缀**，避免误路由。

### 各 Provider 配置示例

**Anthropic API Key：**

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
./target/debug/claw --model sonnet prompt "hello"
```

**Anthropic Bearer Token / 代理：**

```bash
export ANTHROPIC_BASE_URL="http://127.0.0.1:8080"
export ANTHROPIC_AUTH_TOKEN="local-dev-token"
./target/debug/claw --model "claude-sonnet-4-6" prompt "reply with ready"
```

**OpenAI 兼容接口：**

```bash
export OPENAI_BASE_URL="http://127.0.0.1:8000/v1"
export OPENAI_API_KEY="local-dev-token"
./target/debug/claw --model "qwen2.5-coder" prompt "reply with ready"
```

**OpenRouter：**

```bash
export OPENAI_BASE_URL="https://openrouter.ai/api/v1"
export OPENAI_API_KEY="sk-or-v1-..."
./target/debug/claw --model "openai/gpt-4.1-mini" prompt "summarize this repository"
```

**Ollama（本地模型）：**

```bash
export OPENAI_BASE_URL="http://127.0.0.1:11434/v1"
unset OPENAI_API_KEY
./target/debug/claw --model "llama3.2" prompt "summarize this repository in one sentence"
```

**DashScope / Qwen：**

```bash
export DASHSCOPE_API_KEY="sk-..."
./target/debug/claw --model "qwen/qwen-max" prompt "hello"
./target/debug/claw --model "qwen-plus" prompt "hello"
```

---

## 四、基本使用方式

### 四种启动模式

```bash
# 1. 交互式 REPL
./target/debug/claw

# 2. 单次 prompt
./target/debug/claw prompt "summarize this repository"

# 3. 简写模式
./target/debug/claw "explain rust/crates/runtime/src/lib.rs"

# 4. JSON 输出（适合脚本/自动化）
./target/debug/claw --output-format json prompt "status"
```

### 常用 CLI 命令

```bash
./target/debug/claw status
./target/debug/claw sandbox
./target/debug/claw agents
./target/debug/claw mcp
./target/debug/claw skills
./target/debug/claw system-prompt --cwd .. --date 2026-04-04
```

### CLI Flags

- `--model`：指定模型
- `--output-format`：输出格式
- `--permission-mode`：权限模式
- `--dangerously-skip-permissions`：跳过权限检查
- `--allowedTools`：限制可用工具
- `--resume`：恢复会话

---

## 五、REPL 斜杠命令参考

### 基本操作

| 操作 | 说明 |
|------|------|
| 上/下方向键 | 浏览历史输入 |
| Ctrl-R | 反向搜索历史 |
| Tab | 自动补全命令、模式和最近会话 |
| Ctrl-C | 清空当前输入 |
| Shift+Enter / Ctrl+J | 插入换行 |
| 自动保存 | 会话自动保存到 `.claw/sessions/<session-id>.jsonl` |

### 会话管理

| 命令 | 说明 |
|------|------|
| `/help` | 显示帮助 |
| `/status` | 查看会话状态 |
| `/compact` | 压缩历史减少上下文（不删除） |
| `/clear [--confirm]` | 清空当前会话，重新开始 |
| `/cost` | 查看累计 token 使用成本 |
| `/version` | 查看版本与构建信息 |
| `/stats` | 查看工作区和会话统计 |
| `/history [count]` | 查看对话历史摘要 |
| `/tokens` | 查看当前 token 数量 |
| `/cache` | 查看提示缓存统计 |
| `/resume <path>` | 加载保存的会话 |
| `/resume latest` | 恢复最近会话 |
| `/session list` | 列出已管理的会话 |
| `/session switch <id>` | 切换到指定会话 |
| `/session fork [branch]` | 从当前会话分叉新会话 |
| `/session delete <id> [--force]` | 删除指定会话 |

### 代码与工程辅助

| 命令 | 说明 |
|------|------|
| `/diff` | 查看工作区改动 |
| `/commit` | 生成 commit message 并执行 git commit |
| `/pr [context]` | 起草或创建 Pull Request |
| `/issue [context]` | 起草或创建 GitHub Issue |
| `/bughunter [scope]` | 扫描代码库中的潜在 bug |
| `/ultraplan [task]` | 针对任务执行深入的多步骤规划 |
| `/teleport <符号或路径>` | 按符号名或路径快速跳转 |
| `/export [file]` | 导出对话到文件 |
| `/init` | 为当前仓库创建 starter `CLAUDE.md` |
| `/review` | 代码审查 |
| `/security-review` | 安全审查 |

### 工具与插件

| 命令 | 说明 |
|------|------|
| `/mcp [list\|show <server>\|help]` | 查看/管理 MCP 服务 |
| `/agents [list\|help]` | 查看已配置代理 |
| `/skills [list\|install\|help\|<skill>]` | 查看/安装/调用技能 |
| `/plugin [list\|install\|enable\|disable\|uninstall\|update]` | 管理插件 |
| `/approve`（别名 `/yes`, `/y`） | 批准待执行的工具调用 |
| `/deny`（别名 `/no`, `/n`） | 拒绝待执行的工具调用 |

### 配置与调试

| 命令 | 说明 |
|------|------|
| `/model [model]` | 查看或切换模型 |
| `/permissions [mode]` | 查看或切换权限模式 |
| `/config [env\|hooks\|model\|plugins]` | 查看配置 |
| `/memory` | 查看已加载的 CLAUDE.md 指令 |
| `/providers` | 查看可用提供商 |
| `/sandbox` | 查看沙箱隔离状态 |
| `/debug-tool-call` | 重放上一次工具调用（调试） |
| `/doctor` | 诊断安装、配置和环境健康 |

### 日常开发推荐流程

```
/resume latest  →  /status  →  /diff  →  修改代码  →  /commit  →  /pr
```

---

## 六、权限模式

### 三种模式

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| `read-only` | 只读 | 审查代码、了解项目 |
| `workspace-write` | 可写工作区 | 修改文件、重构 |
| `danger-full-access` | 完全访问 | 高风险操作（谨慎） |

### 使用方式

```bash
# CLI 参数
./target/debug/claw --permission-mode read-only prompt "summarize Cargo.toml"
./target/debug/claw --permission-mode workspace-write prompt "update README.md"

# REPL 内切换
/permissions workspace-write
```

### 限制工具

```bash
./target/debug/claw --allowedTools read,glob "inspect the runtime crate"
```

### 安全原则

- 先从 `read-only` 开始
- 真要改文件再切 `workspace-write`
- 不要轻易使用 `danger-full-access`

---

## 七、Session 管理

会话保存在当前工作区的 `.claw/sessions/` 目录。

### 核心操作

```bash
# 恢复最近会话
./target/debug/claw --resume latest

# 恢复后直接执行命令
./target/debug/claw --resume latest /status /diff
```

### 容易混淆的命令

- `/resume latest`：从保存文件恢复最近一次会话内容
- `/session switch <id>`：切换到某个已管理的会话
- `/clear`：清空当前会话，**重新开始**（删除）
- `/compact`：**压缩**历史减少上下文负担（不删除）

---

## 八、配置文件

### 加载优先级（后者覆盖前者）

1. `~/.claw.json`
2. `~/.config/claw/settings.json`
3. `/.claw.json`
4. `/.claw/settings.json`
5. `/.claw/settings.local.json`

### 推荐实践

- 用户级默认设置：`~/.config/claw/settings.json`
- 项目级设置：`.claw/settings.json`
- 本机私有覆盖：`.claw/settings.local.json`

### 自定义模型别名

```json
{
  "aliases": {
    "fast": "claude-haiku-4-5-20251213",
    "smart": "claude-opus-4-6",
    "cheap": "grok-3-mini"
  }
}
```

使用：`./target/debug/claw --model fast prompt "hello"`

---

## 九、代理设置

支持标准环境变量，适用于 Anthropic、OpenAI-compatible、xAI-compatible：

```bash
export HTTPS_PROXY="http://proxy.example:3128"
export HTTP_PROXY="http://proxy.example:3128"
export NO_PROXY="localhost,127.0.0.1,.corp.example"
```

---

## 十、容器化使用

适合不想污染本机环境、干净测试、CI、远程环境。

```bash
# 构建镜像
docker build -t claw-code-dev -f Containerfile .

# 在容器里跑测试
docker run --rm -it \
  -v "$PWD":/workspace \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev \
  cargo test --workspace

# 打开容器 shell
docker run --rm -it \
  -v "$PWD":/workspace \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev
```

进入后可执行：`cargo build --workspace`、`cargo test --workspace` 等。

---

## 十一、Rust 工作区结构

主要 crate：

| Crate | 职责 |
|-------|------|
| `api` | provider client、streaming、预检查 |
| `commands` | slash command 定义与帮助文本 |
| `runtime` | session、config、permissions、MCP、prompt、auth |
| `rusty-claude-cli` | 主 CLI 二进制 |
| `tools` | 内建工具、技能解析、agent runtime |
| `telemetry` | 追踪与用量 |
| `plugins` | 插件管理 |

---

## 十二、常见坑

1. **`cargo install claw-code` 行不通**——要从源码构建
2. **`sk-ant-*` 误填入 `ANTHROPIC_AUTH_TOKEN`**——`ANTHROPIC_API_KEY` 用于 `sk-ant-*`，`ANTHROPIC_AUTH_TOKEN` 用于 bearer token
3. **模型名没带前缀导致误路由**——多 provider 环境务必显式写 `openai/...`、`qwen-...` 等前缀
4. **以为支持 OpenAI 全部接口**——核心仅支持 `/v1/chat/completions` 兼容，不是全兼容所有 OpenAI API

---

## 十三、实用技巧清单

1. 第一次一定先 `/doctor` 检查 API key、provider 路由、运行环境
2. 多 provider 环境里显式写模型前缀
3. 本地模型（Ollama）优先走 OpenAI-compatible 路径
4. 自动化场景优先用 `--output-format json`
5. 长任务靠 session 恢复，不要每次重来
6. 修改代码前后都 `/diff` 确认改动
7. 限制 agent 能力面用 `--allowedTools`
8. 要干净环境用容器

---

## 十四、学习路径

### 新手上路（最快跑起来）

1. 读 README 了解项目定位和安装警告
2. 读 USAGE.md 掌握运行方式
3. 执行：`cargo build --workspace` → `claw doctor` → `claw prompt "hello"`
4. 学会 `--model`、`--permission-mode`
5. 学会 `/status`、`/resume latest`

### 接入多种 Provider

重点练习四组配置：Anthropic、OpenRouter、Ollama、DashScope。

### 深入理解设计

1. `rust/README.md` 了解能力面和 crate 结构
2. `claw-code.codes` 站点补充理解架构背景（注意：内容围绕的是 `instructkr/claw-code`，并非当前主仓库）

---

## 十五、关键资料来源

### 最值得看的三份文档

1. **README** — 项目入口和关键警告
2. **USAGE.md** — 如何运行、认证、切 provider、管理 session
3. **rust/README.md** — CLI 能力面、crate 结构、slash commands 面

### 补充

1. **docs/container.md** — 容器化使用
2. **claw-code.codes** — 架构参考站（不能替代官方仓库文档）

---

## 十六、四套开箱即用配置

### 方案 A：Claude / Anthropic

```bash
git clone https://github.com/ultraworkers/claw-code
cd claw-code/rust && cargo build --workspace
export ANTHROPIC_API_KEY="sk-ant-..."
./target/debug/claw doctor
./target/debug/claw --model sonnet prompt "hello"
```

### 方案 B：OpenRouter

```bash
export OPENAI_BASE_URL="https://openrouter.ai/api/v1"
export OPENAI_API_KEY="sk-or-v1-..."
./target/debug/claw --model openai/gpt-4.1-mini prompt "summarize this repo"
```

### 方案 C：Ollama 本地模型

```bash
export OPENAI_BASE_URL="http://127.0.0.1:11434/v1"
unset OPENAI_API_KEY
./target/debug/claw --model llama3.2 prompt "summarize this repo"
```

### 方案 D：DashScope / Qwen

```bash
export DASHSCOPE_API_KEY="sk-..."
./target/debug/claw --model qwen-plus prompt "hello"
```
