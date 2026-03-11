---
name: check-backend
description: "检查您刚写的代码是否符合后端开发指南。"
---

# 后端代码检查

检查您刚写的代码是否符合后端开发指南。

请执行以下步骤：
1. 运行 `git status` 查看修改的文件
2. 阅读 `.trellis/spec/backend/index.md` 了解哪些指南适用
3. 根据您修改的内容，阅读相关的指南文件：
   - 数据库更改 → `.trellis/spec/backend/database-guidelines.md`
   - 错误处理 → `.trellis/spec/backend/error-handling.md`
   - 日志更改 → `.trellis/spec/backend/logging-guidelines.md`
   - 类型更改 → `.trellis/spec/backend/type-safety.md`
   - 任何更改 → `.trellis/spec/backend/quality-guidelines.md`
4. 根据指南检查您的代码
5. 报告任何违规行为并在发现时修复它们
