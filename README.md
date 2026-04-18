
# Krio介绍

Krio 是一个面向 Agent 的本地代码工作区工具。

它的目标不是替代完整的终端，而是为 Agent 提供一组稳定、可组合、可结构化消费的本地能力，让 Agent 能更安全、更高效地理解和操作代码仓库。

Krio 主要解决以下问题：

- 让 Agent 以统一命令访问工作区文件、目录与代码结构
- 默认提供 JSON 输出，方便上层 Agent 直接解析与编排
- 在需要人工查看时，也支持 `--format text` 输出
- 聚焦代码理解与研发辅助场景，例如文件读取、文本搜索、符号定位、定义/引用分析、Git 差异查看、测试执行、工作区摘要与上下文恢复

典型使用场景：

- Agent 读取文件片段并分析代码
- Agent 在仓库内搜索文本、文件和符号
- Agent 定位某个符号的定义、引用和影响范围
- Agent 汇总当前工作区状态，恢复最近开发上下文
- Agent 在修改后执行测试并收集结果


## Krio 本地配置与使用手册

## 目录

- [快速开始](#快速开始)
- [系统要求](#系统要求)
- [安装与环境](#安装与环境)
- [启动方式](#启动方式)
- [命令总览](#命令总览)
- [命令参考](#命令参考)
- [输出格式](#输出格式)
- [常见问题](#常见问题)

---

## 快速开始

### 1. 准备对应平台的可执行文件

建议使用当前发布版本对应的二进制文件，并按平台重命名：

| 平台 | 建议文件名 |
|------|------------|
| Windows | `krio.exe` |
| macOS Intel | `krio` |
| macOS Apple Silicon | `krio` |
| Linux | `krio` |

如果你下载到的文件带有版本号，例如 `krio-1.0.0-windows-amd64.exe`，建议手动重命名为上表中的名称，后续命令更方便。

### 2. 将 Krio 所在目录加入 PATH（可选但推荐）

**Windows PowerShell：**

```powershell
# 临时生效（当前会话）
$env:PATH += ";D:\Path\To\Krio\bin"

# 永久生效（对新开终端生效）
[Environment]::SetEnvironmentVariable(
  "PATH",
  $env:PATH + ";D:\Path\To\Krio\bin",
  "User"
)
```

**Linux/macOS：**

```bash
# 临时生效
export PATH="$PATH:/path/to/krio/bin"

# 永久生效（bash）
echo 'export PATH="$PATH:/path/to/krio/bin"' >> ~/.bashrc

# 永久生效（zsh）
echo 'export PATH="$PATH:/path/to/krio/bin"' >> ~/.zshrc
```

### 3. 验证安装

```bash
krio version
krio help
```

---

## 系统要求

- 操作系统：Windows 10+、macOS、Linux
- 架构：amd64（x86_64）或 arm64
- 依赖：无额外运行时依赖，二进制可直接运行
- 代码语义相关命令支持：当前主要支持 `.go` 和 `.java` 文件

---

## 安装与环境

### Windows

1. 将可执行文件放到一个稳定目录，建议使用英文路径。
2. 如有需要，将文件重命名为 `krio.exe`。
3. 将该目录加入 `PATH`。
4. 打开新的终端窗口后运行 `krio version` 验证。

### macOS / Linux

1. 将可执行文件放到目录中，例如 `/usr/local/bin` 或自定义目录。
2. 将文件重命名为 `krio`。
3. 添加执行权限：

```bash
chmod +x /path/to/krio
```

4. 将文件所在目录加入 `PATH`。
5. 运行 `krio version` 验证。

### 环境变量

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `KRIO_TEXT_ENCODING` | 控制终端文本输出编码。Windows 传统中文终端建议设为 `gbk` | `utf-8` |

**Windows CMD：**

```cmd
set KRIO_TEXT_ENCODING=gbk
```

**Windows PowerShell：**

```powershell
$env:KRIO_TEXT_ENCODING = "gbk"
```

**Linux/macOS：**

```bash
export KRIO_TEXT_ENCODING=utf-8
```

### 常用配置示例

**Windows PowerShell：**

```powershell
$env:KRIO_TEXT_ENCODING = "gbk"
$krio = "D:\Work\WorkGoLang\Krio\bin"
$env:PATH = "$env:PATH;$krio"
```

**Linux/macOS：**

```bash
export KRIO_TEXT_ENCODING=utf-8
export PATH="$PATH:/path/to/krio/bin"
```

---

## 启动方式

Krio 支持两种模式。

### 1. CLI 模式

执行单条命令后退出：

```bash
krio <command> [flags]
```

例如：

```bash
krio tree --format text
krio search-text --query "func main" --glob "*.go"
```

### 2. REPL 交互模式

不带参数直接运行 `krio` 会进入交互模式：

```bash
krio
```

REPL 特点：

- 提示符显示当前工作目录
- 自动读写历史记录文件 `~/.krio_history`
- 内置命令：`cd`、`exit`、`quit`、`?`
- 也可以直接执行 CLI 命令，如 `tree`、`ls`、`search-text`
- 在 REPL 中执行 Krio CLI 命令时，若未显式指定 `--format`，默认会追加 `--format=text`，更适合交互查看
- 输入非 Krio 命令时，会尝试当作系统命令执行

常见 REPL 示例：

```text
krio
~/work> pwd
~/work> cd D:\Work\WorkGoLang\Krio
~/work/Krio> tree --max-depth 2
~/work/Krio> search-text --query "TODO" --glob "*.go"
~/work/Krio> exit
```

---

## 命令总览

当前支持的命令如下：

- `help` / `-h` / `--help`
- `version` / `--version`
- `chao` / `--chao`
- `tree`
- `pwd`
- `ls`
- `ll`
- `read-file`
- `find-files`
- `search-text`
- `search-symbol`
- `symbol-at`
- `enclosing-symbol`
- `find-definition`
- `find-references`
- `impact`
- `resume`
- `git-diff`
- `run-test`
- `workspace-summary`

---

## 命令参考

### 通用说明

- 大多数命令默认输出 JSON
- 传入 `--format text` 可输出适合终端阅读的文本格式
- 传入 `--pretty` 可格式化 JSON 输出
- 路径参数同时支持相对路径和绝对路径

### 基础命令

#### `help`

```bash
krio help
krio --help
```

显示命令列表与基本用法。

#### `version`

```bash
krio version
krio --version
```

显示当前 Krio 版本。

#### `chao`

```bash
krio chao
krio --chao
```

显示 Krio Banner。

### 文件与目录

#### `tree` - 输出目录树

```bash
krio tree [path] [flags]
```

参数：

- `--root <path>`：根目录，默认当前目录
- `--max-depth <n>`：最大深度，默认 `4`
- `--max-entries <n>`：最大节点数，默认 `5000`
- `--include-files`：是否包含文件，默认 `true`
- `--include-dirs`：是否包含目录，默认 `true`
- `--ignore <pattern>`：忽略模式，可重复指定
- `--format json|text`
- `--pretty`

示例：

```bash
krio tree
krio tree ./src --max-depth 2 --format text
krio tree --root . --ignore ".git" --ignore "node_modules" --pretty
```

#### `pwd` - 显示当前目录

```bash
krio pwd [--format text]
```

该命令不接受位置参数。

#### `ls` - 列出目录内容

```bash
krio ls [path] [flags]
```

参数：

- `--path <path>`：目录路径，默认当前目录
- `--all`：包含隐藏文件
- `--max-entries <n>`：最大输出条目数，默认 `5000`
- `--format json|text`
- `--pretty`

示例：

```bash
krio ls
krio ls ./cmd --format text
krio ls --all --max-entries 100
```

#### `ll` - 详细列出目录内容

```bash
krio ll [path] [flags]
```

参数与 `ls` 一致，文本模式下会额外显示类型、大小和修改时间。

#### `read-file` - 读取文件

```bash
krio read-file <file> [flags]
krio read-file --path <file> [flags]
```

参数：

- `--path <file>`：文件路径
- `--range <start:end>`：按行读取，例如 `1:50`
- `--max-bytes <n>`：最大读取字节数，默认 `1048576`
- `--format json|text`
- `--pretty`

说明：

- `--range` 的分隔符是冒号 `:`
- 若文件大小超过 `--max-bytes`，命令会返回错误

示例：

```bash
krio read-file ./main.go
krio read-file ./main.go --range 1:80 --format text
krio read-file --path ./cmd/tree.go --max-bytes 2097152
```

### 搜索与代码分析

#### `find-files` - 查找文件

```bash
krio find-files [flags]
```

参数：

- `--root <path>`：搜索根目录，默认当前目录
- `--name <substring>`：文件名包含的字符串
- `--glob <pattern>`：glob 模式，可重复指定
- `--ext <extension>`：扩展名，可重复指定，支持写成 `go` 或 `.go`
- `--ignore <pattern>`：忽略模式，可重复指定
- `--max-results <n>`：最大结果数，默认 `2000`
- `--format json|text`
- `--pretty`

说明：

- 必须至少指定一个条件：`--name`、`--glob`、`--ext`

示例：

```bash
krio find-files --ext go
krio find-files --name test --glob "**/*_test.go"
krio find-files --root . --ext md --ignore ".git" --format text
```

#### `search-text` - 搜索文本内容

```bash
krio search-text [flags]
```

参数：

- `--root <path>`：搜索根目录，默认当前目录
- `--query <text>`：明文搜索
- `--regex <pattern>`：正则搜索
- `--case-sensitive`：区分大小写
- `--glob <pattern>`：文件过滤模式，可重复指定
- `--ignore <pattern>`：忽略模式，可重复指定
- `--context <n>`：上下文行数，默认 `2`
- `--max-hits <n>`：最大命中数，默认 `2000`
- `--format json|text`
- `--pretty`

说明：

- `--query` 和 `--regex` 必须且只能指定一个
- 当前只会处理 UTF-8 文本文件

示例：

```bash
krio search-text --query "func main"
krio search-text --regex "func\\s+\\w+\\s*\\(" --glob "*.go"
krio search-text --query "TODO" --context 1 --ignore ".git" --format text
```

#### `search-symbol` - 搜索代码符号

```bash
krio search-symbol --name <symbol> [flags]
```

参数：

- `--root <path>`：搜索根目录，默认当前目录
- `--name <symbol>`：必填，符号名称
- `--type <type>`：符号类型过滤
- `--glob <pattern>`：文件过滤模式，可重复指定
- `--ignore <pattern>`：忽略模式，可重复指定
- `--max-results <n>`：最大结果数，默认 `2000`
- `--format json|text`
- `--pretty`

支持的类型：

- Go：`func`、`var`、`type`、`const`
- Java：`class`、`interface`、`method`、`variable`

示例：

```bash
krio search-symbol --name Run --type func
krio search-symbol --name UserService --type class
krio search-symbol --name data --glob "*.go" --format text
```

#### `symbol-at` - 获取指定位置的符号

```bash
krio symbol-at --file <file> --line <n> --column <n> [flags]
```

参数：

- `--file <file>`：源文件路径
- `--line <n>`：1-based 行号
- `--column <n>`：1-based 列号
- `--format json|text`
- `--pretty`

说明：

- 当前仅支持 `.go`、`.java`

示例：

```bash
krio symbol-at --file ./cmd/tree.go --line 287 --column 18 --format text
```

#### `enclosing-symbol` - 获取当前位置所在的外层符号

```bash
krio enclosing-symbol --file <file> --line <n> --column <n> [flags]
```

参数与 `symbol-at` 基本一致，用于查询当前位置属于哪个类型、类、方法或函数。

#### `find-definition` - 查找符号定义

```bash
krio find-definition --file <file> --line <n> --column <n> [flags]
```

参数：

- `--file <file>`：源文件路径
- `--root <path>`：工作区根目录，默认取文件所在目录
- `--line <n>`：1-based 行号
- `--column <n>`：1-based 列号
- `--max-results <n>`：最大候选定义数，默认 `20`
- `--glob <pattern>`：文件过滤模式，可重复指定
- `--ignore <pattern>`：忽略模式，可重复指定
- `--format json|text`
- `--pretty`

#### `find-references` - 查找符号引用

```bash
krio find-references --file <file> --line <n> --column <n> [flags]
```

参数：

- `--file <file>`：源文件路径
- `--root <path>`：工作区根目录，默认自动探测
- `--line <n>`：1-based 行号
- `--column <n>`：1-based 列号
- `--context <n>`：上下文行数，默认 `1`
- `--max-results <n>`：最大引用数，默认 `2000`
- `--glob <pattern>`：文件过滤模式，可重复指定
- `--ignore <pattern>`：忽略模式，可重复指定
- `--format json|text`
- `--pretty`

#### `impact` - 分析符号影响范围

```bash
krio impact --file <file> --line <n> --column <n> [flags]
```

参数：

- `--file <file>`：源文件路径
- `--root <path>`：工作区根目录，默认自动探测
- `--line <n>`：1-based 行号
- `--column <n>`：1-based 列号
- `--max-results <n>`：最大分析引用数，默认 `2000`
- `--include-references`：在 JSON 中附带原始引用列表
- `--glob <pattern>`：文件过滤模式，可重复指定
- `--ignore <pattern>`：忽略模式，可重复指定
- `--format json|text`
- `--pretty`

### Git 与工作区

#### `resume` - 恢复最近工作上下文

```bash
krio resume [flags]
```

参数：

- `--root <path>`：工作区根目录，默认当前目录
- `--commits <n>`：读取最近提交数，默认 `5`
- `--changed-limit <n>`：返回的变更文件上限，默认 `20`
- `--format json|text`
- `--pretty`

说明：

- 若当前目录不是 Git 仓库，会返回较低置信度的工作区信息

#### `git-diff` - 显示 Git 差异

```bash
krio git-diff [flags]
```

参数：

- `--path <path>`：文件或目录路径，默认当前目录
- `--format json|text`
- `--pretty`

示例：

```bash
krio git-diff
krio git-diff --path ./cmd
```

#### `workspace-summary` - 生成工作区摘要

```bash
krio workspace-summary [flags]
```

参数：

- `--root <path>`：工作区根目录，默认当前目录
- `--format json|text`
- `--pretty`

### 测试

#### `run-test` - 运行测试

```bash
krio run-test [flags]
```

参数：

- `--path <path>`：测试文件或目录路径，默认当前目录
- `--pattern <name>`：测试名称过滤
- `--format json|text`
- `--pretty`

说明：

- Go 项目会执行 `go test`
- Maven 项目会执行 `mvn test`

示例：

```bash
krio run-test
krio run-test --path ./cmd
krio run-test --pattern TestTree
```

---

## 输出格式

### JSON 输出

除 `help`、`version`、`chao` 外，大多数命令默认输出 JSON，基本结构如下：

```json
{
  "success": true,
  "command": "tree",
  "elapsed": "10.123ms",
  "root": "D:/Work/WorkGoLang/Krio",
  "data": {}
}
```

失败时示例：

```json
{
  "success": false,
  "command": "read-file",
  "elapsed": "1.234ms",
  "root": "D:/Work/WorkGoLang/Krio/main.go",
  "error": {
    "code": "E_USAGE",
    "message": "missing file path",
    "details": {}
  }
}
```

不同命令的 `data` 字段结构不同，请以具体命令输出为准。

### 文本输出

通过 `--format text` 可获得更适合人读的输出，例如：

```bash
krio tree --format text
krio read-file ./main.go --range 1:20 --format text
krio find-references --file ./cmd/tree.go --line 287 --column 18 --format text
```

在 REPL 中执行 Krio 命令时，如果未显式指定 `--format`，默认会使用文本输出。

---

## 常见问题

### Q: Windows 上中文显示乱码？

A: 先尝试设置：

```powershell
$env:KRIO_TEXT_ENCODING = "gbk"
```

如果你是新开的 PowerShell 或 CMD 窗口，也可以把它写入启动脚本或系统环境变量。

### Q: `read-file` 的行范围怎么写？

A: 使用 `--range start:end`，例如：

```bash
krio read-file ./main.go --range 10:50
```

不是 `10-50`。

### Q: `find-files` 为什么直接报错？

A: 该命令必须至少指定一个搜索条件：

- `--name`
- `--glob`
- `--ext`

例如：

```bash
krio find-files --ext go
```

### Q: 代码语义命令支持哪些文件？

A: 当前 `search-symbol`、`symbol-at`、`enclosing-symbol`、`find-definition`、`find-references`、`impact` 主要面向 `.go` 和 `.java` 文件。

### Q: 如何忽略目录或文件？

A: 使用可重复指定的 `--ignore <pattern>`，例如：

```bash
krio tree --ignore ".git" --ignore "node_modules"
krio search-text --query "TODO" --ignore "vendor" --ignore "**/*.min.js"
```

---
