[!] **前提条件**：此命令只能在人类测试并提交代码后使用。

**AI 不得执行 git commit** - 只能读取历史（`git log`、`git status`、`git diff`）。

---

## 记录工作进度

### 步骤 1：获取上下文并检查任务

```bash
python3 ./.trellis/scripts/get_context.py --mode record
```

[!] 归档其工作**实际完成**的任务——根据工作状态判断，而不是 task.json 中的 `status` 字段：
- 代码已提交？→ 归档它（不要等 PR）
- 所有验收标准都满足？→ 归档它
- 不要仅仅因为 `status` 仍说 `planning` 或 `in_progress` 就跳过归档

```bash
python3 ./.trellis/scripts/task.py archive <task-name>
```

### 步骤 2：一键添加会话

```bash
# 方法 1：简单参数
python3 ./.trellis/scripts/add_session.py \
  --title "Session Title" \
  --commit "hash1,hash2" \
  --summary "Brief summary of what was done"

# 方法 2：通过 stdin 传递详细内容
cat << 'EOF' | python3 ./.trellis/scripts/add_session.py --title "Title" --commit "hash"
| Feature | Description |
|---------|-------------|
| New API | Added user authentication endpoint |
| Frontend | Updated login form |

**Updated Files**:
- `packages/api/modules/auth/router.ts`
- `apps/web/modules/auth/components/login-form.tsx`
EOF
```

**自动完成**：
- [OK] 将会话追加到 journal-N.md
- [OK] 自动检测行数，如果 >2000 行则创建新文件
- [OK] 更新 index.md（总会话数 +1，上次活跃，行数统计，历史）
- [OK] 自动提交 .trellis/workspace 和 .trellis/tasks 变更

---

## 脚本命令参考

| 命令 | 目的 |
|---------|---------|
| `python3 ./.trellis/scripts/get_context.py --mode record` | 获取 record-session 的上下文 |
| `python3 ./.trellis/scripts/add_session.py --title "..." --commit "..."` | **一键添加会话（推荐）** |
| `python3 ./.trellis/scripts/task.py archive <name>` | 归档已完成的任务（自动提交） |
| `python3 ./.trellis/scripts/task.py list` | 列出活动任务 |
