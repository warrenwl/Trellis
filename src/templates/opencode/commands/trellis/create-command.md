# 创建新斜杠命令

根据用户需求，在 `.cursor/commands/`（带 `trellis-` 前缀）和 `.opencode/commands/trellis/` 目录中创建新的斜杠命令。

## 用法

```
/trellis:create-command <command-name> <description>
```

**示例**：
```
/trellis:create-command review-pr Check PR code changes against project guidelines
```

## 执行步骤

### 1. 解析输入

从用户输入中提取：
- **命令名称**：使用 kebab-case（例如 `review-pr`）
- **描述**：命令应该完成什么

### 2. 分析需求

根据描述确定命令类型：
- **初始化**：读取文档，建立上下文
- **开发前**：读取指南，检查依赖
- **代码检查**：验证代码质量和指南合规性
- **记录**：记录进度、问题、结构更改
- **生成**：生成文档、代码模板

### 3. 生成命令内容

根据命令类型，生成适当的内容：

**简单命令**（1-3 行）：
```markdown
简明指令描述要做什么
```

**复杂命令**（带步骤）：
```markdown
# Command Title

Command description

## Steps

### 1. First Step
Specific action

### 2. Second Step
Specific action

## Output Format (if needed)

Template
```

### 4. 创建文件

在两个目录中创建：
- `.cursor/commands/trellis-<command-name>.md`
- `.opencode/commands/trellis/<command-name>.md`

### 5. 确认创建

输出结果：
```
[OK] Created Slash Command: /<command-name>

File paths:
- .cursor/commands/trellis-<command-name>.md
- .opencode/commands/trellis/<command-name>.md

Usage:
/trellis:<command-name>

Description:
<description>
```

## 命令内容指南

### [OK] 好的命令内容

1. **清晰简洁**：立即可理解
2. **可执行**：AI 可以直接按照步骤执行
3. **范围明确**：清楚界限做什么和不做什么
4. **有输出**：指定预期输出格式（如需要）

### [X] 避免

1. **太模糊**：例如"优化代码"
2. **太复杂**：单个命令不应超过 100 行
3. **功能重复**：先检查是否存在类似命令

## 命名约定

| 命令类型 | 前缀 | 示例 |
|--------------|---------|---------|
| 会话开始 | `start` | `start` |
| 开发前 | `before-` | `before-frontend-dev` |
| 检查 | `check-` | `check-frontend` |
| 记录 | `record-` | `record-session` |
| 生成 | `generate-` | `generate-api-doc` |
| 更新 | `update-` | `update-changelog` |
| 其他 | 动词优先 | `review-code`, `sync-data` |

## 示例

### 输入
```
/trellis:create-command review-pr Check PR code changes against project guidelines
```

### 生成的命令内容
```markdown
# PR Code Review

Check current PR code changes against project guidelines.

## Steps

### 1. Get Changed Files
```bash
git diff main...HEAD --name-only
```

### 2. Categorized Review

**Frontend files** (`apps/web/`):
- Reference `.trellis/spec/frontend/index.md`

**Backend files** (`packages/api/`):
- Reference `.trellis/spec/backend/index.md`

### 3. Output Review Report

Format:

## PR Review Report

### Changed Files
- [file list]

### Check Results
- [OK] Passed items
- [X] Issues found

### Suggestions
- [improvement suggestions]
```
