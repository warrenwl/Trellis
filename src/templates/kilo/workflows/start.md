# 开始会话

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

**按照 workflow.md 中的说明** - 它包含：
- 核心原则（先读后写、遵循标准等）
- 文件系统结构
- 开发过程
- 最佳实践

### 步骤 2：获取当前上下文

```bash
python3 ./.trellis/scripts/get_context.py
```

这显示：开发者身份、git 状态、当前任务（如有）、活动任务。

### 步骤 3：阅读指南索引

```bash
cat .trellis/spec/frontend/index.md  # 前端指南
cat .trellis/spec/backend/index.md   # 后端指南
cat .trellis/spec/guides/index.md    # 思维指南
```

### 步骤 4：报告并询问

报告你学到的并询问："你想做什么？"

---

## 任务分类

当用户描述任务时，对其进行分类：

| 类型 | 标准 | 工作流 |
|------|----------|----------|
| **问题** | 用户询问代码、架构或某事如何工作 | 直接回答 |
| **简单修复** | 拼写错误修复、注释更新、单行更改、< 5 分钟 | 直接编辑 |
| **开发任务** | 任何代码更改：修改逻辑、添加功能、修复 bug、涉及多个文件 | **任务工作流** |

### 决策规则

> **如果有疑问，使用任务工作流。**
>
> 任务工作流确保规范被注入到代理中，从而产生更高质量的代码。
> 开销很小，但收益显著。

---

## 问题 / 简单修复

对于问题或简单修复，直接处理：

1. 回答问题或进行修复
2. 如果代码被更改，提醒用户运行 `/trellis:finish-work`

---

## 任务工作流（开发任务）

**为什么用这个工作流？**
- 研究代理分析需要哪些规范
- 规范配置在 jsonl 文件中
- 实施代理通过 Hook 注入接收规范
- 检查代理根据规范验证
- 结果：自动遵循项目约定的代码

### 概述：两个入口点

```
来自头脑风暴（复杂任务）：
  PRD 确认 → 研究 → 配置上下文 → 激活 → 实施 → 检查 → 完成

来自简单任务：
  确认 → 创建任务 → 编写 PRD → 研究 → 配置上下文 → 激活 → 实施 → 检查 → 完成
```

**关键原则：研究在需求明确之后进行（PRD 存在）。**

> **子任务分解**：对于有多个独立工作项的复杂任务，考虑使用 `--parent` 标志或 `add-subtask` 命令创建子任务。
> 详见 `/trellis:brainstorm` 步骤 8。

---

## 阶段 1：建立需求

### 路径 A：从头脑风暴（跳到阶段 2）

PRD 和任务目录已从头脑风暴存在。直接跳到阶段 2。

### 路径 B：从简单任务

**步骤 1：确认理解** `[AI]`

快速确认：
- 目标是什么？
- 什么类型的开发？（前端 / 后端 / 全栈）
- 有什么特定要求或约束？

**步骤 2：创建任务目录** `[AI]`

```bash
TASK_DIR=$(python3 ./.trellis/scripts/task.py create "<title>" --slug <name>)
```

**步骤 3：编写 PRD** `[AI]`

在任务目录中创建 `prd.md`：

```markdown
# <任务标题>

## 目标
<我们要实现什么>

## 需求
- <需求 1>
- <需求 2>

## 验收标准
- [ ] <标准 1>
- [ ] <标准 2>

## 技术笔记
<任何技术决策或约束>
```

---

## 阶段 2：准备实施（共享）

> 两条路径在这里汇合。PRD 和任务目录必须在继续之前存在。

**步骤 4：代码-规范深度检查** `[AI]`

如果任务涉及基础设施或跨层契约，在定义代码-规范深度之前不要开始实施。

当变更包含以下任何一项时触发此要求：
- 新或更改的命令/API 签名
- 数据库模式或迁移更改
- 基础设施集成（存储、队列、缓存、密钥、环境契约）
- 跨层负载转换

继续之前必须有：
- [ ] 目标规范文件已确定
- [ ] 具体契约已定义（签名、字段、环境密钥）
- [ ] 验证和错误矩阵已定义
- [ ] 至少定义了一个好/基础/坏案例

**步骤 5：研究代码库** `[AI]`

根据确认的 PRD，调用研究代理找到相关规范和模式：

```
Task(
  subagent_type: "research",
  prompt: "Analyze the codebase for this task:

  Task: <goal from PRD>
  Type: <frontend/backend/fullstack>

  Please find:
  1. Relevant spec files in .trellis/spec/
  2. Existing code patterns to follow (find 2-3 examples)
  3. Files that will likely need modification

  Output:
  ## Relevant Specs
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

添加研究代理找到的规范：

```bash
# 对于每个相关规范和代码模式：
python3 ./.trellis/scripts/task.py add-context "$TASK_DIR" implement "<path>" "<reason>"
python3 ./.trellis/scripts/task.py add-context "$TASK_DIR" check "<path>" "<reason>"
```

**步骤 7：激活任务** `[AI]`

```bash
python3 ./.trellis/scripts/task.py start "$TASK_DIR"
```

这设置 `.current-task` 以便 hooks 可以注入上下文。

---

## 阶段 3：执行（共享）

**步骤 8：实施** `[AI]`

调用实施代理（规范通过 hook 自动注入）：

```
Task(
  subagent_type: "implement",
  prompt: "Implement the task described in prd.md.

  Follow all specs that have been injected into your context.
  Run lint and typecheck before finishing.",
  model: "opus"
)
```

**步骤 9：检查质量** `[AI]`

调用检查代理（规范通过 hook 自动注入）：

```
Task(
  subagent_type: "check",
  prompt: "Review all code changes against the specs.

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
   - 准备好后提交
   - 运行 `/trellis:record-session` 记录这个会话

---

## 继续现有任务

如果 `get_context.py` 显示当前任务：

1. 阅读任务的 `prd.md` 了解目标
2. 检查 `task.json` 了解当前状态和阶段
3. 询问用户："继续处理 <task-name>？"

如果是，从适当步骤恢复（通常是步骤 7 或 8）。

---

## 命令参考

### 用户命令 `[USER]`

| 命令 | 使用时机 |
|---------|-------------|
| `/trellis:start` | 开始会话（本命令） |
| `/trellis:parallel` | 需要隔离 worktree 的复杂任务 |
| `/trellis:finish-work` | 提交更改之前 |
| `/trellis:record-session` | 完成任务后 |

### AI 脚本 `[AI]`

| 脚本 | 用途 |
|---------|---------|
| `python3 ./.trellis/scripts/get_context.py` | 获取会话上下文 |
| `python3 ./.trellis/scripts/task.py create` | 创建任务目录 |
| `python3 ./.trellis/scripts/task.py init-context` | 初始化 jsonl 文件 |
| `python3 ./.trellis/scripts/task.py add-context` | 添加规范到 jsonl |
| `python3 ./.trellis/scripts/task.py start` | 设置当前任务 |
| `python3 ./.trellis/scripts/task.py finish` | 清除当前任务 |
| `python3 ./.trellis/scripts/task.py archive` | 归档已完成的任务 |

### 子代理 `[AI]`

| 代理 | 用途 | Hook 注入 |
|-------|---------|----------------|
| research | 分析代码库 | 否（直接读取） |
| implement | 写代码 | 是（implement.jsonl） |
| check | 审查和修复 | 是（check.jsonl） |
| debug | 修复特定问题 | 是（debug.jsonl） |

---

## 关键原则

> **规范被注入，而非被记住。**
>
> 任务工作流确保代理自动接收相关规范。
> 这比期望 AI "记住"约定更可靠。
