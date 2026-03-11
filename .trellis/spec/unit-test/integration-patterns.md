# 集成测试模式

> CLI 命令的函数级集成测试模式。

---

## 方法：函数级集成（方法 B）

不生成 CLI 子进程，而是在真实临时目录中直接导入并调用 `init()` / `update()` 函数。这提供：

- 快速执行（~400ms 每个测试文件）
- 可重现结果（无网络，无 TTY）
- 通过 mock 精确控制外部依赖
- 从入口到文件系统输出的完整代码路径覆盖

**权衡**：不测试 CLI 参数解析（commander 层）。

---

## 标准测试设置

```typescript
describe("command() integration", () => {
  let tmpDir: string;

  beforeEach(() => {
    tmpDir = fs.mkdtempSync(path.join(os.tmpdir(), "trellis-test-"));
    vi.spyOn(process, "cwd").mockReturnValue(tmpDir);
    vi.spyOn(console, "log").mockImplementation(noop);
    vi.spyOn(console, "error").mockImplementation(noop);
  });

  afterEach(() => {
    vi.restoreAllMocks();
    vi.unstubAllGlobals();  // 仅当使用了 vi.stubGlobal 时需要
    fs.rmSync(tmpDir, { recursive: true, force: true });
  });
});
```

---

## 常见模式

### 模式：设置项目（用于更新测试）

更新测试需要初始化项目作为前置条件：

```typescript
async function setupProject(): Promise<void> {
  await init({ yes: true, force: true });
}

it("test case", async () => {
  await setupProject();
  // ... 修改状态 ...
  await update({ force: true });
  // ... 断言结果 ...
});
```

### 模式：完整快照比较

用于验证操作是真正的无操作：

```typescript
const snapshotBefore = new Map<string, string>();
const walk = (dir: string) => {
  for (const entry of fs.readdirSync(dir, { withFileTypes: true })) {
    const full = path.join(dir, entry.name);
    if (entry.isDirectory()) walk(full);
    else snapshotBefore.set(path.relative(tmpDir, full), fs.readFileSync(full, "utf-8"));
  }
};
walk(tmpDir);

await operation();

// 比较：无添加，无删除，无更改
```

### 模式：模拟模板版本更改

测试自动更新检测（模板更改，用户未修改）：

```typescript
// 1. 将"旧内容"写入模板文件
const oldContent = "# Old version\n";
fs.writeFileSync(targetFull, oldContent);

// 2. 更新哈希文件以匹配旧内容（以便更新认为用户未修改它）
const hashes = JSON.parse(fs.readFileSync(hashFile, "utf-8"));
hashes[targetRelative] = computeHash(oldContent);
fs.writeFileSync(hashFile, JSON.stringify(hashes, null, 2));

// 3. 运行更新 — 应该自动更新到当前模板
await update({ force: true });
expect(fs.readFileSync(targetFull, "utf-8")).toBe(currentTemplateContent);
```

### 模式：降级保护

```typescript
// 设置项目版本到未来
fs.writeFileSync(versionPath, "99.99.99");

await update({});

// 版本不应更改 — 更新拒绝降级
expect(fs.readFileSync(versionPath, "utf-8")).toBe("99.99.99");
```

---

## 测试矩阵设计

集成测试场景应组织为 PRD 中的编号矩阵：

| # | 场景 | 选项 | 验证 |
|---|----------|---------|--------------|
| 1 | 无操作（相同版本） | `{}` | 零文件更改，无备份 |
| 2 | 试运行 | `{ dryRun: true }` | 无文件修改 |
| 3 | 删除文件重新创建 | `{ force: true }` | 文件恢复 |
| ... | | | |

每个测试都编号（`#1`、`#2`、...）匹配矩阵以实现可追溯性。

---

## 通过集成测试发现的 Bug

集成测试在发现**跨模块不一致**方面很有效：

1. **模板占位符往返**：`init` 解析 `{{PYTHON_CMD}}` → `python3`，但 `update` 与原始 `{{PYTHON_CMD}}` 比较。每次更新都检测到虚假更改。

2. **模板列表不匹配**：`update` 列出 `init` 未创建的文件，导致同版本更新时出现幻"新文件"检测。

3. **项目类型条件模板被忽略**：`createSpecTemplates()` 接受 `projectType` 但忽略它（参数名 `_projectType`），始终创建后端 + 前端规范。`collectTemplateFiles()` 无条件包含所有规范文件，无论哪些目录实际存在。纯后端项目在 init 时获得空前端规范目录，而 update 始终跟踪前端文件，即使目录已被删除。

这三个 bug 对单元测试（隔离测试模块）是不可见的，但在测试完整 init→update 流程时立即显现。

---

## 做法 / 不做

### 做法

- 使用真实文件系统操作（不 mock fs）
- 测试完整流程：入口函数 → 文件系统输出
- 验证正面结果（文件创建）和负面结果（文件未更改）
- 每次测试后清理临时目录

### 不做

- 不要 mock 内部模块以模拟模板更改 — 改为使用文件系统操作
- 不要在测试之间共享临时目录
- 不要在断言中依赖特定模板内容（使用 `computeHash` 或从 init 输出读取）
