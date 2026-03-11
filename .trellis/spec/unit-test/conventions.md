# 测试约定

> 文件命名、结构、断言模式。

---

## 测试基础设施

| 项目 | 值 |
|------|-------|
| 框架 | Vitest 4.x |
| 配置 | `vitest.config.ts` |
| 包含 | `test/**/*.test.ts` |
| 排除 | `third/**` |
| Lint 范围 | `eslint src/ test/` |
| 模块系统 | ESM (`"type": "module"` + `"module": "NodeNext"`) |
| 覆盖率提供者 | `@vitest/coverage-v8` |
| 覆盖率命令 | `pnpm test:coverage` |
| 覆盖率范围 | `src/**/*.ts`（排除 `src/cli/index.ts`）|
| 覆盖率报告 | `text`（终端）、`html`（`./coverage/index.html`）、`json-summary` |

---

## 何时编写测试

### 必须编写

| 更改类型 | 测试类型 | 示例 |
|-------------|-----------|---------|
| 新纯函数/工具函数 | 单元测试 | 添加 `compareVersions()` → 测试边界值 |
| 新平台 | 单元（由 `registry-invariants.test.ts` 自动覆盖）| 添加 opencode → 不变量验证一致性 |
| Bug 修复 | 回归测试 | 修复 Windows 编码 → 添加到 `regression.test.ts` |
| 更改 init/update 行为 | 集成测试 | 更改降级逻辑 → 在 `update.integration.test.ts` 中添加/更新场景 |

### 不需要测试

| 更改类型 | 原因 |
|-------------|--------|
| 模板文本/文档内容更改 | 无逻辑更改 |
| 新迁移清单 JSON | `registry-invariants.test.ts` 自动验证格式 |
| CLI 标志描述文本 | 仅显示用 |

### 必须更新现有测试

| 更改类型 | 要更新什么 |
|-------------|----------------|
| 新命令/技能添加到平台 | 在该平台的测试文件中添加到 `EXPECTED_COMMAND_NAMES` / `EXPECTED_SKILL_NAMES` |
| 新命令添加到任何平台 | 添加到**所有**平台测试文件（claude、cursor、iflow、codex）— 参见平台集成规范的必需命令列表 |

### 决策流程

```
这个更改有逻辑分支吗？
├─ 否（纯数据/文本）→ 不编写测试
└─ 是
   ├─ 独立函数有可预测的输入→输出？ → 单元测试
   ├─ 修复历史 bug？ → 回归测试（验证源中存在修复）
   └─ 更改 init/update 端到端行为？ → 集成测试
```

---

## 文件命名

```
test/
  types/
    ai-tools.test.ts          # src/types/ai-tools.ts 的单元测试
  commands/
    update-internals.test.ts   # 内部函数的单元测试
    init.integration.test.ts   # init() 命令的集成测试
    update.integration.test.ts # update() 命令的集成测试
  regression.test.ts           # 跨版本回归测试
```

**规则**：
- 在 `test/` 下镜像 `src/` 目录结构
- 后缀：`.test.ts` 用于单元测试，`.integration.test.ts` 用于集成测试
- 每个源模块一个测试文件（例外：回归测试）

---

## 测试结构

### 标准模式

```typescript
import { describe, it, expect } from "vitest";

describe("functionName", () => {
  it("does X when given Y", () => {
    const result = functionName(input);
    expect(result).toBe(expected);
  });
});
```

### 带设置/清理

```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from "vitest";

describe("module", () => {
  let tmpDir: string;

  beforeEach(() => {
    tmpDir = fs.mkdtempSync(path.join(os.tmpdir(), "trellis-test-"));
  });

  afterEach(() => {
    vi.restoreAllMocks();
    fs.rmSync(tmpDir, { recursive: true, force: true });
  });
});
```

---

## 断言模式

### 首选精确匹配器

```typescript
// 好：精确
expect(result).toBe("expected");
expect(array).toEqual(["a", "b"]);

// 避免：宽松
expect(result).toBeTruthy();
expect(array.length).toBeGreaterThan(0);
```

### 无操作验证的快照比较

当断言操作零更改时，使用完整目录快照：

```typescript
// 收集操作前所有文件 + 内容
const before = new Map<string, string>();
walk(dir, (filePath, content) => before.set(filePath, content));

// 运行操作
await operation();

// 收集之后并对比
const after = new Map<string, string>();
walk(dir, (filePath, content) => after.set(filePath, content));

const added = [...after.keys()].filter((k) => !before.has(k));
const removed = [...before.keys()].filter((k) => !after.has(k));
expect(added).toEqual([]);
expect(removed).toEqual([]);
```

---

## ESLint 兼容性

测试必须通过与 `src/` 相同的 ESLint 规则。常见解决方法：

```typescript
// 空函数（no-empty-function 规则）
// eslint-disable-next-line @typescript-eslint/no-empty-function
const noop = () => {};
vi.spyOn(console, "log").mockImplementation(noop);

// 避免非空断言
// 不好：match![0]
// 好：(match as [unknown])[0]
```

---

## 测试反模式

测试应该验证**有意义的行为**，而不是重述 TypeScript 或运行时已经保证的内容。以下反模式是在全面测试审计期间发现的，应该避免。

### 增长数据的硬编码计数

```typescript
// 不好 - 每次添加清单/脚本时都断开
expect(scripts.size).toBe(23);
expect(versions.length).toBe(23);

// 好 - 来自真实来源的动态计数
const jsonFiles = fs.readdirSync(manifestDir).filter(f => f.endsWith(".json"));
expect(versions.length).toBe(jsonFiles.length);
expect(versions.length).toBeGreaterThan(0);
```

**原因**：硬编码计数会在无关更改时创建误报失败，需要不断手动更新。

### 同义反复断言

```typescript
// 不好 - 测试 registry[key] === registry[key]
const config = getToolConfig(id);
expect(config).toBe(AI_TOOLS[id]); // getToolConfig 只是返回 AI_TOOLS[id]

// 不好 - 测试函数返回自己的输入
const dirs = getTemplateDirs(id);
expect(dirs).toEqual(AI_TOOLS[id].templateDirs); // getTemplateDirs 只是返回 .templateDirs
```

**原因**：这些测试验证的是 JavaScript 对象属性访问是否工作，而不是我们的代码是否正确。如果是简单的查找，不要测试它 — 改为测试**消费者行为**。

### 冗余类型检查（TypeScript 保证）

```typescript
// 不好 - TypeScript 已在编译时保证这些
expect(typeof settingsTemplate).toBe("string");
expect(Array.isArray(commands)).toBe(true);
expect(typeof cmd.name).toBe("toBe");

// 好：改为测试有意义的属性
expect(settingsTemplate.length).toBeGreaterThan(0);
expect(commands.length).toBeGreaterThan(0);
```

**原因**：在严格的 TypeScript 项目中，测试中的运行时类型检查增加噪音而不捕获真实 bug。

### 跨文件重复覆盖

```typescript
// 不好：registry-invariants.test.ts 和 index.test.ts 都测试：
// - PLATFORM_IDS 长度匹配 AI_TOOLS 键
// - cliFlag 唯一性
// - configDir 以点开头

// 好：在 ONE 规范位置测试每个不变量
// registry-invariants.test.ts：内部一致性（唯一标志、无冲突、保留名称）
// index.test.ts：派生辅助函数正确性（getConfiguredPlatforms、isManagedPath 等）
```

**原因**：重复测试给予虚假的覆盖感，使重构更困难，并增加维护负担。

### 测试内的冗余断言

```typescript
// 不好：解析测试已经证明它是有效的 JSON 字符串
it("is valid JSON", () => {
  expect(() => JSON.parse(settingsTemplate)).not.toThrow();
});
it("is a non-empty string", () => { // 如果解析成功则冗余
  expect(settingsTemplate.length).toBeGreaterThan(0);
});

// 好：合并为一个有意义的断言
it("is valid non-empty JSON", () => {
  const parsed = JSON.parse(settingsTemplate);
  expect(parsed).toBeTruthy();
});
```

### 重构后的过时回归测试

```typescript
// 不好：回归测试检查代码移动后的旧位置
it("[beta.10] git_context.py has inline encoding fix", () => {
  expect(commonGitContext).toContain('sys.platform == "win32"');  // 移动到 __init__.py 了！
});

// 好：更新为检查新位置
it("[beta.10] common/__init__.py has centralized encoding fix", () => {
  expect(commonInit).toContain('sys.platform == "win32"');
});
```

**原因**：当重构将代码移动到文件之间时（例如，将编码从各个脚本集中到 `common/__init__.py`），检查特定文件中特定字符串的回归测试将断开。回归仍然被防止 — 只是在不同的文件中。

**预防**：重构跨文件代码时，在 `test/regression.test.ts` 中搜索受影响文件的引用，并更新断言以匹配新位置。

### 决策规则

编写测试前，问：

1. **TypeScript 已经保证这个？** → 跳过（typeof、Array.isArray、属性存在）
2. **这是一个简单的直通？** → 跳过（返回属性的 getter）
3. **这在其他地方已经测试过？** → 跳过（避免跨文件重复）
4. **这依赖于随时间增长的数据？** → 使用动态计数
5. **这测试真实行为还是只是重述实现？** → 只测试行为

---

## 做法 / 不做

### 做法

- 每个测试使用独立的临时目录（无共享状态）
- 在 `afterEach` 中清理临时目录
- 在 `afterEach` 中用 `vi.restoreAllMocks()` 恢复所有 mock
- 使用 `vi.mocked()` 进行类型安全的 mock 访问
- 为可追溯性到 PRD 编号测试场景（`#1`、`#2`、...）
- 使用从真实来源（文件系统、注册表）派生的动态计数
- 测试有意义的行为，而非实现细节

### 不做

- 不要依赖测试执行顺序
- 不要使用定时器、网络或全局状态
- 测试完成后不要留下临时文件
- 不要在测试文件中使用 `any`（相同的 ESLint 规则适用）
- 使用 `vi.stubGlobal` 时不要忘记 `vi.unstubAllGlobals()`
- 不要对增长的数据集（清单、脚本、平台）硬编码计数
- 不要在 TypeScript 测试中添加 `typeof` 或 `Array.isArray` 检查
- 不要在多个测试文件中重复相同的断言
- 不要编写只是验证 `x === x` 的同义反复测试
