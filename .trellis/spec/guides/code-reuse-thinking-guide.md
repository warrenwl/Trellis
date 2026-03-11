# 代码复用思维指南

> **目的**：在创建新代码之前停下来思考——它已经存在了吗？

---

## 问题

**重复代码是 #1 不一致 bug 来源。**

当您复制粘贴或重写现有逻辑时：
- Bug 修复不会传播
- 行为随时间推移而分歧
- 代码库变得更难理解

---

## 编写新代码之前

### 第 1 步：先搜索

```bash
# 搜索相似的函数名
grep -r "functionName" .

# 搜索相似的逻辑
grep -r "keyword" .
```

### 第 2 步：问这些问题

| 问题 | 如果是... |
|----------|-----------|
| 类似的函数存在吗？ | 使用或扩展它 |
| 这个模式在其他地方使用吗？ | 遵循现有模式 |
| 这可以成为共享工具吗？ | 在正确位置创建它 |
| 您是从另一个文件复制代码吗？ | **停止** - 提取到共享位置 |

---

## 常见重复模式

### 模式 1：复制粘贴函数

**不好**：将验证函数复制到另一个文件

**好**：提取到共享工具，在需要的地方导入

### 模式 2：相似组件

**不好**：创建一个与现有组件 80% 相似的新组件

**好**：用 props/变体扩展现有组件

### 模式 3：重复常量

**不好**：在多个文件中定义相同的常量

**好**：单一真实来源，到处导入

---

## 何时抽象

**抽象当**：
- 相同代码出现 3+ 次
- 逻辑复杂到可能有 bug
- 多人可能需要这个

**不要抽象当**：
- 仅使用一次
- 简单的单行代码
- 抽象比重复更复杂

---

## 批量修改之后

当您对多个文件进行类似更改时：

1. **审查**：您是否捕获了所有实例？
2. **搜索**：运行 grep 找到任何遗漏
3. **考虑**：这应该被抽象吗？

---

## 提交前的检查清单

- [ ] 搜索了现有类似代码
- [ ] 没有应该共享的复制粘贴逻辑
- [ ] 常量定义在一个地方
- [ ] 相似模式遵循相同结构

---

## 陷阱：Python if/elif/else 穷尽检查

**问题**：Python 的 if/elif/else 链没有编译时穷尽检查。当您向 `Literal` 类型（例如 `Platform`）添加新值时，现有的 if/elif/else 链静默进入 `else` 分支并使用错误的默认值。

**症状**：新平台部分工作——某些方法返回 Claude 默认值而非平台特定值。没有错误引发。

**示例**（`cli_adapter.py`）：
```python
# 不好："gemini" 进入 else，返回 "claude"
@property
def cli_name(self) -> str:
    if self.platform == "opencode":
        return "opencode"
    else:
        return "claude"  # gemini 静默获得 "claude"！

# 好：每个平台显式分支
@property
def cli_name(self) -> str:
    if self.platform == "opencode":
        return "opencode"
    elif self.platform == "gemini":
        return "gemini"
    else:
        return "claude"
```

**预防**：当向 Python `Literal` 类型添加新值时，搜索所有切换该类型的 if/elif/else 链并添加显式分支。不要依赖 `else` 对新值是正确的。

---

## 陷阱：产生相同输出的不对称机制

**问题**：当两种不同机制必须产生相同的文件集时（例如，init 的递归目录复制 vs 更新的手动 `files.set()`），结构更改（重命名、移动、添加子目录）仅通过自动机制传播。手动机制静默漂移。

**症状**：init 完美工作，但 update 在错误路径创建文件或完全遗漏文件。

**预防检查清单**：
- [ ] 迁移目录结构时，搜索引用旧结构的所有代码路径
- [ ] 如果一个路径是自动派生（glob/copy）而另一个是手动列出，手动的需要更新
- [ ] 添加比较两种机制输出的回归测试

**另见**：`backend/platform-integration.md` → "collectTemplates 路径漂移" 的具体示例。

---

## 模板文件注册（Trellis 特定）

当向 `src/templates/trellis/scripts/` 添加新文件时：

**关键**：新脚本文件必须在**三处**注册：

1. **`src/templates/trellis/index.ts`**：
   - 添加 `export const xxxScript = readTemplate("scripts/path/file.py");`
   - 添加到 `getAllScripts()` Map

2. **`src/commands/update.ts`**：
   - 添加到导入语句
   - 添加到 `collectTemplateFiles()` Map

**为什么重要**：没有注册，`trellis update` 不会将文件同步到用户项目。Bug 修复和新功能不会传播。

### 新脚本的快速检查清单

```bash
# 添加新 .py 文件后，验证：
grep -l "newFileName" src/templates/trellis/index.ts  # 应该匹配
grep -l "newFileName" src/commands/update.ts          # 应该匹配
```
