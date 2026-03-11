# 更新代码-规范 - 捕获可执行契约

当你学到有价值的东西时（通过调试、实现或讨论），使用此命令更新相关代码-规范文档。

**时机**：完成任务、修复 bug 或发现新模式后

---

## 代码-规范优先规则（关键）

在此项目中，实现工作的"规范"意味着**代码-规范**：
- 可执行契约（不仅仅是原则性文本）
- 具体签名、负载字段、环境键和边界行为
- 可测试的验证/错误行为

如果更改触及基础设施或跨层契约，代码-规范深度是强制的。

### 强制触发条件

当更改包含以下任何一项时应用代码-规范深度：
- 新/更改的命令或 API 签名
- 跨层请求/响应契约更改
- 数据库模式/迁移更改
- 基础设施集成（存储、队列、缓存、密钥、环境接线）

### 强制输出（7 个部分）

对于触发的任务，包含以下所有部分：
1. 范围 / 触发
2. 签名（命令/API/数据库）
3. 契约（请求/响应/环境）
4. 验证和错误矩阵
5. Good/Base/Bad 案例
6. 所需测试（带断言点）
7. 错误 vs 正确（至少一对）

---

## 何时更新代码-规范

| 触发 | 示例 | 目标规范 |
|---------|---------|-------------|
| **实现了功能** | 添加了带 giget 的模板下载 | 相关 `backend/` 或 `frontend/` 文件 |
| **做了设计决策** | 使用类型字段 + 映射表实现可扩展性 | 相关代码-规范 + "设计决策"部分 |
| **修复了 bug** | 发现错误处理的微妙问题 | `backend/error-handling.md` |
| **发现了模式** | 找到更好的代码结构方式 | 相关 `backend/` 或 `frontend/` 文件 |
| **遇到陷阱** | 了解到 X 必须在 Y 之前做 | 相关代码-规范 + "常见错误"部分 |
| **建立了约定** | 团队同意命名模式 | `quality-guidelines.md` |
| **新思维触发** | "做 Y 前别忘了检查 X" | `guides/*.md`（作为检查清单项，不是详细规则） |

**关键洞察**：代码-规范更新不仅仅是为了问题。每个功能实现都包含设计决策和契约，未来的 AI/开发人员需要这些来安全执行。

---

## 规范结构概述

```
.trellis/spec/
├── backend/           # 后端编码标准
│   ├── index.md       # 概述和链接
│   └── *.md           # 主题特定指南
├── frontend/          # 前端编码标准
│   ├── index.md       # 概述和链接
│   └── *.md           # 主题特定指南
└── guides/            # 思维检查清单（不是编码规范！）
    ├── index.md       # 指南索引
    └── *.md           # 主题特定指南
```

### 关键：代码-规范 vs 指南 - 知道区别

| 类型 | 位置 | 目的 | 内容风格 |
|------|----------|---------|---------------|
| **代码-规范** | `backend/*.md`, `frontend/*.md` | 告诉 AI"如何安全实现" | 签名、契约、矩阵、案例、测试点 |
| **指南** | `guides/*.md` | 帮助 AI"考虑什么" | 检查清单、问题、指向规范的指针 |

**决策规则**：问自己：

- "这是**如何写**代码" → 放入 `backend/` 或 `frontend/`
- "这是**写之前**要考虑什么" → 放入 `guides/`

**示例**：

| 学习内容 | 错误位置 | 正确位置 |
|----------|----------------|------------------|
| "Windows stdout 使用 `reconfigure()` 而不是 `TextIOWrapper`" | ❌ `guides/cross-platform-thinking-guide.md` | ✅ `backend/script-conventions.md` |
| "编写跨平台代码时记得检查编码" | ❌ `backend/script-conventions.md` | ✅ `guides/cross-platform-thinking-guide.md` |

**指南应该是指向规范**的简短检查清单，而不是复制详细规则。

---

## 更新过程

### 步骤 1：识别你学到了什么

回答这些问题：

1. **你学到了什么？**（具体点）
2. **为什么重要？**（它防止什么问题？）
3. **它属于哪里？**（哪个规范文件？）

### 步骤 2：分类更新类型

| 类型 | 描述 | 操作 |
|------|-------------|--------|
| **设计决策** | 为什么选择方法 X 而不是 Y | 添加到"设计决策"部分 |
| **项目约定** | 我们如何在此项目中做 X | 添加到相关部分并带示例 |
| **新模式** | 发现的可重用方法 | 添加到"模式"部分 |
| **禁止模式** | 导致问题的某些东西 | 添加到"反模式"或"不要"部分 |
| **常见错误** | 容易犯的错误 | 添加到"常见错误"部分 |
| **约定** | 同意的标准 | 添加到相关部分 |
| **陷阱** | 非显而易见的行为 | 添加警告提示 |

### 步骤 3：阅读目标代码-规范

编辑之前，阅读当前代码-规范以便：
- 了解现有结构
- 避免重复内容
- 找到更新的正确部分

```bash
cat .trellis/spec/<category>/<file>.md
```

### 步骤 4：执行更新

遵循这些原则：

1. **具体**：包含具体示例，而不仅仅是抽象规则
2. **解释原因**：说明这防止什么问题
3. **展示契约**：添加签名、负载字段和错误行为
4. **展示代码**：为关键模式添加代码片段
5. **保持简短**：每个部分一个概念

### 步骤 5：更新索引（如需要）

如果你添加了新部分或代码-规范状态更改，更新类别的 `index.md`。

---

## 更新模板

### 基础设施/跨层工作的强制模板

```markdown
## Scenario: <name>

### 1. Scope / Trigger
- Trigger: <why this requires code-spec depth>

### 2. Signatures
- Backend command/API/DB signature(s)

### 3. Contracts
- Request fields (name, type, constraints)
- Response fields (name, type, constraints)
- Environment keys (required/optional)

### 4. Validation & Error Matrix
- <condition> -> <error>

### 5. Good/Base/Bad Cases
- Good: ...
- Base: ...
- Bad: ...

### 6. Tests Required
- Unit/Integration/E2E with assertion points

### 7. Wrong vs Correct
#### Wrong
...
#### Correct
...
```

### 添加设计决策

```markdown
### Design Decision: [Decision Name]

**Context**: What problem were we solving?

**Options Considered**:
1. Option A - brief description
2. Option B - brief description

**Decision**: We chose Option X because...

**Example**:
\`\`\`typescript
// How it's implemented
code example
\`\`\`

**Extensibility**: How to extend this in the future...
```

### 添加项目约定

```markdown
### Convention: [Convention Name]

**What**: Brief description of the convention.

**Why**: Why we do it this way in this project.

**Example**:
\`\`\`typescript
// How to follow this convention
code example
\`\`\`

**Related**: Links to related conventions or specs.
```

### 添加新模式

```markdown
### Pattern Name

**Problem**: What problem does this solve?

**Solution**: Brief description of the approach.

**Example**:
\`\`\`
// Good
code example

// Bad
code example
\`\`\`

**Why**: Explanation of why this works better.
```

### 添加禁止模式

```markdown
### Don't: Pattern Name

**Problem**:
\`\`\`
// Don't do this
bad code example
\`\`\`

**Why it's bad**: Explanation of the issue.

**Instead**:
\`\`\`
// Do this instead
good code example
\`\`\`
```

### 添加常见错误

```markdown
### Common Mistake: Description

**Symptom**: What goes wrong

**Cause**: Why this happens

**Fix**: How to correct it

**Prevention**: How to avoid it in the future
```

### 添加陷阱

```markdown
> **Warning**: Brief description of the non-obvious behavior.
>
> Details about when this happens and how to handle it.
```

---

## 交互模式

如果你不确定要更新什么，回答这些提示：

1. **你刚刚完成了什么？**
   - [ ] 修复了 bug
   - [ ] 实现了功能
   - [ ] 重构了代码
   - [ ] 讨论了方法

2. **你学到了或决定什么？**
   - 设计决策（为什么选 X 而不是 Y）
   - 项目约定（我们如何做 X）
   - 非显而易见的行为（陷阱）
   - 更好的方法（模式）

3. **未来的 AI/开发人员需要知道这个吗？**
   - 理解代码如何工作 → 是，更新规范
   - 维护或扩展功能 → 是，更新规范
   - 避免重复错误 → 是，更新规范
   - 纯粹的一次性实现细节 → 也许跳过

4. **它涉及哪个领域？**
   - [ ] 后端代码
   - [ ] 前端代码
   - [ ] 跨层数据流
   - [ ] 代码组织/重用
   - [ ] 质量/测试

---

## 质量检查清单

完成代码-规范更新前：

- [ ] 内容具体且可操作？
- [ ] 包含了代码示例？
- [ ] 解释了原因，而不仅仅是是什么？
- [ ] 包含了可执行的签名/契约？
- [ ] 包含了验证和错误矩阵？
- [ ] 包含了 Good/Base/Bad 案例？
- [ ] 包含了带断言点的所需测试？
- [ ] 在正确的代码-规范文件中？
- [ ] 重复了现有内容？
- [ ] 新团队成员能理解吗？

---

## 与其他命令的关系

```
开发流程：
  学到东西 → /trellis:update-spec → 知识捕获
       ↑                                  ↓
  /trellis:break-loop ←──────────────────── 未来会话受益
  (深度 bug 分析)
```

- `/trellis:break-loop` - 深度分析 bug，经常揭示需要规范更新
- `/trellis:update-spec` - 实际执行更新（此命令）
- `/trellis:finish-work` - 提醒你检查是否需要更新规范

---

## 核心哲学

> **代码-规范是活文档。每个调试会话、每个"啊哈时刻"都是让实现契约更清晰的机会。**

目标是**制度记忆**：
- 一个人学到的东西，每个人都受益
- AI 在一个会话中学到的东西，持久到未来会话
- 错误变成记录的护栏
