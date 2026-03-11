# 脚本规范

> `.trellis/scripts/` 目录中 Python 脚本的标准。

---

## 概述

所有工作流脚本均使用 **Python 3.10+** 编写，以确保跨平台兼容性。脚本仅使用标准库（无外部依赖）。

---

## 目录结构

```
.trellis/scripts/
├── __init__.py           # 包初始化
├── common/               # 共享模块
│   ├── __init__.py
│   ├── paths.py          # 路径常量和函数
│   ├── developer.py      # 开发者身份管理
│   ├── task_queue.py     # 任务队列 CRUD
│   ├── task_utils.py     # 任务辅助函数
│   ├── phase.py          # 多代理阶段追踪
│   ├── registry.py       # 代理注册表管理
│   ├── config.py         # 配置读取器（config.yaml、hooks）
│   ├── worktree.py       # Git worktree 工具 + YAML 解析器
│   └── git_context.py    # Git/会话上下文
├── hooks/                # 生命周期钩子脚本（项目特定）
│   └── linear_sync.py    # 示例：将任务同步到 Linear
├── multi_agent/          # 多代理流水线脚本
│   ├── __init__.py
│   ├── start.py          # 启动 worktree 代理
│   ├── status.py         # 监控代理状态
│   ├── plan.py           # 启动计划代理
│   ├── cleanup.py        # 清理 worktree
│   └── create_pr.py      # 从任务创建 PR
├── task.py               # 主要任务管理 CLI
├── get_context.py        # 会话上下文检索
├── init_developer.py     # 开发者初始化
├── get_developer.py      # 获取当前开发者
├── add_session.py        # 会话记录
└── create_bootstrap.py   # Bootstrap 任务创建
```

---

## 脚本类型

### 库模块（`common/*.py`）

由其他脚本导入的共享工具。**请勿直接运行。**

```python
# common/paths.py - 示例库模块

from __future__ import annotations

from pathlib import Path

# 常量
DIR_WORKFLOW = ".trellis"
DIR_SCRIPTS = "scripts"
DIR_TASKS = "tasks"

def get_repo_root() -> Path | None:
    """通过查找 .trellis 目录来找到仓库根目录。"""
    current = Path.cwd().resolve()
    while current != current.parent:
        if (current / DIR_WORKFLOW).is_dir():
            return current
        current = current.parent
    return None
```

### 入口脚本（`*.py`）

用户直接运行的 CLI 工具。包含使用说明的文档字符串。

```python
#!/usr/bin/env python3
"""
任务管理脚本。

用法：
    python3 task.py create "<title>" [--slug <name>]
    python3 task.py init-context <dir> <dev_type>
    python3 task.py add-context <dir> <file> <reason>
    python3 task.py validate <dir>
    python3 task.py list-context <dir>
    python3 task.py start <dir>
    python3 task.py finish
    python3 task.py set-branch <dir> <branch>
    python3 task.py set-base-branch <dir> <branch>
    python3 task.py set-scope <dir> <scope>
    python3 task.py create-pr [dir] [--dry-run]
    python3 task.py archive <task-name>
    python3 task.py list [--mine] [--status <status>]
    python3 task.py list-archive [YYYY-MM]
"""

from __future__ import annotations

import argparse
import sys
from pathlib import Path

from common.paths import get_repo_root, DIR_WORKFLOW


def main() -> int:
    """主入口点。"""
    parser = argparse.ArgumentParser(description="任务管理")
    # ... 参数设置
    args = parser.parse_args()
    # ... 命令分发
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

---

## 编码标准

### 类型提示

使用现代类型提示（Python 3.10+ 语法）：

```python
# 好
def get_tasks(status: str | None = None) -> list[dict]:
    ...

def read_json(path: Path) -> dict | None:
    ...

# 不好 - 旧式
from typing import Optional, List, Dict
def get_tasks(status: Optional[str] = None) -> List[Dict]:
    ...
```

### 路径处理

始终使用 `pathlib.Path`：

```python
# 好
from pathlib import Path

def read_file(path: Path) -> str:
    return path.read_text(encoding="utf-8")

config_path = repo_root / DIR_WORKFLOW / "config.json"

# 不好 - 字符串拼接
config_path = repo_root + "/" + DIR_WORKFLOW + "/config.json"
```

### JSON 操作

使用辅助函数以保持一致的错误处理：

```python
import json
from pathlib import Path


def read_json(path: Path) -> dict | None:
    """读取 JSON 文件，错误时返回 None。"""
    try:
        return json.loads(path.read_text(encoding="utf-8"))
    except (FileNotFoundError, json.JSONDecodeError):
        return None


def write_json(path: Path, data: dict) -> bool:
    """写入 JSON 文件，返回成功状态。"""
    try:
        path.write_text(
            json.dumps(data, indent=2, ensure_ascii=False),
            encoding="utf-8"
        )
        return True
    except Exception:
        return False
```

### 子进程执行

```python
import subprocess
from pathlib import Path


def run_command(
    cmd: list[str],
    cwd: Path | None = None
) -> tuple[int, str, str]:
    """运行命令并返回（返回码、stdout、stderr）。"""
    result = subprocess.run(
        cmd,
        cwd=cwd,
        capture_output=True,
        text=True
    )
    return result.returncode, result.stdout, result.stderr
```

---

## 跨平台兼容性

### 关键：Windows stdio 编码（stdout + stdin）

在 Windows 上，Python 的 stdout 和 stdin 默认为系统代码页（例如中国为 GBK/CP936，西方为 CP1252）。这会导致：
- 打印非 ASCII 字符时发生 `UnicodeEncodeError`（stdout）
- 读取管道输入的 UTF-8 内容时发生 `UnicodeDecodeError`（stdin），例如通过 `cat << EOF | python3 script.py` 输入的中文文本

**问题链（stdout）**：

```
Windows 代码页 = GBK (936)
    ↓
Python stdout 默认为 GBK 编码
    ↓
子进程输出包含特殊字符 → 替换为 \ufffd（替换字符）
    ↓
json.dumps(ensure_ascii=False) → print()
    ↓
GBK 无法编码 \ufffd → UnicodeEncodeError: 'gbk' codec can't encode character
```

**问题链（stdin）**：

```
AI 代理通过 heredoc 管道传输 UTF-8 内容：cat << 'EOF' | python3 add_session.py ...
    ↓
Python stdin 默认为 GBK 编码（PowerShell 默认代码页）
    ↓
sys.stdin.read() 将字节解码为 GBK，而非 UTF-8
    ↓
中文文本乱码或 UnicodeDecodeError
```

**根本原因**：即使在子进程调用中设置 `PYTHONIOENCODING`，**父进程的 stdio** 仍然使用系统代码页。

---

#### 好：在 `common/__init__.py` 中集中修复编码

所有 stdio 编码在一处处理。`from common import ...` 的脚本自动获得修复：

```python
# common/__init__.py
import io
import sys

def _configure_stream(stream):
    """在 Windows 上配置流为 UTF-8 编码。"""
    if hasattr(stream, "reconfigure"):
        stream.reconfigure(encoding="utf-8", errors="replace")
        return stream
    elif hasattr(stream, "detach"):
        return io.TextIOWrapper(stream.detach(), encoding="utf-8", errors="replace")
    return stream

if sys.platform == "win32":
    sys.stdout = _configure_stream(sys.stdout)
    sys.stderr = _configure_stream(sys.stderr)
    sys.stdin = _configure_stream(sys.stdin)    # 别忘了 stdin！
```

---

#### 不好：在单个脚本中内联编码代码

```python
# 不好 - 在每个脚本中重复，容易忘记 stdin
import sys
if sys.platform == "win32":
    sys.stdout.reconfigure(encoding="utf-8", errors="replace")
    # 忘记了 stdin！管道传输的中文文本会出错。
```

**为什么这样不好**：
1. **容易忘记流**：stdout 修复了但 stdin 在多个脚本中被忽略，导致实际用户 bug
2. **代码重复**：相同逻辑在 `add_session.py`、`git_context.py` 等之间复制粘贴
3. **覆盖不一致**：有些脚本只修复 stdout，有些修复 stdout+stderr，没有一个修复 stdin

**实际故障**：Windows 用户报告使用 `cat << EOF | python3 add_session.py` 时中文文本乱码。原因：stdin 从未重新配置为 UTF-8。

---

#### 总结

| 方法 | 可用？ | 原因 |
|--------|--------|--------|
| `common/__init__.py` 集中修复 | ✅ 是 | 所有流，所有脚本，一处 |
| `sys.stdout.reconfigure(encoding="utf-8")` | ⚠️ 部分 | 仅 stdout；容易忘记 stdin/stderr |
| `io.TextIOWrapper(sys.stdout.buffer, ...)` | ❌ 否 | 创建包装器，不修复底层编码 |
| `PYTHONIOENCODING=utf-8` 环境变量 | ⚠️ 部分 | 仅在 Python 启动**前**设置才有效 |

### 关键：始终明确使用 `python3`

Windows 不支持 shebang（`#!/usr/bin/env python3`）。始终使用明确的 `python3` 记录调用：

```python
# 在文档字符串中
"""
用法：
    python3 task.py create "My Task"
    python3 task.py list --mine
"""

# 在错误消息中
print("用法: python3 task.py <command>")
print("运行: python3 ./.trellis/scripts/init_developer.py <name>")

# 在帮助文本中
print("下一步：")
print("  python3 task.py start <dir>")
```

### 路径分隔符

使用 `pathlib.Path` - 它自动处理分隔符：

```python
# 好 - 在所有平台上都能工作
path = Path(".trellis") / "scripts" / "task.py"

# 不好 - 仅 Unix
path = ".trellis/scripts/task.py"
```

---

## 任务生命周期钩子

### 范围 / 触发器

任务生命周期事件（`after_create`、`after_start`、`after_finish`、`after_archive`）执行在 `config.yaml` 中配置的用户定义的 shell 命令。

### 签名

```python
# config.py — 从配置中读取钩子命令
def get_hooks(event: str, repo_root: Path | None = None) -> list[str]

# task.py — 执行钩子（从不阻塞主操作）
def _run_hooks(event: str, task_json_path: Path, repo_root: Path) -> None
```

### 约定

**配置格式**（`config.yaml`）：
```yaml
hooks:
  after_create:
    - "python3 .trellis/scripts/hooks/my_hook.py create"
  after_start:
    - "python3 .trellis/scripts/hooks/my_hook.py start"
  after_archive:
    - "python3 .trellis/scripts/hooks/my_hook.py archive"
```

**传递给钩子的环境变量**：

| 键 | 类型 | 描述 |
|-----|------|-------------|
| `TASK_JSON_PATH` | 绝对路径字符串 | 任务的 `task.json` 路径 |

- `cwd` 设置为 `repo_root`
- 钩子继承父进程环境 + `TASK_JSON_PATH`

### 子进程执行

```python
import os
import subprocess

env = {**os.environ, "TASK_JSON_PATH": str(task_json_path)}

result = subprocess.run(
    cmd,
    shell=True,
    cwd=repo_root,
    env=env,
    capture_output=True,
    text=True,
    encoding="utf-8",    # 必需：跨平台
    errors="replace",    # 必需：跨平台
)
```

### 验证和错误矩阵

| 条件 | 行为 |
|--------|----------|
| 配置中没有 `hooks` 键 | 无操作（空列表）|
| `hooks` 不是字典 | 无操作（空列表）|
| 事件键缺失 | 无操作（空列表）|
| 钩子命令退出非零 | `[WARN]` 到 stderr，继续执行下一个钩子 |
| 钩子命令抛出异常 | `[WARN]` 到 stderr，继续执行下一个钩子 |
| `linearis` 未安装 | 钩子失败并警告，任务操作成功 |

### 错误与正确

#### 错误 — 钩子失败时阻塞
```python
result = subprocess.run(cmd, shell=True, check=True)  # 失败时抛出异常！
```

#### 正确 — 警告并继续
```python
try:
    result = subprocess.run(cmd, shell=True, ...)
    if result.returncode != 0:
        print(f"[WARN] 钩子失败: {cmd}", file=sys.stderr)
except Exception as e:
    print(f"[WARN] 钩子错误: {cmd} — {e}", file=sys.stderr)
```

### 钩子脚本模式

需要项目特定配置（API 密钥、用户 ID）的钩子脚本应该：
1. 将配置存储在**被 gitignore 的**本地文件（例如 `.trellis/hooks.local.json`）
2. 启动时读取配置，如果缺失则显示清晰消息失败
3. 保持脚本本身可提交（无硬编码密钥）

```python
# .trellis/scripts/hooks/my_hook.py — 可提交，无密钥
CONFIG = _load_config()  # 从 .trellis/hooks.local.json 读取（被 gitignore）
TEAM = CONFIG.get("linear", {}).get("team", "")
```

---

## 自动提交模式

修改 `.trellis/` 跟踪文件的脚本应自动提交其更改以保持工作区整洁。使用 `--no-commit` 标志选择退出。

### 约定：变更后自动提交

```python
def _auto_commit(scope: str, message: str, repo_root: Path) -> None:
    """暂存并提交特定 .trellis/ 子目录中的更改。"""
    subprocess.run(["git", "add", "-A", scope], cwd=repo_root, capture_output=True)
    # 检查是否有暂存的更改
    result = subprocess.run(
        ["git", "diff", "--cached", "--quiet", "--", scope],
        cwd=repo_root,
    )
    if result.returncode == 0:
        print("[OK] 没有更改可提交。", file=sys.stderr)
        return
    commit_result = subprocess.run(
        ["git", "commit", "-m", message],
        cwd=repo_root, capture_output=True, text=True,
    )
    if commit_result.returncode == 0:
        print(f"[OK] 已自动提交: {message}", file=sys.stderr)
    else:
        print(f"[WARN] 自动提交失败: {commit_result.stderr.strip()}", file=sys.stderr)
```

**使用此模式的脚本**：
- `add_session.py` — 记录会话后提交 `.trellis/workspace` + `.trellis/tasks`
- `task.py archive` — 归档任务后提交 `.trellis/tasks`

**始终为自动提交的脚本添加 `--no-commit` 标志**，以便用户可以退出。

---

## CLI 模式扩展模式

### 设计决策：`--mode` 用于上下文相关的输出

当脚本需要针对不同用例产生不同输出时，使用 `--mode`（而非单独的脚本或额外标志）。

**示例**：`get_context.py` 有两种模式：
- `--mode default` — 完整会话上下文（开发者、GIT 状态、最近提交、当前任务、活跃任务、我的任务、日志、路径）
- `--mode record` — 专注记录会话的输出（我的活跃任务优先并强调、GIT 状态、最近提交、当前任务）

```python
parser.add_argument(
    "--mode", "-m",
    choices=["default", "record"],
    default="default",
    help="输出模式：default（完整上下文）或 record（用于记录会话）",
)
```

**何时添加新模式**（而非新脚本）：
- 输出是相同数据的子集/重排序
- 底层数据源是共享的
- 区别在于呈现方式，而非数据获取

---

## 错误处理

### 退出码

| 代码 | 含义 |
|------|---------|
| 0 | 成功 |
| 1 | 常规错误 |
| 2 | 用法错误（参数错误）|

### 错误消息

将错误打印到 stderr 并附带上下文：

```python
import sys

def error(msg: str) -> None:
    """将错误消息打印到 stderr。"""
    print(f"错误: {msg}", file=sys.stderr)

# 用法
if not repo_root:
    error("不在 Trellis 项目中（未找到 .trellis 目录）")
    sys.exit(1)
```

---

## 参数解析

使用 `argparse` 保持一致的 CLI 接口：

```python
import argparse


def main() -> int:
    parser = argparse.ArgumentParser(
        description="任务管理",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
示例：
  python3 task.py create "添加登录" --slug add-login
  python3 task.py list --mine --status in_progress
"""
    )

    subparsers = parser.add_subparsers(dest="command", required=True)

    # create 命令
    create_parser = subparsers.add_parser("create", help="创建新任务")
    create_parser.add_argument("title", help="任务标题")
    create_parser.add_argument("--slug", help="URL 友好名称")

    # list 命令
    list_parser = subparsers.add_parser("list", help="列出任务")
    list_parser.add_argument("--mine", "-m", action="store_true")
    list_parser.add_argument("--status", "-s", choices=["planning", "in_progress", "review", "completed"])

    args = parser.parse_args()

    if args.command == "create":
        return cmd_create(args)
    elif args.command == "list":
        return cmd_list(args)

    return 0
```

---

## 导入约定

### 包内的相对导入

```python
# 在 task.py（根级别）
from common.paths import get_repo_root, DIR_WORKFLOW
from common.developer import get_developer

# 在 common/developer.py 中
from .paths import get_repo_root, DIR_WORKFLOW
```

### 标准库导入

分组并排序导入：

```python
# 1. 未来导入
from __future__ import annotations

# 2. 标准库
import argparse
import json
import os
import subprocess
import sys
from datetime import datetime
from pathlib import Path

# 3. 本地导入
from common.paths import get_repo_root
from common.developer import get_developer
```

---

## 做 / 不做

### 做

- 对所有路径操作使用 `pathlib.Path`
- 使用类型提示（Python 3.10+ 语法）
- 从 `main()` 返回退出码
- 将错误打印到 stderr
- 在文档和消息中始终使用 `python3`
- 对所有文件操作使用 `encoding="utf-8"`

### 不做

- 不使用字符串路径拼接
- 当 `pathlib` 可用时不使用 `os.path`
- 不依赖 shebang 进行调用文档
- 不使用 `print()` 输出错误（使用 stderr）
- 不硬编码路径 - 使用 `common/paths.py` 中的常量
- 不使用外部依赖（仅限标准库）

---

## 示例：完整脚本

参见 `.trellis/scripts/task.py` 中的完整示例，包含：
- 多个子命令
- 参数解析
- JSON 文件操作
- 错误处理
- 跨平台路径处理

---

## 迁移说明

> **历史背景**：脚本在 v0.3.0 从 Bash 迁移到 Python 以实现跨平台兼容性。旧的 shell 脚本存档在 `.trellis/scripts-shell-archive/` 中（如果保留）。
