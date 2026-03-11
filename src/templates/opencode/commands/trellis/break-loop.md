# 打破循环 - 深度 Bug 分析

调试完成后，使用此命令进行深度分析，打破"修复 bug → 忘记 → 重复"的循环。

---

## 分析框架

从以下 5 个维度分析你刚刚修复的 bug：

### 1. 根因类别

这个 bug 属于哪个类别？

| 类别 | 特征 | 示例 |
|----------|-----------------|---------|
| **A. 缺失规范** | 没有如何做的文档 | 新功能没有检查清单 |
| **B. 跨层契约** | 层之间接口不清晰 | API 返回格式与预期不同 |
| **C. 变更传播失败** | 改了一处，遗漏其他 | 更改了函数签名，遗漏调用点 |
| **D. 测试覆盖缺口** | 单元测试通过，集成失败 | 单独工作，组合时失败 |
| **E. 隐式假设** | 代码依赖未记录的假设 | 时间戳秒 vs 毫秒 |

### 2. 修复失败原因（如适用）

如果尝试了多次修复才成功，分析每次失败：

- **表面修复**：修复了症状，没修复根因
- **范围不完整**：找到根因，但没覆盖所有情况
- **工具限制**：grep 遗漏，类型检查不够严格
- **心智模型**：一直在同一层查找，没考虑跨层

### 3. 预防机制

什么机制可以防止这种情况再次发生？

| 类型 | 描述 | 示例 |
|------|-------------|---------|
| **文档** | 写下来让大家知道 | 更新思维指南 |
| **架构** | 从结构上让错误不可能 | 类型安全包装器 |
| **编译时** | TypeScript 严格，无 any | 签名更改导致编译错误 |
| **运行时** | 监控、告警、扫描 | 检测孤立实体 |
| **测试覆盖** | E2E 测试、集成测试 | 验证完整流程 |
| **代码审查** | 检查清单、PR 模板 | "你检查了 X 吗？" |

### 4. 系统性扩展

这个 bug 揭示了哪些更广泛的问题？

- **类似问题**：这个问题还可能出现在哪里？
- **设计缺陷**：是否存在根本的架构问题？
- **流程缺陷**：是否有开发流程改进？
- **知识差距**：团队是否缺少某些理解？

### 5. 知识捕获

将洞察固化为系统知识：

- [ ] 更新 `.trellis/spec/guides/` 思维指南
- [ ] 更新 `.trellis/spec/backend/` 或 `frontend/` 文档
- [ ] 创建问题记录（如适用）
- [ ] 为根因修复创建功能工单
- [ ] 如需要更新检查命令

---

## 输出格式

请按此格式输出分析：

```markdown
## Bug Analysis: [Short Description]

### 1. Root Cause Category
- **Category**: [A/B/C/D/E] - [Category Name]
- **Specific Cause**: [Detailed description]

### 2. Why Fixes Failed (if applicable)
1. [First attempt]: [Why it failed]
2. [Second attempt]: [Why it failed]
...

### 3. Prevention Mechanisms
| Priority | Mechanism | Specific Action | Status |
|----------|-----------|-----------------|--------|
| P0 | ... | ... | TODO/DONE |

### 4. Systematic Expansion
- **Similar Issues**: [List places with similar problems]
- **Design Improvement**: [Architecture-level suggestions]
- **Process Improvement**: [Development process suggestions]

### 5. Knowledge Capture
- [ ] [Documents to update / tickets to create]
```

---

## 核心哲学

> **调试的价值不在于修复这个 bug，而在于让这类 bug 不再发生。**

三个层次的洞察：
1. **战术**：如何修复这个 bug
2. **战略**：如何防止这一类 bug
3. **哲学**：如何扩展思维模式

30 分钟的分析节省 30 小时的未来调试。

---

## 分析后：立即行动

**重要**：完成上述分析后，你必须立即：

1. **更新规范/指南** - 不要只列出 TODO，实际更新相关文件：
   - 如果是跨平台问题 → 更新 `cross-platform-thinking-guide.md`
   - 如果是跨层问题 → 更新 `cross-layer-thinking-guide.md`
   - 如果是代码重用问题 → 更新 `code-reuse-thinking-guide.md`
   - 如果是领域特定 → 更新 `backend/*.md` 或 `frontend/*.md`

2. **同步模板** - 更新 `.trellis/spec/` 后，同步到 `src/templates/markdown/spec/`

3. **提交规范更新** - 这是主要输出，不只是分析文本

> **如果分析留在聊天中就没有价值。价值在于更新的规范。**
