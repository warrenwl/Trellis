# 开发工作流

> 基于 [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)

---

## 目录

1. [快速开始（首先执行）](#快速开始首先执行)
2. [工作流概述](#工作流概述)
3. [会话启动流程](#会话启动流程)
4. [开发流程](#开发流程)
5. [会话结束](#会话结束)
6. [文件说明](#文件说明)
7. [最佳实践](#最佳实践)

---

## 快速开始（首先执行）

### 步骤 0：初始化开发者身份（仅首次执行）

> **多开发者支持**：每个开发者/Agent 需要首先初始化自己的身份

```bash
# 检查是否已初始化
python3 ./.trellis/scripts/get_developer.py

# 如果未初始化，运行：
python3 ./.trellis/scripts/init_developer.py <your-name>
# 示例：python3 ./.trellis/scripts/init_developer.py cursor-agent
```

这将创建：
- `.trellis/.developer` - 您的身份文件（gitignored，不提交）
- `.trellis/workspace/<your-name>/` - 您的个人工作区目录

**命名建议**：
- 人类开发者：使用您的名字，例如 `john-doe`
- Cursor AI：`cursor-agent` 或 `cursor-<task>`
- Claude Code：`claude-agent` 或 `claude-<task>`

### 步骤 1：了解当前上下文

```bash
# 一条命令获取完整上下文
python3 ./.trellis/scripts/get_context.py

# 或手动检查：
python3 ./.trellis/scripts/get_developer.py      # 您的身份
python3 ./.trellis/scripts/task.py list          # 活跃任务
git status && git log --oneline -10              # Git 状态
```

### 步骤 2：阅读项目指南 [必需]

**关键**：编写任何代码前必须阅读指南：

```bash
# 阅读前端指南索引（如适用）
cat .trellis/spec/frontend/index.md

# 阅读后端指南索引（如适用）
cat .trellis/spec/backend/index.md
```

**为什么要阅读两者？**
- 了解完整的项目架构
- 了解整个代码库的编码标准
- 了解前端和后端如何交互
- 了解整体代码质量要求

### 步骤 3：编码前 - 阅读具体指南（必需）

根据您的任务，阅读**详细**指南：

**前端任务**：
```bash
cat .trellis/spec/frontend/hook-guidelines.md      # 钩子
cat .trellis/spec/frontend/component-guidelines.md # 组件
cat .trellis/spec/frontend/type-safety.md          # 类型
```

**后端任务**：
```bash
cat .trellis/spec/backend/database-guidelines.md   # 数据库操作
cat .trellis/spec/backend/type-safety.md           # 类型
cat .trellis/spec/backend/logging-guidelines.md    # 日志
```

---

## 工作流概述

### 核心原则

1. **先读后写** - 开始前先了解上下文
2. **遵循标准** - [!] **编码前必须阅读 `.trellis/spec/` 指南**
3. **增量开发** - 一次完成一个任务
4. **及时记录** - 完成后立即更新跟踪文件
5. **记录限制** - [!] **每个日志文档最多 2000 行**

### 文件系统

```
.trellis/
|-- .developer           # 开发者身份（gitignored）
|-- scripts/
|   |-- __init__.py          # Python 包初始化
|   |-- common/              # 共享工具（Python）
|   |   |-- __init__.py
|   |   |-- paths.py         # 路径工具
|   |   |-- developer.py     # 开发者管理
|   |   +-- git_context.py   # Git 上下文实现
|   |-- multi_agent/         # 多 Agent 流水线脚本
|   |   |-- __init__.py
|   |   |-- start.py         # 启动 worktree agent
|   |   |-- status.py        # 监控 agent 状态
|   |   |-- create_pr.py     # 创建 PR
|   |   +-- cleanup.py       # 清理 worktree
|   |-- init_developer.py    # 初始化开发者身份
|   |-- get_developer.py     # 获取当前开发者名称
|   |-- task.py              # 任务管理
|   |-- get_context.py       # 获取会话上下文
|   +-- add_session.py       # 一键会话记录
|-- workspace/           # 开发者工作区
|   |-- index.md         # 工作区索引 + 会话模板
|   +-- {developer}/     # 每个开发者的目录
|       |-- index.md     # 个人索引（带 @@@auto 标记）
|       +-- journal-N.md # 日志文件（顺序编号）
|-- tasks/               # 任务跟踪
|   +-- {MM}-{DD}-{name}/
|       +-- task.json
|-- spec/                # [!] 编码前必须阅读
|   |-- frontend/        # 前端指南（如适用）
|   |   |-- index.md               # 从这里开始 - 指南索引
|   |   +-- *.md                   # 主题特定文档
|   |-- backend/         # 后端指南（如适用）
|   |   |-- index.md               # 从这里开始 - 指南索引
|   |   +-- *.md                   # 主题特定文档
|   +-- guides/          # 思维指南
|       |-- index.md                      # 指南索引
|       |-- cross-layer-thinking-guide.md # 实现前检查清单
|       +-- *.md                          # 其他指南
+-- workflow.md             # 本文档
```

---

## 会话启动流程

### 步骤 1：获取会话上下文

使用统一的上下文脚本：

```bash
# 一条命令获取所有上下文
python3 ./.trellis/scripts/get_context.py

# 或获取 JSON 格式
python3 ./.trellis/scripts/get_context.py --json
```

### 步骤 2：阅读开发指南 [!] 必需

**[!] 关键：编写任何代码前必须阅读指南**

根据您要开发的内容，阅读相应的指南：

**前端开发**（如适用）：
```bash
# 首先阅读索引，然后根据任务阅读具体文档
cat .trellis/spec/frontend/index.md
```

**后端开发**（如适用）：
```bash
# 首先阅读索引，然后根据任务阅读具体文档
cat .trellis/spec/backend/index.md
```

**跨层功能**：
```bash
# 对于跨越多个层的功能
cat .trellis/spec/guides/cross-layer-thinking-guide.md
```

### 步骤 3：选择要开发的任务

使用任务管理脚本：

```bash
# 列出活跃任务
python3 ./.trellis/scripts/task.py list

# 创建新任务（创建带 task.json 的目录）
python3 ./.trellis/scripts/task.py create "<title>" --slug <task-name>
```

---

## 开发流程

### 任务开发流程

```
1. 创建或选择任务
   --> python3 ./.trellis/scripts/task.py create "<title>" --slug <name> 或 list

2. 根据指南编写代码
   --> 阅读 .trellis/spec/ 与任务相关的文档
   --> 跨层功能：阅读 .trellis/spec/guides/

3. 自行测试
   --> 运行项目的 lint/test 命令（见 spec 文档）
   --> 手动功能测试

4. 提交代码
   --> git add <files>
   --> git commit -m "type(scope): description"
       格式：feat/fix/docs/refactor/test/chore

5. 记录会话（一键操作）
   --> python3 ./.trellis/scripts/add_session.py --title "Title" --commit "hash"
```

### 代码质量检查清单

**提交前必须通过**：
- [OK] Lint 检查通过（项目特定命令）
- [OK] 类型检查通过（如适用）
- [OK] 手动功能测试通过

**项目特定检查**：
- 前端见 `.trellis/spec/frontend/quality-guidelines.md`
- 后端见 `.trellis/spec/backend/quality-guidelines.md`

---

## 会话结束

### 一键会话记录

代码提交后，使用：

```bash
python3 ./.trellis/scripts/add_session.py \
  --title "会话标题" \
  --commit "abc1234" \
  --summary "简要总结"
```

这将自动：
1. 检测当前日志文件
2. 如果超过 2000 行限制则创建新文件
3. 追加会话内容
4. 更新 index.md（会话数量、历史表）

### 结束前检查清单

使用 `/trellis:finish-work` 命令运行：
1. [OK] 所有代码已提交，提交消息符合规范
2. [OK] 已通过 `add_session.py` 记录会话
3. [OK] 无 lint/test 错误
4. [OK] 工作区干净（或已记录 WIP）
5. [OK] 如需要已更新 spec 文档

---

## 文件说明

### 1. workspace/ - 开发者工作区

**目的**：记录每个 AI Agent 会话的工作内容

**结构**（多开发者支持）：
```
workspace/
|-- index.md              # 主索引（活跃开发者表）
+-- {developer}/          # 每个开发者的目录
    |-- index.md          # 个人索引（带 @@@auto 标记）
    +-- journal-N.md      # 日志文件（顺序：1, 2, 3...）
```

**更新时机**：
- [OK] 每个会话结束时
- [OK] 完成重要任务时
- [OK] 修复重要 bug 时

### 2. spec/ - 开发指南

**目的**：文档化的开发一致性标准

**结构**（多文档格式）：
```
spec/
|-- frontend/           # 前端文档（如适用）
|   |-- index.md        # 从这里开始
|   +-- *.md            # 主题特定文档
|-- backend/            # 后端文档（如适用）
|   |-- index.md        # 从这里开始
|   +-- *.md            # 主题特定文档
+-- guides/             # 思维指南
    |-- index.md        # 从这里开始
    +-- *.md            # 指南特定文档
```

**更新时机**：
- [OK] 发现新模式时
- [OK] 修复 bug 时发现缺少指导
- [OK] 建立新约定时

### 3. Tasks - 任务跟踪

每个任务是一个包含 `task.json` 的目录：

```
tasks/
|-- 01-21-my-task/
|   +-- task.json
+-- archive/
    +-- 2026-01/
        +-- 01-15-old-task/
            +-- task.json
```

**命令**：
```bash
python3 ./.trellis/scripts/task.py create "<title>" [--slug <name>]   # 创建任务目录
python3 ./.trellis/scripts/task.py archive <name>  # 归档到 archive/{year-month}/
python3 ./.trellis/scripts/task.py list            # 列出活跃任务
python3 ./.trellis/scripts/task.py list-archive    # 列出已归档任务
```

---

## 最佳实践

### [OK] 应该做

1. **会话开始前**：
   - 运行 `python3 ./.trellis/scripts/get_context.py` 获取完整上下文
   - [!] **必须阅读** 相关的 `.trellis/spec/` 文档

2. **开发期间**：
   - [!] **遵循** `.trellis/spec/` 指南
   - 对于跨层功能，使用 `/trellis:check-cross-layer`
   - 一次只开发一个任务
   - 频繁运行 lint 和测试

3. **开发完成后**：
   - 使用 `/trellis:finish-work` 完成检查清单
   - 修复 bug 后，使用 `/trellis:break-loop` 进行深度分析
   - 人类在测试通过后提交
   - 使用 `add_session.py` 记录进度

### [X] 不应该做

1. [!] **不要**跳过阅读 `.trellis/spec/` 指南
2. [!] **不要**让单个日志文件超过 2000 行
3. **不要**同时开发多个不相关的任务
4. **不要**提交有 lint/test 错误的代码
5. **不要**在学习新东西后忘记更新 spec 文档
6. [!] **不要**执行 `git commit` - AI 不应该提交代码

---

## 快速参考

### 开发前必读

| 任务类型 | 必读文档 |
|-----------|-------------------|
| 前端工作 | `frontend/index.md` → 相关文档 |
| 后端工作 | `backend/index.md` → 相关文档 |
| 跨层功能 | `guides/cross-layer-thinking-guide.md` |

### 提交规范

```bash
git commit -m "type(scope): description"
```

**类型**：feat, fix, docs, refactor, test, chore
**范围**：模块名称（例如 auth, api, ui）

### 常用命令

```bash
# 会话管理
python3 ./.trellis/scripts/get_context.py    # 获取完整上下文
python3 ./.trellis/scripts/add_session.py    # 记录会话

# 任务管理
python3 ./.trellis/scripts/task.py list      # 列出任务
python3 ./.trellis/scripts/task.py create "<title>" # 创建任务

# 斜杠命令
/trellis:finish-work          # 提交前检查清单
/trellis:break-loop           # 调试后分析
/trellis:check-cross-layer    # 跨层验证
```

---

## 总结

遵循此工作流确保：
- [OK] 多个会话之间的连续性
- [OK] 一致的代码质量
- [OK] 可跟踪的进度
- [OK] 在 spec 文档中积累知识
- [OK] 透明的团队协作

**核心理念**：先读后写，遵循标准，及时记录，捕获学习成果
