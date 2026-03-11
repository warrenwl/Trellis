# 目录结构

> 后端/CLI 代码在本项目中的组织方式。

---

## 概述

本项目是一个使用 ES 模块的 **TypeScript CLI 工具**。源代码遵循 **自产自用（dogfooding）架构** - Trellis 使用自己的配置文件（`.cursor/`、`.claude/`、`.trellis/`）作为新项目的模板。

---

## 目录布局

```
src/
├── cli/                 # CLI 入口和参数解析
│   └── index.ts         # 主 CLI 入口（Commander.js 设置）
├── commands/            # 命令实现
│   └── init.ts          # 每个命令在各自的文件中
├── configurators/       # 配置生成器
│   ├── index.ts         # 平台注册表（PLATFORM_FUNCTIONS，派生辅助函数）
│   ├── shared.ts        # 共享工具（resolvePlaceholders）
│   ├── claude.ts        # Claude Code 配置器
│   ├── cursor.ts        # Cursor 配置器
│   ├── iflow.ts         # iFlow CLI 配置器
│   ├── opencode.ts      # OpenCode 配置器
│   └── workflow.ts      # 创建 .trellis/ 结构
├── constants/           # 共享常量和路径
│   └── paths.ts         # 路径常量（集中管理）
├── templates/           # 模板工具和通用模板
│   ├── markdown/        # 通用 markdown 模板
│   │   ├── spec/        # 规范模板（*.md.txt）
│   │   ├── init-agent.md    # 项目根文件模板
│   │   ├── agents.md        # 项目根文件模板
│   │   ├── worktree.yaml.txt # 通用 worktree 配置
│   │   └── index.ts     # 模板导出
│   └── extract.ts       # 模板提取工具
├── types/               # TypeScript 类型定义
│   └── ai-tools.ts      # AI 工具类型和注册表
├── utils/               # 共享工具函数
│   ├── compare-versions.ts # 带预发布支持的语义版本比较
│   ├── file-writer.ts   # 带冲突处理的文件写入
│   ├── project-detector.ts # 项目类型检测
│   ├── template-fetcher.ts # 从 GitHub 下载远程模板
│   └── template-hash.ts # 用于更新检测的模板哈希追踪
└── index.ts             # 包入口点（导出公共 API）
```

### 自产自用目录（项目根目录）

这些目录在构建时被复制到 `dist/` 并用作模板：

```
.cursor/                 # Cursor 配置（自产自用）
├── commands/            # Cursor 斜杠命令
│   ├── start.md
│   ├── finish-work.md
│   └── ...

.claude/                 # Claude Code 配置（自产自用）
├── commands/            # 斜杠命令
├── agents/              # 多代理流水线代理
├── hooks/               # 上下文注入钩子
└── settings.json        # 钩子配置

.trellis/                # Trellis 工作流（部分自产自用）
├── scripts/             # Python 脚本（自产自用）
│   ├── common/          # 共享工具（paths.py、developer.py 等）
│   ├── multi_agent/     # 流水线脚本（start.py、status.py 等）
│   ├── hooks/          # 生命周期钩子脚本（项目特定，非自产自用）
│   └── *.py            # 主脚本（task.py、get_context.py 等）
├── workspace/           # 开发者进度追踪
│   └── index.md         # 索引模板（自产自用）
├── spec/                # 项目指南（非自产自用）
│   ├── backend/         # 后端开发文档
│   ├── frontend/        # 前端开发文档
│   └── guides/          # 思维指南
├── workflow.md          # 工作流文档（自产自用）
├── worktree.yaml        # Worktree 配置（Trellis 特定）
└── .gitignore           # Git 忽略规则（自产自用）
```

---

## 自产自用架构

### 什么是自产自用

直接从 Trellis 项目复制到用户项目的文件：

| 源文件 | 目标位置 | 描述 |
|--------|---------|------|
| `.cursor/` | `.cursor/` | 整个目录复制 |
| `.claude/` | `.claude/` | 整个目录复制 |
| `.trellis/scripts/` | `.trellis/scripts/` | 复制所有脚本 |
| `.trellis/workflow.md` | `.trellis/workflow.md` | 直接复制 |
| `.trellis/.gitignore` | `.trellis/.gitignore` | 直接复制 |
| `.trellis/workspace/index.md` | `.trellis/workspace/index.md` | 直接复制 |

### 什么不是自产自用

使用通用模板的文件（在 `src/templates/` 中）：

| 模板源 | 目标位置 | 原因 |
|-------|---------|------|
| `src/templates/markdown/spec/**/*.md.txt` | `.trellis/spec/**/*.md` | 用户填写项目特定内容 |
| `src/templates/markdown/worktree.yaml.txt` | `.trellis/worktree.yaml` | 语言无关模板 |
| `src/templates/markdown/init-agent.md` | `init-agent.md` | 项目根文件 |
| `src/templates/markdown/agents.md` | `AGENTS.md` | 项目根文件 |

### 构建流程

```bash
# scripts/copy-templates.js 将自产自用源复制到 dist/
pnpm build

# 结果：
dist/
├── .cursor/           # 来自项目根目录 .cursor/
├── .claude/           # 来自项目根目录 .claude/
├── .trellis/          # 来自项目根目录 .trellis/（过滤后）
│   ├── scripts/       # 所有脚本
│   ├── workspace/
│   │   └── index.md   # 只有 index.md，没有开发者子目录
│   ├── workflow.md
│   ├── worktree.yaml
│   └── .gitignore
└── templates/         # 来自 src/templates/（无 .ts 文件）
    └── markdown/
        └── spec/      # 通用模板
```

---

## 模块组织

### 层级职责

| 层级 | 目录 | 职责 |
|-----|------|------|
| CLI | `cli/` | 解析参数，显示帮助，调用命令 |
| Commands | `commands/` | 实现 CLI 命令，协调操作 |
| Configurators | `configurators/` | 为工具复制/生成配置 |
| Templates | `templates/` | 提取模板内容，提供工具 |
| Types | `types/` | TypeScript 类型定义 |
| Utils | `utils/` | 可复用的工具函数 |
| Constants | `constants/` | 共享常量（路径、名称） |

### 配置器模式

配置器使用 `cpSync` 进行目录直接复制（自产自用）：

```typescript
// configurators/cursor.ts
export async function configureCursor(cwd: string): Promise<void> {
  const sourcePath = getCursorSourcePath(); // dist/.cursor/ 或 .cursor/
  const destPath = path.join(cwd, ".cursor");
  cpSync(sourcePath, destPath, { recursive: true });
}
```

### 模板提取

`extract.ts` 提供读取自产自用文件的工具：

```typescript
// 获取 .trellis/ 的路径（开发和生产环境都可用）
getTrellisSourcePath(): string

// 从 .trellis/ 读取文件
readTrellisFile(relativePath: string): string

// 从 .trellis/ 复制目录，保留可执行脚本
copyTrellisDir(srcRelativePath: string, destPath: string, options?: { executable?: boolean }): void
```

---

## 命名约定

### 文件和目录

| 约定 | 示例 | 用途 |
|-----|------|-----|
| `kebab-case` | `file-writer.ts` | 所有 TypeScript 文件 |
| `kebab-case` | `multi-agent/` | 所有目录 |
| `*.ts` | `init.ts` | TypeScript 源文件 |
| `*.md.txt` | `index.md.txt` | markdown 模板文件 |
| `*.yaml.txt` | `worktree.yaml.txt` | yaml 模板文件 |

### 为什么模板使用 `.txt` 扩展名

模板使用 `.txt` 扩展名是为了：
- 防止 IDE markdown 预览渲染模板
- 明确这些是模板源，不是实际文档
- 避免与实际 markdown 文件混淆

---

## 做 / 不做

### 做

- 尽可能从项目自己的配置文件进行自产自用
- 使用 `cpSync` 复制整个目录
- 将通用模板保存在 `src/templates/markdown/`
- 对模板文件使用 `.md.txt` 或 `.yaml.txt`
- 修改自产自用源（`.cursor/`、`.claude/`、`.trellis/scripts/`）时也要更新
- 记录脚本调用时始终明确使用 `python3`（Windows 兼容性）

### 不做

- 不要硬编码文件列表 - 而应复制整个目录
- 不要在模板和自产自用源之间重复内容
- 不要将项目特定内容放在通用模板中
- 不要对 spec/ 使用自产自用（用户需要填写这些）

---

## 设计决策

### 远程模板下载（giget）

**背景**：需要下载 GitHub 子目录以支持远程模板。

**考虑的选项**：
1. `degit` / `tiged` - 简单，但没有程序化 API
2. `giget` - TypeScript 原生，有程序化 API，被 Nuxt/UnJS 使用
3. 手动 GitHub API - 太复杂

**决策**：使用 `giget`，因为：
- TypeScript 原生，带程序化 API
- 支持 GitHub 子目录：`gh:user/repo/path/to/subdir`
- 内置缓存支持离线
- 由 UnJS 生态系统积极维护

**示例**：
```typescript
import { downloadTemplate } from "giget";

await downloadTemplate("gh:mindfold-ai/docs/marketplace/specs/electron-fullstack", {
  dir: destDir,
  preferOffline: true,
});
```

### 目录冲突策略（skip/overwrite/append）

**背景**：下载远程模板时，目标目录可能已存在。

**决策**：三种策略，默认 `skip`：
- `skip` - 如果目录存在则不下载（安全默认值）
- `overwrite` - 删除现有内容，重新下载
- `append` - 只复制不存在的文件（合并）

**原因**：giget 原生不支持 append，所以我们：
1. 下载到临时目录
2. 遍历并只复制缺失的文件
3. 清理临时目录

**示例**：
```typescript
// append 策略实现
const tempDir = path.join(os.tmpdir(), `trellis-template-${Date.now()}`);
await downloadTemplate(source, { dir: tempDir });
await copyMissing(tempDir, destDir);  // 只复制不存在的文件
await fs.promises.rm(tempDir, { recursive: true });
```

### 可扩展模板类型映射

**背景**：目前只有 `spec` 模板，但未来需要 `skill`、`command`、`full` 类型。

**决策**：使用类型字段 + 映射表实现可扩展性：

```typescript
const INSTALL_PATHS: Record<string, string> = {
  spec: ".trellis/spec",
  skill: ".claude/skills",
  command: ".claude/commands",
  full: ".",  // 整个项目根目录
};

// 用法：根据模板类型自动检测安装路径
const destDir = INSTALL_PATHS[template.type] || INSTALL_PATHS.spec;
```

**可扩展性**：添加新模板类型：
1. 在 `INSTALL_PATHS` 中添加条目
2. 在 `index.json` 中添加新类型的模板
3. 下载逻辑无需代码更改
