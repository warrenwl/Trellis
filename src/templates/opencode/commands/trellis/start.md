# 启动会话

初始化你的 AI 开发会话并开始处理任务。

---

## 操作类型

| 标记 | 含义 | 执行者 |
|--------|---------|----------|
| `[AI]` | 由 AI 执行的 Bash 脚本或 Task 调用 | 你（AI） |
| `[USER]` | 由用户执行的斜杠命令 | 用户 |

---

## 初始化 `[AI]`

### 步骤 1：了解开发工作流

首先，阅读工作流指南以了解开发过程：

```bash
cat .trellis/workflow.md
```

**按照 workflow.md 中的指示** - 它包含：
- 核心原则（先读后写、遵循标准等）
- 文件系统结构
- 开发过程
- 最佳实践

### 步骤 2：获取当前上下文

```bash
python3 ./.trellis/scripts/get_context.py
```

显示：开发人员身份、git 状态、当前任务（如有）、活动任务。

### 步骤 3：阅读指南索引

```bash
cat .trellis/spec/frontend/index.md  # 前端指南
cat .trellis/spec/backend/index.md   # 后端指南
cat ./.trellis/spec/guides/index.md    # 思维指南
```

### 步骤 4：报告并询问

报告你学到的并询问："你想做什么？"

---

## 任务分类

当用户描述任务时，对其进行分类：

| 类型 | 标准 | 工作流 |
|------|----------|----------|
| **问题** | 用户询问代码、架构或某事如何工作 | 直接回答 |
| **简单修复** | 拼笔误修复、注释更新、单行更改、< 5 分钟 | 直接编辑 |
| **简单任务** | 目标清晰、1-2 个文件、范围明确定义 | 快速确认 → 任务工作流 |
| **复杂任务** | 目标模糊、多个文件、架构决策 | **头脑风暴 → 任务工作流** |

### 决策规则

> **如果有疑问，使用头脑风暴 + 任务工作流。**
>
> 任务工作流确保代码-规范上下文被注入到智能体，从而产生更高质量的代码。
> 开销很小，但收益显著。

---

## 问题 / 简单修复

对于问题或简单修复，直接工作：

1. 回答问题或进行修复
2. 如果代码更改了，提醒用户运行 `/trellis:finish-work`

---

## 复杂任务 - 先头脑风暴

对于复杂或模糊的任务，使用头脑风暴过程来澄清需求。

参见 `/trellis:brainstorm` 获取完整过程。总结：

1. **确认并分类** - 说出你的理解
2. **创建任务目录** - 在 `prd.md` 中跟踪不断演变的需求
3. **一次问一个问题** - 每次回答后更新 PRD
4. **提出方法** - 对于架构决策
5. **确认最终需求** - 获得明确批准
6. **继续任务工作流** - 在 PRD 中有明确需求

> **子任务分解**：如果头脑风暴揭示多个独立工作项，考虑使用 `--parent` 标志或 `add-subtask` 命令创建子任务。
> 详见 `/trellis:brainstorm` 步骤 8。

---

## 任务工作流（开发任务）

**为什么用此工作流？**
- 研究智能体分析需要哪些代码-规范文件
- 代码-规范文件在 jsonl 文件中配置
- 实现智能体通过 Hook 注入接收代码-规范上下文
- 检查智能体根据代码-规范要求验证
- 结果：自动遵循项目约定的代码

### 概述：两个入口点

```
从头脑风暴（复杂任务）：
  PRD 确认 → 研究 → 配置上下文 → 激活 → 实现 → 检查 → 完成

从简单任务：
  确认 → 创建任务 → 编写 PRD → 研究 → 配置上下文 → 激活 → 实现 → 检查 → 完成
```

**关键原则：研究发生在需求明确之后（PRD 存在）。**

---

### 阶段 1：建立需求

#### 路径 A：从头脑风暴（跳到阶段 2）

PRD 和任务目录已从头脑风暴存在。直接跳到阶段 2。

#### 路径 B：从简单任务

**步骤 1：确认理解** `[AI]`

快速确认：
- 目标是什么？
- 什么类型的开发？（frontend / backend / fullstack）
- 有什么具体要求或约束？

**步骤 2：创建任务目录** `[AI]`

```bash
TASK_DIR=$(python3 ./.trellis/scripts/task.py create "<title>" --slug <name>)
```

**步骤 3：编写 PRD** `[AI]`

在任务目录中创建 `prd.md`：

```markdown
# <Task Title>

## Goal
<What we're trying to achieve>

## Requirements
- <Requirement 1>
- <Requirement 2>

## Acceptance Criteria
- [ ] <Criterion 1>
- [ ] <Criterion 2>

## Technical Notes
<Any technical decisions or constraints>
```

---

### 阶段 2：准备实现（共享）

> 两条路径在此汇聚。PRD 和任务目录在继续前必须存在。

**步骤 4：代码-规范深度检查** `[AI]`

如果任务触及基础设施或跨层契约，在定义代码-规范深度之前不要开始实现。

当更改包含以下任何一项时触发此要求：
- 新或更改的命令/API 签名
- 数据库模式或迁移更改
- 基础设施集成（存储、队列、缓存、密钥、环境契约）
- 跨层负载转换

继续前必须有：
- [ ] 要更新的目标代码-规范文件已识别
- [ ] 具体契约已定义（签名、字段、环境键）
- [ ] 验证和错误矩阵已定义
- [ ] 至少定义了一个 Good/Base/Bad 案例

**步骤 5：研究代码库** `[AI]`

根据确认的 PRD，调用研究智能体查找相关规范和模式：

```
Task(
  subagent_type: "research",
  prompt: "Analyze the codebase for this task:

  Task: <goal from PRD>
  Type: <frontend/backend/fullstack>

  Please find:
  1. Relevant code-spec files in .trellis/spec/
  2. Existing code patterns to follow (find 2-3 examples)
  3. Files that will likely need modification

  Output:
  ## Relevant Code-Specs
  - <path>: <why it's relevant>

  ## Code Patterns Found
  - <pattern>: <example file path>

  ## Files to Modify
  - <path>: <what change>",
  model: "opus"
)
```

**步骤 6：配置上下文** `[AI]`

初始化默认上下文：

```bash
python3 ./.trellis/scripts/task.py init-context "$TASK_DIR" <type>
# type: backend | frontend | fullstack
```

添加研究智能体找到的代码-规范文件：

```bash
# 对于每个相关代码-规范和代码模式：
python3 ./.trellis/scripts/task.py add-context "$TASK_DIR" implement "<path>" "<reason>"
python3 ./.trellis/scripts/task.py add-context "$TASK_DIR" check "<path>" "<reason>"
```

**步骤 7：激活任务** `[AI]`

```bash
python3 ./.trellis/scripts/task.py start "$TASK_DIR"
```

这设置 `.current-task` 以便 hooks 可以注入上下文。

---

### 阶段 3：执行（共享）

**步骤 8：实现** `[AI]`

调用实现智能体（代码-规范上下文由 hook 自动注入）：

```
Task(
  subagent_type: "implement",
  prompt: "Implement the task described in prd.md.

  Follow all code-spec files that have been injected into your context.
  Run lint and typecheck before finishing.",
  model: "opus"
)
```

**步骤 9：检查质量** `[AI]`

调用检查智能体（代码-规范上下文由 hook 自动注入）：

```
Task(
  subagent_type: "check",
  prompt: "Review all code changes against the code-spec requirements.

  Fix any issues you find directly.
  Ensure lint and typecheck pass.",
  model: "opus"
)
```

**步骤 10：完成** `[AI]`

1. 验证 lint 和 typecheck 通过
2. 报告实现了什么
3. 提醒用户：
   - 测试更改
   - 准备就绪时提交
   - 运行 `/trellis:record-session` 记录此会话

---

## 继续现有任务

如果 `get_context.py` 显示当前任务：

1. 阅读任务的 `prd.md` 了解目标
2. 检查 `task.json` 了解当前状态和阶段
3. 询问用户："继续处理 <task-name>？"

如果是，从适当步骤继续（通常是步骤 7 或 8）。

---

## 命令参考

### 用户命令 `[USER]`

| 命令 | 使用时机 |
|---------|-------------|
| `/trellis:start` | 开始会话（此命令） |
| `/trellis:brainstorm` | 澄清模糊需求（从 start 调用） |
| `/trellis:parallel` | 需要隔离 worktree 的复杂任务 |
| `/trellis:finish-work` | 提交更改前 |
| `/trellis:record-session` | 完成任务后 |

### AI 脚本 `[AI]`

| 脚本 | 用途 |
|---------|---------|
| `python3 ./.trellis/scripts/get_context.py` | 获取会话上下文 |
| `python3 ./.trellis/scripts/task.py create` | 创建任务目录 |
| `python3 ./.trellis/scripts/task.py init-context` | 初始化 jsonl 文件 |
| `python3 ./.trellis/scripts/task.py add-context` | 添加代码-规范/上下文文件到 jsonl |
| `python3 ./.trellis/scripts/task.py start` | 设置当前任务 |
| `python3 ./.trellis/scripts/task.py finish` | 清除当前任务 |
| `python3 ./.trellis/scripts/task.py archive` | 归档已完成任务 |

### 子智能体 `[AI]`

| 智能体 | 用途 | Hook 注入 |
|-------|---------|----------------|
| research | 分析代码库 | 否（直接读取） |
| implement | 写代码 | 是（implement.jsonl） |
| check | 审查和修复 | 是（check.jsonl） |
| debug | 修复特定问题 | 是（debug.jsonl） |

---

## 关键原则

> **代码-规范上下文是注入的，不是记住的。**
>
> 任务工作流确保智能体自动接收相关代码-规范上下文。
> 这比希望 AI"记住"约定更可靠。
