---
name: create-command
description: "创建新 Skill"
---

# 创建新 Skill

根据用户需求在 `.qoder/skills/<skill-name>/SKILL.md` 中创建新的 Qoder skill。

## 使用方法

```bash
$create-command <skill-name> <description>
```

**示例**：
```bash
$create-command review-pr Check PR code changes against project guidelines
```

## 执行步骤

### 1. 解析输入

从用户输入中提取：
- **Skill 名称**：使用 kebab-case（例如 `review-pr`）
- **描述**：skill 应该完成什么

### 2. 分析需求

根据描述确定 skill 类型：
- **初始化**：读取文档，建立上下文
- **开发前**：读取规范，检查依赖
- **代码检查**：验证代码质量和规范遵循
- **记录**：记录进度、问题、结构变更
- **生成**：生成文档或代码模板

### 3. 生成 Skill 内容

最小的 `SKILL.md` 结构：

```markdown
---
name: <skill-name>
description: "<description>"
---

# <Skill 标题>

<关于何时以及如何使用此 skill 的说明>
```

### 4. 创建文件

创建：
- `.qoder/skills/<skill-name>/SKILL.md`

### 5. 确认创建

输出结果：

```text
[OK] Created Skill: <skill-name>

File path:
- .qoder/skills/<skill-name>/SKILL.md

Usage:
- Trigger directly with $<skill-name>
- Or open /skills and select it

Description:
<description>
```

## Skill 内容指南

### [OK] 好的 skill 内容

1. **清晰简洁**：立即可理解
2. **可执行**：AI 可以直接按照步骤执行
3. **范围明确**：清楚知道要做什么和不做什么
4. **有输出**：指定预期输出格式（如需要）

### [X] 避免

1. **太模糊**：例如"优化代码"
2. **太复杂**：单个 skill 不应超过 100 行
3. **重复功能**：先检查是否存在类似的 skill

## 命名约定

| Skill 类型 | 前缀 | 示例 |
|------------|--------|---------|
| 会话开始 | `start` | `start` |
| 开发前 | `before-` | `before-frontend-dev` |
| 检查 | `check-` | `check-frontend` |
| 记录 | `record-` | `record-session` |
| 生成 | `generate-` | `generate-api-doc` |
| 更新 | `update-` | `update-changelog` |
| 其他 | 动词优先 | `review-code`, `sync-data` |
