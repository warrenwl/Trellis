# Mock 策略

> 测试中 mock 的原则和模式。

---

## 核心原则：最小化 Mock

仅 mock **外部依赖**，且仅当它们是：
1. 非确定性（网络、时间、随机）
2. 交互式（TTY 提示）
3. 有副作用（子进程、写入系统路径的文件系统）

**永远不要 mock 内部模块** — 让真实代码执行完整路径。

---

## 标准 Mock 集

对于命令级集成测试，这是最小集合：

| 依赖 | 为什么 Mock | 如何 Mock | 用于 |
|------------|----------|-----|---------|
| `figlet` | ASCII 横幅，不可测试输出 | `vi.mock("figlet")` | init, update |
| `inquirer` | 交互式提示，CI 中无 TTY | `vi.mock("inquirer")` | init, update |
| `node:child_process` | Git 配置、Python 脚本调用 | `vi.mock("node:child_process")` | init, update |
| `fetch` (global) | npm registry 网络调用 | `vi.stubGlobal("fetch")` | update only |
| `process.cwd()` | 重定向到临时目录 | `vi.spyOn(process, "cwd")` | init, update |
| `console.log/error` | 静音输出 | `vi.spyOn(console, "log")` | init, update |

---

## Mock 模式

### 模块 Mock（提升）

```typescript
// 放在文件顶部 — vitest 提升 vi.mock 调用
vi.mock("figlet", () => ({
  default: { textSync: vi.fn(() => "TRELLIS") },
}));

vi.mock("inquirer", () => ({
  default: { prompt: vi.fn().mockResolvedValue({ proceed: true }) },
}));

vi.mock("node:child_process", () => ({
  execSync: vi.fn().mockReturnValue(""),
}));
```

### Global Stub

```typescript
// 在 beforeEach 中 — 不提升，必须在 setup 中
vi.stubGlobal("fetch", vi.fn().mockResolvedValue({
  ok: true,
  json: () => Promise.resolve({ version: VERSION }),
}));

// 在 afterEach 中 — 必须恢复
vi.unstubAllGlobals();
```

### Spy（部分 mock）

```typescript
// 在 beforeEach 中
vi.spyOn(process, "cwd").mockReturnValue(tmpDir);
vi.spyOn(console, "log").mockImplementation(noop);

// 在 afterEach 中
vi.restoreAllMocks(); // 恢复所有 spy
```

---

## inquirer Mock：init vs update

两个命令有不同的 inquirer 跳过条件：

**init**：`--yes` 标志跳过所有 inquirer 提示。Mock 可以返回空 `{}`。

```typescript
vi.mock("inquirer", () => ({
  default: { prompt: vi.fn().mockResolvedValue({}) },
}));
```

**update**：`--dryRun` 在确认提示前返回。所有其他模式（`force`、`skipAll`、`createNew`）仍会进入确认提示。Mock 必须返回 `{ proceed: true }`。

```typescript
vi.mock("inquirer", () => ({
  default: { prompt: vi.fn().mockResolvedValue({ proceed: true }) },
}));
```

---

## 不要 Mock 的东西

| 什么 | 为什么 |
|------|-----|
| `fs` (node:fs) | 测试在真实临时目录上运行 |
| `path` (node:path) | 纯计算，确定性 |
| 内部模块（`configurators/`、`utils/`、`templates/`）| 让真实代码执行 |
| `chalk` | 自动检测无 TTY 并禁用颜色 |

---

## 已知陷阱

### 模块级状态：`setWriteMode`

`file-writer.ts` 有模块级状态用于写入模式。如果一个测试设置 `force` 模式，后续测试继承它除非重置。`init()` 函数内部调用 `setWriteMode()`，因此调用 `init()` 的集成测试是安全的。但 `writeFile` 的直接单元测试必须显式管理此状态。

### 模板占位符解析

`collectPlatformTemplates()` 必须返回已解析的 `{{PYTHON_CMD}}` 的模板（与 `configurePlatform()` 写入磁盘的内容匹配）。`configurators/shared.ts` 中的 `resolvePlaceholders()` 函数处理此问题。如果向模板添加新占位符，必须在 `configure()` 和 `collectTemplates()` 中都解析。

---

## 做法 / 不做

### 做法

- 保持 mock 数量最小（目前 4 个外部依赖）
- 如果断言调用计数，在测试之间使用 `vi.mocked(fn).mockClear()`
- 将 mock 返回值与真实 API 形状匹配

### 不做

- 不要 mock 内部模块以强制特定代码路径
- 使用 `vi.stubGlobal` 时不要忘记 `vi.unstubAllGlobals()`
- 不要假设 mock 状态在测试之间重置，无需显式 `mockClear()` 或 `restoreAllMocks()`
