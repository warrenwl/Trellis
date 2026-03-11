---
name: check-frontend
description: "检查你刚写的代码是否符合前端开发规范。"
---

# 前端代码检查

检查你刚写的代码是否符合前端开发规范。

执行以下步骤：
1. 运行 `git status` 查看修改的文件
2. 阅读 `.trellis/spec/frontend/index.md` 了解哪些规范适用
3. 根据你更改的内容，阅读相关的规范文件：
   - 组件更改 → `.trellis/spec/frontend/component-guidelines.md`
   - Hook 更改 → `.trellis/spec/frontend/hook-guidelines.md`
   - 状态更改 → `.trellis/spec/frontend/state-management.md`
   - 类型更改 → `.trellis/spec/frontend/type-safety.md`
   - 任何更改 → `.trellis/spec/frontend/quality-guidelines.md`
4. 根据规范检查你的代码
5. 报告任何违规行为并修复它们
