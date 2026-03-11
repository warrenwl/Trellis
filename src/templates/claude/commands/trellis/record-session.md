[!] **前置条件**：此命令只能在人类测试并提交代码后使用。

**AI 不得执行 git commit** - 只能读取历史（`git log`、`git status`、`git diff`）。

---

## 记录工作进度

### 步骤 1: 获取上下文并检查任务

```bash
python3 ./.trellis/scripts/get_context.py --mode record
```

[!] 归档其工作**实际完成**的任务——根据工作状态判断，而不是 task.json 中的 `status` 字段：
- 代码已提交？→ 归档它（不要等到 PR）
- 所有验收标准已满足？→ 归档它
- 不要仅仅因为 `status` 仍显示 `planning` 或 `in_progress` 就跳过归档

```bash
python3 ./.trellis/scripts/task.py archive <task-name>
```

### 步骤 2: 一键添加会话

```bash
# 方法 1：简单参数
python3 ./.trellis/scripts/add_session.py \
  --title "Session Title" \
  --commit "hash1,hash2" \
  --summary "Brief summary of what was done"

# 方法 2：通过 stdin 传递详细内容
cat << 'EOF' | python3 ./.trellis/scripts/add_session.py --title "Title" --commit "hash"
| 特性 | 描述 |
|---------|-------------|
| 新 API | 添加了用户认证端点 |
| 前端 | 更新了登录表单 |

**更新的文件**：
- `packages/api/modules/auth/router.ts`
- `apps/web/modules/auth/components/login-form.tsx`
EOF
```

**自动完成**：
- [OK] 追加会话到 journal-N.md
- [OK] 自动检测行数，超过 2000 行则创建新文件
- [OK] 更新 index.md（总会话数 +1、最后活跃、行统计、历史）
- [OK] 自动提交 .trellis/workspace 和 .trellis/tasks 更改

---

## 脚本命令参考

| 命令 | 目的 |
|---------|---------|
| `python3 ./.trellis/scripts/get_context.py --mode record` | 获取 record-session 的上下文 |
| `python3 ./.trellis/scripts/add_session.py --title "..." --commit "..."` | **一键添加会话（推荐）** |
| `python3 ./.trellis/scripts/task.py archive <name>` | 归档已完成的任务（自动提交）|
| `python3 ./.trellis/scripts/task.py list` | 列出活跃任务 |
