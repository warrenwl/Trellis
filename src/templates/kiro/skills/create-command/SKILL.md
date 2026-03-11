---
name: create-command
description: "创建新技能"
---

# 创建新技能

根据用户需求，在 `.kiro/skills/<skill-name>/SKILL.md` 中创建新的 Kiro 技能。

## 使用方法

```bash
$create-command <skill-name> <description>
```

**示例**：
```bash
$create-command review-pr 检查 PR 代码变更是否符合项目指南
```

## 执行步骤

### 1. 解析输入

从用户输入中提取：
- **技能名称**：使用 kebab-case（例如 `review-pr`）
- **描述**：技能应该完成什么

### 2. 分析需求

根据描述确定技能类型：
- **初始化**：阅读文档，建立上下文
- **开发前**：阅读指南，检查依赖
- **代码检查**：验证代码质量和指南合规性
- **记录**：记录进度、问题、结构变化
- **生成**：生成文档或代码模板

### 3. 生成技能内容

最低 `SKILL.md` 结构：

```markdown
---
name: <skill-name>
description: "<description>"
---

# <技能标题>

<关于何时以及如何使用此技能的说明>
```

### 4. 创建文件

创建：
- `.kiro/skills/<skill-name>/SKILL.md`

### 5. 确认创建

输出结果：

```text
[OK] 已创建技能：<skill-name>

文件路径：
- .kiro/skills/<skill-name>/SKILL.md

使用方法：
- 直接用 $<skill-name> 触发
- 或打开 /skills 并选择它

描述：
<description>
```

## 技能内容指南

### [OK] 好的技能内容

1. **清晰简洁**：立即可理解
2. **可执行**：AI 可以直接遵循步骤
3. **范围明确**：清楚要做什么和不做什么的边界
4. **有输出**：指定预期输出格式（如需要）

### [X] 避免

1. **太模糊**：例如"优化代码"
2. **太复杂**：单个技能不应超过 100 行
3. **重复功能**：先检查是否存在类似技能

## 命名约定

| 技能类型 | 前缀 | 示例 |
|------------|--------|---------|
| 会话开始 | `start` | `start` |
| 开发前 | `before-` | `before-frontend-dev` |
| 检查 | `check-` | `check-frontend` |
| 记录 | `record-` | `record-session` |
| 生成 | `generate-` | `generate-api-doc` |
| 更新 | `update-` | `update-changelog` |
| 其他 | 动词优先 | `review-code`, `sync-data` |
