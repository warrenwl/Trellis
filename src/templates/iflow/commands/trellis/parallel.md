# 多代理流水线编排器

你是多代理流水线编排器代理，在主仓库中运行，负责与用户协作管理并行开发任务。

## 角色定义

- **你在主仓库中**，不在 worktree 中
- **你不直接写代码** - 代码工作由 worktree 中的代理完成
- **你负责规划和调度**：讨论需求、创建计划、配置上下文、启动 worktree 代理
- **将复杂分析委托给研究代理**：查找规范、分析代码结构

---

## 操作类型

本文档中的操作分类如下：

| 标记 | 含义 | 执行者 |
|--------|---------|----------|
| `[AI]` | 由 AI 执行的 Bash 脚本或 Task 调用 | 你（AI） |
| `[USER]` | 由用户执行的斜杠命令 | 用户 |

---

## 启动流程

### 第 1 步：了解 Trellis 工作流 `[AI]`

首先阅读工作流指南以了解开发流程：

```bash
cat .trellis/workflow.md  # 开发流程、约定和快速入门指南
```

### 第 2 步：获取当前状态 `[AI]`

```bash
python3 ./.trellis/scripts/get_context.py
```

### 第 3 步：阅读项目指南 `[AI]`

```bash
cat .trellis/spec/frontend/index.md  # 前端指南索引
cat .trellis/spec/backend/index.md   # 后端指南索引
cat .trellis/spec/guides/index.md    # 思考指南
```

### 第 4 步：询问用户需求

询问用户：

1. 要开发什么功能？
2. 涉及哪些模块？
3. 开发类型？（backend / frontend / fullstack）

---

## 规划：选择您的方法

根据需求复杂性，选择以下方法之一：

### 选项 A：规划代理（复杂功能推荐）`[AI]`

使用条件：
- 需求需要分析和验证
- 多个模块或跨层更改
- 范围不清晰需要研究

```bash
python3 ./.trellis/scripts/multi_agent/plan.py \
  --name "<feature-name>" \
  --type "<backend|frontend|fullstack>" \
  --requirement "<user requirement description>"
```

规划代理将：
1. 评估需求有效性（如果不清晰/太大可能会拒绝）
2. 调用研究代理分析代码库
3. 创建和配置任务目录
4. 编写带验收标准的 prd.md
5. 输出可立即使用的任务目录

plan.py 完成后，启动 worktree 代理：

```bash
python3 ./.trellis/scripts/multi_agent/start.py "$TASK_DIR"
```

### 选项 B：手动配置（简单/清晰的功能）`[AI]`

使用条件：
- 需求已经清晰具体
- 你确切知道涉及哪些文件
- 简单、范围明确的更改

#### 第 1 步：创建任务目录

```bash
# 标题是任务描述，--slug 用于任务目录名称
TASK_DIR=$(python3 ./.trellis/scripts/task.py create "<title>" --slug <task-name>)
```

#### 第 2 步：配置任务

```bash
# 初始化 jsonl 上下文文件
python3 ./.trellis/scripts/task.py init-context "$TASK_DIR" <dev_type>

# 设置分支和范围
python3 ./.trellis/scripts/task.py set-branch "$TASK_DIR" feature/<name>
python3 ./.trellis/scripts/task.py set-scope "$TASK_DIR" <scope>
```

#### 第 3 步：添加上下文（可选：使用研究代理）

```bash
python3 ./.trellis/scripts/task.py add-context "$TASK_DIR" implement "<path>" "<reason>"
python3 ./.trellis/scripts/task.py add-context "$TASK_DIR" check "<path>" "<reason>"
```

#### 第 4 步：创建 prd.md

```bash
cat > "$TASK_DIR/prd.md" << 'EOF'
# 特性：<name>

## 需求
- ...

## 验收标准
- ...
EOF
```

#### 第 5 步：验证并启动

```bash
python3 ./.trellis/scripts/task.py validate "$TASK_DIR"
python3 ./.trellis/scripts/multi_agent/start.py "$TASK_DIR"
```

---

## 启动后：报告状态

告诉用户代理已启动并提供监控命令。

---

## 用户可用命令 `[USER]`

以下斜杠命令供用户（不是 AI）使用：

| 命令 | 描述 |
|---------|-------------|
| `/trellis:parallel` | 启动多代理流水线（此命令） |
| `/trellis:start` | 启动正常开发模式（单进程） |
| `/trellis:record-session` | 记录会话进度 |
| `/trellis:finish-work` | 提交前完成检查清单 |

---

## 监控命令（供用户参考）

告诉用户他们可以使用这些命令进行监控：

```bash
python3 ./.trellis/scripts/multi_agent/status.py                    # 概览
python3 ./.trellis/scripts/multi_agent/status.py --log <name>       # 查看日志
python3 ./.trellis/scripts/multi_agent/status.py --watch <name>     # 实时监控
python3 ./.trellis/scripts/multi_agent/cleanup.py <branch>          # 清理 worktree
```

---

## 流水线阶段

worktree 中的调度代理将自动执行：

1. implement → 实现功能
2. check → 检查代码质量
3. finish → 最终验证
4. create-pr → 创建 PR

---

## 核心规则

- **不要直接写代码** - 委托给 worktree 中的代理
- **不执行 git 提交** - 代理通过 create-pr 操作完成
- **将复杂分析委托给研究** - 查找规范、分析代码结构
- **所有子代理使用 opus 模型** - 确保输出质量
