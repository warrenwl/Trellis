---
name: check-backend
description: "检查你刚写的代码是否符合后端开发规范。"
---

# 后端代码检查

检查你刚写的代码是否符合后端开发规范。

执行以下步骤：
1. 运行 `git status` 查看修改的文件
2. 阅读 `.trellis/spec/backend/index.md` 了解哪些规范适用
3. 根据你更改的内容，阅读相关的规范文件：
   - 数据库更改 → `.trellis/spec/backend/database-guidelines.md`
   - 错误处理 → `.trellis/spec/backend/error-handling.md`
   - 日志更改 → `.trellis/spec/backend/logging-guidelines.md`
   - 类型更改 → `.trellis/spec/backend/type-safety.md`
   - 任何更改 → `.trellis/spec/backend/quality-guidelines.md`
4. 根据规范检查你的代码
5. 报告任何违规行为并修复它们
