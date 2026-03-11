# 质量指南

> 后端/CLI 开发的代码质量标准。

---

## 概述

本项目强制执行严格的 TypeScript 和 ESLint 规则以保持代码质量。配置优先考虑类型安全、显式声明和现代 JavaScript 模式。

---

## TypeScript 配置

### 严格模式

项目在 `tsconfig.json` 中使用 `strict: true`：

```json
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext"
  }
}
```

启用以下功能：
- `strictNullChecks` - 必须显式处理 null 和 undefined
- `strictFunctionTypes` - 严格检查函数参数类型
- `strictPropertyInitialization` - 类属性必须初始化
- `noImplicitAny` - 所有类型必须显式
- `noImplicitThis` - `this` 必须有显式类型

---

## ESLint 规则

### 禁止的模式

| 规则 | 设置 | 原因 |
|------|---------|--------|
| `@typescript-eslint/no-explicit-any` | `error` | 强制正确类型化 |
| `@typescript-eslint/no-non-null-assertion` | `error` | 防止运行时空错误 |
| `no-var` | `error` | 使用 `const` 或 `let` |

### 必需的模式

| 规则 | 设置 | 描述 |
|------|---------|-------------|
| `@typescript-eslint/explicit-function-return-type` | `error` | 所有函数必须声明返回类型 |
| `@typescript-eslint/prefer-nullish-coalescing` | `error` | 默认值使用 `??` 而非 `\|\|` |
| `@typescript-eslint/prefer-optional-chain` | `error` | 可选访问使用 `?.` |
| `prefer-const` | `error` | 变量不重新赋值时使用 `const` |

### 异常

```javascript
// eslint.config.js
rules: {
  "@typescript-eslint/explicit-function-return-type": [
    "error",
    {
      allowExpressions: true,          // 回调中的箭头函数可以
      allowTypedFunctionExpressions: true,  // 类型化函数表达式可以
    },
  ],
  "@typescript-eslint/no-unused-vars": [
    "error",
    {
      argsIgnorePattern: "^_",   // 未使用的参数前缀加 _
      varsIgnorePattern: "^_",   // 未使用的变量前缀加 _
    },
  ],
}
```

---

## 代码模式

### 返回类型声明

所有函数必须有显式返回类型：

```typescript
// 好：显式返回类型
function detectProjectType(cwd: string): ProjectType {
  // ...
}

async function init(options: InitOptions): Promise<void> {
  // ...
}

// 不好：缺少返回类型（ESLint 错误）
function detectProjectType(cwd: string) {
  // ...
}
```

### 空值合并

默认值使用 `??`，而非 `||`：

```typescript
// 好：空值合并
const name = options.name ?? "default";
const allDeps = { ...pkg.dependencies, ...pkg.devDependencies };
const depNames = Object.keys(allDeps ?? {});

// 不好：逻辑或（将空字符串、0 视为 falsy）
const name = options.name || "default";
```

### 可选链

可选属性访问使用 `?.`：

```typescript
// 好：可选链
const version = config?.version;
const deps = pkg?.dependencies?.["react"];

// 不好：手动检查
const version = config && config.version;
```

### Const 声明

默认使用 `let`，仅在需要重新赋值时使用 `let`：

```typescript
// 好：const 用于不重新赋值
const cwd = process.cwd();
const options: InitOptions = { force: true };

// 好：let 用于重新赋值
let developerName = options.user;
if (!developerName) {
  developerName = detectFromGit();
}

// 不好：let 用于不重新赋值
let cwd = process.cwd();  // ESLint 错误：prefer-const
```

### 未使用的变量

未使用的参数加下划线前缀：

```typescript
// 好：加下划线前缀
function handler(_req: Request, res: Response): void {
  res.send("OK");
}

// 不好：未使用且无前缀（ESLint 错误）
function handler(req: Request, res: Response): void {
  res.send("OK");
}
```

---

## 接口和类型模式

### 接口定义

为结构化数据定义接口：

```typescript
// 好：选项的接口
interface InitOptions {
  cursor?: boolean;
  claude?: boolean;
  yes?: boolean;
  user?: string;
  force?: boolean;
}

// 好：返回类型的接口
interface WriteOptions {
  mode: WriteMode;
}
```

### 类型别名

联合和计算类型使用类型别名：

```typescript
// 好：联合的类型别名
export type AITool = "claude-code" | "cursor" | "opencode";
export type WriteMode = "ask" | "force" | "skip" | "append";
export type ProjectType = "frontend" | "backend" | "fullstack" | "unknown";

// 好：带 const 断言的类型别名
export const DIR_NAMES = {
  WORKFLOW: ".trellis",
  PROGRESS: "agent-traces",
} as const;
```

### 导出模式

显式导出类型：

```typescript
// 好：显式类型导出
export type { WriteMode, WriteOptions };
export { writeFile, ensureDir };

// 好：组合导出
export type WriteMode = "ask" | "force" | "skip" | "append";
export function writeFile(path: string, content: string): Promise<boolean> {
  // ...
}
```

---

## 禁止的模式

### 永远不要使用 `any`

```typescript
// 不好：显式 any
function process(data: any): void { }

// 好：正确类型化
function process(data: Record<string, unknown>): void { }
function process<T>(data: T): void { }
```

### 永远不要使用非空断言

```typescript
// 不好：非空断言
const name = user!.name;

// 好：正确的空检查
const name = user?.name ?? "default";
if (user) {
  const name = user.name;
}
```

### 永远不要使用 `var`

```typescript
// 不好：var 声明
var count = 0;

// 好：const 或 let
const count = 0;
let mutableCount = 0;
```

---

## 质量检查清单

提交前，确保：

- [ ] `pnpm lint` 通过且无错误
- [ ] `pnpm typecheck` 通过且无错误
- [ ] 所有函数都有显式返回类型
- [ ] 代码中无 `any` 类型
- [ ] 无非空断言（`x!` 操作符）
- [ ] 默认值使用 `??` 而非 `||`
- [ ] 可选属性访问使用 `?.`
- [ ] 默认使用 `const`，仅在需要时使用 `let`
- [ ] 未使用的变量前缀加 `_`

---

## 运行质量检查

```bash
# 运行 ESLint
pnpm lint

# 运行 TypeScript 类型检查
pnpm typecheck

# 运行两者
pnpm lint && pnpm typecheck
```

---

## CLI 设计模式

### 显式标志优先

当 CLI 同时有显式标志（`--tool`）和便捷标志（`-y`）时，显式标志必须始终获胜：

```typescript
// 不好：-y 覆盖显式标志
if (options.yes) {
  tools = ["cursor", "claude"]; // 忽略 --iflow, --opencode!
} else if (options.cursor || options.iflow) {
  // 从标志构建...
}

// 好：首先检查显式标志
const hasExplicitTools = options.cursor || options.iflow || options.opencode;
if (hasExplicitTools) {
  // 从显式标志构建（带或不带 -y 都可以工作）
} else if (options.yes) {
  // 仅在无显式标志时使用默认值
}
```

**原因**：用户是有意指定显式标志的。`-y` 标志意味着"跳过交互式提示"，而不是"忽略我的其他标志"。

### 数据驱动配置

处理多个类似选项时，使用带元数据的数组而非重复的 if-else：

```typescript
// 不好：重复的 if-else
if (options.cursor) tools.push("cursor");
if (options.claude) tools.push("claude");
if (options.iflow) tools.push("iflow");
// ... 重复逻辑，容易遗漏一个

// 好：数据驱动方式
const TOOLS = [
  { key: "cursor", name: "Cursor", defaultChecked: true },
  { key: "claude", name: "Claude Code", defaultChecked: true },
  { key: "iflow", name: "iFlow CLI", defaultChecked: false },
] as const;

// 单一真实来源用于：
// - 从标志构建：TOOLS.filter(t => options[t.key])
// - 交互式选择：TOOLS.map(t => ({ name: t.name, value: t.key }))
// - 默认值：TOOLS.filter(t => t.defaultChecked)
```

**好处**：
- 添加新工具 = 向 TOOLS 数组添加一行
- 显示名称、标志键和默认值放在一起
- 代码重复更少，bug 更少

### 自动检测模式必须在所有代码路径中探测

当 CLI 自动检测模式（例如，市场 vs 直接下载）通过探测资源时，探测必须在使用结果的**每个**代码路径中运行 - 包括 `-y`（非交互）模式：

```typescript
// 不好：探测仅在交互模式运行
let templates: Item[] = [];
if (!options.yes) {
  templates = await fetchIndex(url); // 仅交互式探测
}
// -y 模式：templates 保持 []，直接进入直接模式
// Bug：市场注册表作为原始目录被静默下载

// 好：在需要结果的所有路径中探测
if (options.template) {
  selectedTemplate = options.template; // 显式：无需探测
} else if (!options.yes) {
  // 交互：探测 + 显示选择器
  const result = await probeIndex(url);
  // ...
} else if (registry) {
  // 带注册表的 -y 模式：仍需要探测
  const result = await probeIndex(url);
  if (result.templates.length > 0) {
    // 市场需要选择 — -y 模式无法自动选择
    console.error("Use --template to specify which template");
    return;
  }
}
```

**原因**：`-y` 标志意味着"跳过交互式提示"，而不是"跳过网络操作"。如果模式决策依赖于远程资源，无论是否交互都必须进行探测。

### 重建复合标识符时不要丢弃字段

当解析结构化对象为其组成部分然后重新组装时，包含**所有**解析的字段：

```typescript
// 不好：ref 被解析但重建时被丢弃
const registry = parseSource("gh:org/repo/path#develop");
// registry = { provider: "gh", repo: "org/repo", ref: "develop", ... }
const repoSource = `${registry.provider}:${registry.repo}`;
// 结果："gh:org/repo" — ref "develop" 丢失，默认为 "main"

// 好：包含所有相关字段
const repoSource = `${registry.provider}:${registry.repo}#${registry.ref}`;
// 结果："gh:org/repo#develop"
```

**预防**：从解析对象构建字符串时，审查对象的字段并验证每个字段要么被包含，要么明确不相关。

### 不要：对模式检测逻辑"警告并继续"

当代码根据探测结果决定运行哪种模式时，警告 + 继续在功能上等同于没有修复：

```typescript
// 不好：警告打印但代码仍进入错误模式
if (!probeResult.isNotFound) {
  console.log(chalk.yellow("Warning: network issue, attempting direct download"));
}
// 继续 → 将市场根目录下载为 spec 目录

// 好：中止或循环 — 永远不要静默切换模式
if (!probeResult.isNotFound) {
  console.log(chalk.red("Could not reach registry. Check connection and retry."));
  return; // 或：continue（循环回选择器）
}
```

**原因**："警告并继续"适用于**降级功能**（缺少可选数据）。它不适用于**模式决策** — 错误的模式导致数据损坏，而不仅仅是降级的用户体验。

### 约定：切换分支时重置共享状态

当用户输入或控制流改变上下文时（例如，从官方市场切换到自定义源），重置由前一个上下文填充的任何共享状态：

```typescript
// 不好：fetchedTemplates 仍有官方市场结果
registry = parseRegistrySource(customSource);
// fetchedTemplates.length > 0 → 直接下载守卫永不触发！

// 好：在进入新上下文前重置
registry = parseRegistrySource(customSource);
fetchedTemplates = []; // 清除先前来源的陈旧数据
```

**原因**：跨分支的共享可变状态是静默的 bug 工厂。后面的守卫（`registry && fetchedTemplates.length === 0`）取决于 `fetchedTemplates` 反映的是*当前*来源，而不是之前的来源。

---

## 做法 / 不做

### 做法

- 在所有函数上声明显式返回类型
- 默认使用 `const`
- 默认值使用 `??`
- 可选访问使用 `?.`
- 为结构化数据定义接口
- 未使用的参数前缀加 `_`

### 不做

- 不要使用 `any` 类型
- 不要使用非空断言（`x!` 操作符）
- 不要使用 `var`
- 不要使用 `||` 作为默认值（使用 `??`）
- 不要留下隐式返回类型
- 不要忽略 ESLint 或 TypeScript 错误
