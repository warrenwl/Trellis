# 平台集成指南

如何添加对新 AI CLI 平台的支持（如 Claude Code、Cursor、Gemini CLI、OpenCode、iFlow、Codex、Kilo、Kiro、Qoder）。

---

## 架构

平台支持使用**集中注册表模式**（类似于 Turborepo 的包管理器支持）：

- **数据注册表**：`src/types/ai-tools.ts` — 包含所有平台元数据的 `AI_TOOLS` 记录
- **函数注册表**：`src/configurators/index.ts` — 每个平台的 `PLATFORM_FUNCTIONS` 含 configure/collectTemplates
- **共享工具**：`src/configurators/shared.ts` — init 和 update 路径都使用的 `resolvePlaceholders()`
- **共享工具**：`src/utils/compare-versions.ts` — 支持完整预发布的 `compareVersions()`（由 cli、update、migrations 使用）
- **派生辅助函数**：`ALL_MANAGED_DIRS`、`getConfiguredPlatforms()` 等 — 由 update、init、哈希追踪使用

所有列表（备份目录、模板目录、平台检测、清理白名单）都**自动从注册表派生**。没有分散的硬编码列表。

---

## 检查清单：添加新平台

添加新平台 `{platform}` 时，更新以下内容：

### 第 1 步：类型定义（数据注册表）

| 文件 | 更改 |
|------|--------|
| `src/types/ai-tools.ts` | 添加到 `AITool` 联合类型 |
| `src/types/ai-tools.ts` | 添加到 `CliFlag` 联合类型 |
| `src/types/ai-tools.ts` | 添加到 `TemplateDir` 联合类型 |
| `src/types/ai-tools.ts` | 在 `AI_TOOLS` 记录中添加条目（name、configDir、cliFlag、defaultChecked、hasPythonHooks、templateDirs）|

**此单一条目自动传播到**：`BACKUP_DIRS`、`TEMPLATE_DIRS`、`getConfiguredPlatforms()`、`cleanupEmptyDirs()`、`initializeHashes()`、init `TOOLS[]` 提示、Windows 检测。

### 第 2 步：CLI 标志

| 文件 | 更改 |
|------|--------|
| `src/cli/index.ts` | 添加 `--{platform}` 选项 |
| `src/commands/init.ts` | 在 `InitOptions` 接口添加 `{platform}?: boolean` |

> 注意：Commander.js 选项和 TypeScript 接口需要静态声明 — 无法从注册表派生。`init.ts` 中的编译时断言 `_AssertCliFlagsInOptions` 将捕获缺失的 `InitOptions` 字段 — 如果 `CliFlag` 有不在 `InitOptions` 中的值，您将收到构建错误。

### 第 3 步：配置器（函数注册表）

| 文件 | 更改 |
|------|--------|
| `src/configurators/{platform}.ts` | 创建新配置器（从现有复制，导出 `configure{Platform}`）|
| `src/configurators/index.ts` | 在 `PLATFORM_FUNCTIONS` 中添加条目，含 `configure` 和可选的 `collectTemplates` |

### 第 4 步：模板

**标准模式**（Python 钩子 — 如 Claude、iFlow）：

| 目录 | 内容 |
|-----------|----------|
| `src/templates/{platform}/` | 根目录 |
| `src/templates/{platform}/index.ts` | 导出命令、代理、钩子、设置的函数 |
| `src/templates/{platform}/commands/trellis/` | 斜杠命令（`.md` 文件）|
| `src/templates/{platform}/agents/` | 代理定义（`.md` 文件）|
| `src/templates/{platform}/hooks/` | 钩子脚本（`.py` 文件）|
| `src/templates/{platform}/settings.json` | 平台设置（可使用 `{{PYTHON_CMD}}` 占位符）|

**JS 插件模式**（如 OpenCode）：

| 目录 | 内容 |
|-----------|----------|
| `src/templates/{platform}/` | 根目录 |
| `src/templates/{platform}/commands/trellis/` | 斜杠命令（`.md` 文件）|
| `src/templates/{platform}/plugin/` | JS 插件文件 |
| `src/templates/{platform}/lib/` | JS 库文件 |
| `src/templates/{platform}/package.json` | 插件依赖 |

> 注意：OpenCode 使用 JS 插件而非 Python 钩子，没有 `index.ts` 模板模块，也没有 `collectTemplates` — 因此 `trellis update` 不跟踪 OpenCode 模板文件。如果新平台使用 JS 插件，遵循此模式。

**技能模式**（Codex、Kiro、Qoder）：

| 目录 | 内容 |
|-----------|----------|
| `src/templates/{platform}/` | 根目录 |
| `src/templates/{platform}/index.ts` | 导出列出技能的函数 |
| `src/templates/{platform}/skills/<skill-name>/SKILL.md` | 技能定义 |

> 注意：Codex/Kiro/Qoder 使用技能（而非斜杠命令）。技能内容应使用 `$<skill-name>` / `/skills` 语义，而非 `/trellis:*` 语法。Qoder 技能在每个 SKILL.md 顶部使用 YAML frontmatter（`---\nname: ...\n---`）。

**仅命令模式**（Cursor）：

| 目录 | 内容 |
|-----------|----------|
| `src/templates/{platform}/` | 根目录 |
| `src/templates/{platform}/index.ts` | 导出 `getAllCommands(): CommandTemplate[]` |
| `src/templates/{platform}/commands/` | 斜杠命令（`.md` 文件）|

> 注意：Cursor 使用平面前缀命名（`trellis-start.md` → `/trellis-start`）。无钩子，无代理，无设置。

**工作流模式**（Kilo）：

| 目录 | 内容 |
|-----------|----------|
| `src/templates/{platform}/` | 根目录 |
| `src/templates/{platform}/index.ts` | 导出 `getAllWorkflows(): WorkflowTemplate[]` |
| `src/templates/{platform}/workflows/` | 工作流文件（`.md` 文件）|

> 注意：Kilo 使用平面工作流目录（`workflows/start.md` → `/start`）。无钩子，无代理，无设置。

**TOML 命令模式**（Gemini CLI）：

| 目录 | 内容 |
|-----------|----------|
| `src/templates/{platform}/` | 根目录 |
| `src/templates/{platform}/index.ts` | 导出 `getAllCommands(): CommandTemplate[]`（过滤 `.toml` 而非 `.md`）|
| `src/templates/{platform}/commands/trellis/` | 斜杠命令（`.toml` 文件）|

> 注意：Gemini CLI 是第一个使用 TOML 而非 Markdown 的命令平台。TOML 格式：`description = "..."` + `prompt = """..."""`。子目录命名空间与 Claude 相同（`commands/trellis/start.toml` → `/trellis:start`）。创建 TOML 模板时，对多行提示使用三引号字符串（`"""`）。

**工作流模式**（Antigravity）：

| 目录 | 内容 |
|-----------|----------|
| `src/templates/{platform}/` | 根目录 |
| `src/templates/{platform}/index.ts` | 导出 `getAllWorkflows(): WorkflowTemplate[]` |

> 注意：Antigravity 没有物理模板文件 — 工作流内容在运行时**从 Codex 技能派生**，通过 `adaptSkillContentToWorkflow()`。配置目录是 `.agent/workflows`（而非 `.agent/`）。工作流通过 `/workflow-name` 斜杠命令触发。添加新 Codex 技能时，Antigravity 自动拾取。

**必需命令/技能**：所有平台必须包含以下内容（适应每个平台的格式）：

| 命令 | 目的 | 必需 |
|---------|---------|----------|
| `start` | 会话初始化 | 是 |
| `finish-work` | 提交前检查清单 | 是 |
| `brainstorm` | 需求发现 | 是 |
| `break-loop` | 调试后分析 | 是 |
| `record-session` | 会话录制 | 是 |
| `before-backend-dev` | 阅读后端指南 | 是 |
| `before-frontend-dev` | 阅读前端指南 | 是 |
| `check-backend` | 检查后端代码质量 | 是 |
| `check-frontend` | 检查前端代码质量 | 是 |
| `check-cross-layer` | 跨层验证 | 是 |
| `create-command` | 创建新斜杠命令 | 是 |
| `integrate-skill` | 集成外部技能 | 是 |
| `onboard` | 团队入职指南 | 是 |
| `update-spec` | 更新代码规范文档 | 是 |
| `parallel` | 多代理并行工作 | 可选（平台能力）|
| `migrate-specs` | 迁移规范版本 | 可选 |

> **规则**：当新命令添加到任何平台时，必须添加到所有平台。检查 `src/templates/claude/commands/trellis/` 作为参考列表。

### 第 5 步：模板提取

| 文件 | 更改 |
|------|--------|
| `src/templates/extract.ts` | 添加 `get{Platform}TemplatePath()` 函数 + `get{Platform}SourcePath()` 废弃别名 |

### 第 6 步：Python 脚本（独立运行时）

> **警告**：`cli_adapter.py` 使用 if/elif/else 链**无穷尽检查**。新平台静默进入 `else` 分支（Claude 默认）。您必须为下面列出的**每个方法**添加显式分支。

| 文件 | 更改 |
|------|--------|
| `src/templates/trellis/scripts/common/cli_adapter.py` | 添加到 `Platform` 字面类型、`config_dir_name` 属性、`detect_platform()`、`get_cli_adapter()` 验证 |

**需要显式分支的 cli_adapter.py 方法**（不要依赖 `else` 穿透）：

| 方法 | 决定什么 | 示例 |
|--------|---------------|---------|
| `config_dir_name` | 配置目录名称 | `".gemini"`、`.agent"` |
| `get_trellis_command_path()` | 命令文件路径格式 | `.toml` vs `.md`，子目录 vs 平面 |
| `get_non_interactive_env()` | 非交互环境变量 | `{}`（如果没有）或平台特定 |
| `build_run_command()` | 运行代理的 CLI 命令 | `["gemini", prompt]` 或抛出 ValueError |
| `build_resume_command()` | 恢复会话的 CLI 命令 | `["gemini", "--resume", id]` 或抛出 ValueError |
| `cli_name` | CLI 可执行文件名 | `"gemini"`、`"agy"` |
| `detect_platform()` | 目录检测逻辑 | 检查 `.gemini/` 存在 |
| `get_commands_path()` | 命令目录结构 | `commands/trellis/` 或 `workflows/` |
| `src/templates/trellis/scripts/common/registry.py` | 需要时更新默认平台 |
| `src/templates/trellis/scripts/multi_agent/plan.py` | 仅当 `build_run_command()` 返回有效命令（不是 `raise ValueError`）时添加到 `--platform` 选项 |
| `src/templates/trellis/scripts/multi_agent/start.py` | 仅当 `build_run_command()` 返回有效命令（不是 `raise ValueError`）时添加到 `--platform` 选项 |
| `src/templates/trellis/scripts/multi_agent/status.py` | 如需要添加平台特定行为（仅当支持时）|

> 注意：Python 脚本在用户项目运行时执行 — 它们无法从 TS 注册表导入，在 `cli_adapter.py` 中维护自己的注册表。

> 当前 Codex 集成范围：公共脚本 + `task.py` 上下文路径映射。多代理运行时（`multi_agent/*.py`）故意不在范围内。

### 第 7 步：文档

| 文件 | 更改 |
|------|--------|
| `README.md` | 将平台添加到支持工具列表 |
| `README_CN.md` | 将平台添加到支持工具列表（中文）|

### 第 8 步：构建脚本

| 文件 | 更改 |
|------|--------|
| `scripts/copy-templates.js` | 无需更改（复制整个 `src/templates/` 目录）|

### 第 9 步：项目配置（可选）

如果 Trellis 项目本身应支持新平台：

| 目录 | 内容 |
|-----------|----------|
| `.{platform}/` | 项目自己的配置目录 |
| `.{platform}/commands/trellis/` | 斜杠命令 |
| `.{platform}/agents/` | 代理 |
| `.{platform}/hooks/` | 钩子 |
| `.{platform}/settings.json` | 设置 |

### 第 10 步：Gitignore

| 文件 | 更改 |
|------|--------|
| `.gitignore` | 添加本地配置模式（如 `{platform}.local.json`）|

### 第 11 步：测试（必需）

> **警告**：动态迭代测试（如 `PLATFORM_IDS.forEach`）仅验证注册表元数据。它们不覆盖平台特定的运行时行为。您必须添加显式测试。

| 测试文件 | 添加什么 |
|-----------|-------------|
| `test/templates/{platform}.test.ts` | **新文件**：验证 `getAllCommands()`/`getAllSkills()`/`getAllWorkflows()` 返回预期集合，内容非空，格式有效 |
| `test/configurators/platforms.test.ts` | 检测测试：`getConfiguredPlatforms` 找到 `.{configDir}`。配置器测试：`configurePlatform` 写入预期文件，无编译产物 |
| `test/commands/init.integration.test.ts` | Init 测试：`init({ {platform}: true })` 创建正确目录。负面断言：在现有平台测试中添加 `.{configDir}` 检查 |
| `test/templates/extract.test.ts` | `get{Platform}TemplatePath()` 返回现有目录。`get{Platform}SourcePath()` 废弃别名等于模板路径 |
| `test/regression.test.ts` | 平台注册：`AI_TOOLS.{platform}` 存在且 `configDir` 正确。cli_adapter：`commonCliAdapter` 包含 `"{platform}"` 和 `".{configDir}"`。如果定义了 `collectTemplates`，更新 `withTracking` 列表 |

---

## 您不需要更新的内容

这些现在**自动从注册表派生**：

| 之前硬编码 | 现在派生自 |
|---------------------|------------------|
| `update.ts` 中的 `BACKUP_DIRS` | `configurators/index.ts` 中的 `ALL_MANAGED_DIRS` |
| `template-hash.ts` 中的 `TEMPLATE_DIRS` | `configurators/index.ts` 中的 `ALL_MANAGED_DIRS` |
| `update.ts` 中的 `getConfiguredPlatforms()` | `configurators/index.ts` 中的 `getConfiguredPlatforms()` |
| `update.ts` 中的 `cleanupEmptyDirs()` 白名单 | `configurators/index.ts` 中的 `isManagedPath()` / `isManagedRootDir()` |
| `update.ts` 中的 `collectTemplateFiles()` if/else | `collectPlatformTemplates()` 分发循环 |
| `init.ts` 中的 `TOOLS[]` | `configurators/index.ts` 中的 `getInitToolChoices()` |
| `init.ts` 中的配置器分发 | `configurators/index.ts` 中的 `configurePlatform()` |
| `init.ts` 中的 Windows Python 检测 | `configurators/index.ts` 中的 `getPlatformsWithPythonHooks()` |

---

## 按平台的命令格式

| 平台 | 命令格式 | 文件格式 | 示例 |
|----------|---------------|-------------|---------|
| Claude Code | `/trellis:xxx` | Markdown (`.md`) | `/trellis:start` |
| Cursor | `/trellis-xxx` | Markdown (`.md`) | `/trellis-start` |
| OpenCode | `/trellis:xxx` | Markdown (`.md`) | `/trellis:start` |
| iFlow | `/trellis:xxx` | Markdown (`.md`) | `/trellis:start` |
| Gemini CLI | `/trellis:xxx` | TOML (`.toml`) | `/trellis:start` |
| Kilo | `/<workflow-name>` | Markdown (`.md`) | `/start` |
| Codex | `$<skill-name>` / `/skills` | Markdown (`SKILL.md`) | `$start` |
| Kiro | `$<skill-name>` / `/skills` | Markdown (`SKILL.md`) | `$start` |
| Qoder | `$<skill-name>` / `/skills` | Markdown (`SKILL.md`) | `$start` |
| Antigravity | `/<workflow-name>` | Markdown (`.md`) | `/start` |

创建平台模板时，确保引用匹配平台的交互格式和文件格式。

---

## Windows 编码修复

所有输出到 stdout 的钩子脚本必须包含 Windows 编码修复：

```python
# IMPORTANT: Force stdout to use UTF-8 on Windows
# This fixes UnicodeEncodeError when outputting non-ASCII characters
if sys.platform == "win32":
    import io as _io
    if hasattr(sys.stdout, "reconfigure"):
        sys.stdout.reconfigure(encoding="utf-8", errors="replace")  # type: ignore[union-attr]
    elif hasattr(sys.stdout, "detach"):
        sys.stdout = _io.TextIOWrapper(sys.stdout.detach(), encoding="utf-8", errors="replace")  # type: ignore[union-attr]
```

---

## 常见错误

### 忘记添加到 PLATFORM_FUNCTIONS

**症状**：`trellis init` 配置了平台，但 `trellis update` 不跟踪其模板文件。

**修复**：在 `src/configurators/index.ts` 的 `PLATFORM_FUNCTIONS` 中添加带 `collectTemplates` 函数的条目。

### cli_adapter.py 中缺少平台

**症状**：Python 脚本失败并显示"Unsupported platform"错误。

**修复**：在 `cli_adapter.py` 中的 `Platform` 字面类型、`config_dir_name` 属性和 `get_cli_adapter()` 验证中添加平台。

### 模板中命令格式错误

**症状**：斜杠命令不工作或显示错误格式。

**修复**：检查平台的命令格式并更新模板中的所有命令引用。

### Codex 模板从项目 `.agents/skills` 而非 `src/templates` 复制

**症状**：生成的模板意外包含特定于仓库的自定义，并与模板真实来源漂移。

**修复**：始终使用 `src/templates/{platform}/...` 作为 `init/update` 的源模板。不要从项目运行时目录复制。

### Codex 技能目录存在但缺少 `SKILL.md`

**症状**：扫描技能时模板加载失败并显示 `ENOENT`。

**修复**：保持 `src/templates/codex/skills/<skill-name>/SKILL.md` 完整；移除技能时，同时删除 `SKILL.md` 和目录。

### 配置器中 EXCLUDE_PATTERNS 缺少 `.js`

**症状**：在生产构建（`dist/`）中，`trellis init` 将编译的 `index.js`（以及 `.js.map`、`.d.ts`）复制到用户配置目录（如 `.gemini/index.js`）。

**原因**：配置器的 `EXCLUDE_PATTERNS` 不过滤 `.js` 文件。在开发（`src/`）中，仅存在 `.ts` 文件，所以问题不可见。在生产中，`tsc` 将 `index.ts` 编译为 `index.js` 到 `dist/templates/{platform}/`，`copyDirFiltered` 复制它。

**修复**：确保 `EXCLUDE_PATTERNS` 包含 `.js`、`.js.map`、`.d.ts`、`.d.ts.map` — 匹配 Cursor 配置器模式。Claude 配置器正确排除这些；从那里复制。

**预防**：创建新配置器时，从现有配置器（如 `cursor.ts`）复制完整的 `EXCLUDE_PATTERNS`，不要从头编写。

### 缺少 CLI 标志或 InitOptions 字段

**症状**：`trellis init --{platform}` 不工作。

**修复**：在 `src/cli/index.ts` 中添加 `--{platform}` 选项，在 `src/commands/init.ts` 的 `InitOptions` 中添加 `{platform}?: boolean`。这些是静态声明，无法从注册表派生。

### collectTemplates 中模板占位符未解析

**症状**：`trellis update` 每次运行都自动更新平台文件，即使什么都没更改。更新摘要显示钩子/设置"已更改"。

**原因**：`configurePlatform()` 在 init 时写入文件时将 `{{PYTHON_CMD}}` 解析为 `python3`/`python`，但 `collectPlatformTemplates()` 返回带未解析 `{{PYTHON_CMD}}` 的原始模板。哈希比较认为它们不同。

**修复**：在 `PLATFORM_FUNCTIONS` 的 `collectTemplates` lambda 中应用 `configurators/shared.ts` 中的 `resolvePlaceholders()`。添加到模板的任何新占位符必须在 `configure()` 和 `collectTemplates()` 中**都**解析。

### 更新中列出的模板但 init 未创建

**症状**：`trellis update` 始终检测到"新文件"要添加，即使在同版本新初始化的项目上。

**原因**：`update.ts` 中的 `collectTemplateFiles()` 列出了 init 中 `createSpecTemplates()` / `createWorkflowStructure()` 从未创建的文件。两个模板列表不同步。

**修复**：确保 `collectTemplateFiles()` 中列出的每个文件在 `init` 期间实际创建。如果文件是项目特定的（不是用户模板），不要将其包含在更新模板列表中。

### 项目类型条件内容未在 init 或 update 中门控

**症状**：纯后端项目在 `trellis init` 后获得空前端规范模板。用户删除不需要的 `spec/frontend/` 目录后，`trellis update` 重新创建它。

**原因（init）**：`workflow.ts` 中的 `createSpecTemplates()` 接收了 `projectType` 但忽略了它（参数名 `_projectType`）。所有项目类型都获得后端和前端规范目录。

**原因（update）**：`update.ts` 中的 `collectTemplateFiles()` 无条件在模板映射中包含所有 13 个后端 + 前端规范文件，而不检查 `spec/backend/` 或 `spec/frontend/` 是否实际存在于磁盘上。

**修复（init）**：使用 `projectType` 条件创建规范目录：
- `"backend"` → 仅指南 + 后端
- `"frontend"` → 仅指南 + 前端
- `"fullstack"` / `"unknown"` → 指南 + 两者

**修复（update）**：用 `fs.existsSync()` 检查包装后端/前端规范文件块（与 `getConfiguredPlatforms()` 检查平台目录相同的模式）。

**规则**：当 init 根据项目类型条件创建内容时，update 必须在将文件包含到模板映射中之前检查目录存在性。两条路径必须一致。

### iFlow getAllCommands() 读取错误的目录级别（已修复）

**症状**：`trellis update` 跟踪零 iFlow 命令 — init 时命令正确复制，但未跟踪更新差异。

**原因**：iFlow `getAllCommands()` 调用 `listFiles("commands")` 返回 `["trellis"]`（一个目录，不是 `.md` 文件）。修复为读取 `listFiles("commands/trellis")`。

**状态**：已修复 — `getAllCommands()` 现在从正确的子目录读取。

### PRD 假设平台能力未经研究

**症状**：实现构建了错误的抽象（例如，命令而非技能，或反之）。发现后需要重大返工。

**原因**：PRD 基于对平台工作方式的假设编写（例如，"Trae 像 Kilo 一样使用命令"），而未验证官方文档或 GitHub 仓库。

**修复**：为新平台编写 PRD 之前，研究平台的实际扩展机制：
- 检查官方文档了解支持的格式（技能、命令、规则、工作流）
- 检查平台 GitHub 仓库了解目录结构约定
- 验证用户如何调用扩展（斜杠命令、AI自动匹配、手动提及）

**预防**：在 PRD 定稿前添加"研究"步骤。PRD 应引用平台能力声明的来源。

### 将仅 IDE 平台添加到多代理 --platform 选项

**症状**：`python3 plan.py --platform {platform}` 接受值但在 `build_run_command()` 失败，因为平台没有 CLI 可执行文件。

**原因**：平台被添加到 `plan.py`/`start.py` 的 `--platform` 选项，而未检查是否支持 CLI 代理。仅 IDE 平台（如 Trae、Codex、Kiro）无法运行无头 CLI 代理。

**修复**：仅当平台有 `supports_cli_agents: true`（即 `build_run_command` 不抛出 `ValueError`）时才将平台添加到 `plan.py`/`start.py` 的 `--platform` 选项。

**规则**：多代理脚本中的 `--platform` 选项应与 `build_run_command()` 返回有效命令的平台匹配，而非注册表中的所有平台。

### 更新命令内容但忘记其他平台

**症状**：在 Claude 模板中更新 `record-session.md` 后，其他平台（iFlow、Kilo、OpenCode、Gemini）仍使用旧内容（如缺少 `--mode record` 标志、过时的命令参考表）。

**原因**：相同内容的命令模板存在于多个平台。更新一个平台的版本而未同步其他平台使它们不一致。

**修复**：修改任何命令模板内容后，检查具有相同命令的**所有**平台：

```bash
# 找到具有此命令的所有平台
find src/templates/*/commands/trellis/ -name "record-session.*"
```

**关键区别**：
- "向所有平台添加新命令" → 由上面的必需命令表覆盖
- "跨平台更新现有命令内容" → 这个错误。内容更改（新标志、重写步骤、更新参考表）必须传播到每个平台的副本。

**注意**：Gemini 使用 `.toml` 格式 — 内容必须适应（三引号字符串、`\\` 行连续）。其他所有平台使用 `.md`。

### 复制模板中的过时平台引用

**症状**：Qoder 技能引用"Claude Code"语法或 Kiro 特定调用模式。

**原因**：从现有平台复制创建模板时，平台特定引用（命令语法、平台名称、调用指令）未更新。

**修复**：复制模板后，搜索并替换所有对源平台的引用。检查：
- 平台名称提及（如"Claude Code"、"Kiro"）
- 命令调用语法（如 `/trellis:xxx` vs `$skill-name`）
- 配置目录引用（如 `.claude/` vs `.qoder/`）

---

### iFlow collectTemplates 路径漂移

**症状**：`trellis update` 在 `.iflow/commands/{name}.md`（平面）创建 iFlow 命令，而非 `.iflow/commands/trellis/{name}.md`（正确）。

**原因**：当命令从平面迁移到 `trellis/` 子目录时，`configure()` 使用递归目录复制（自动正确），但 `collectTemplates()` 使用 `files.set()` 手动构造路径（需要手动更新）。iFlow 的 `collectTemplates` 未更新 — 它生成 `.iflow/commands/${cmd.name}.md` 而非 `.iflow/commands/trellis/${cmd.name}.md`。

**修复**：在 iFlow 的 `collectTemplates`（`configurators/index.ts` 第 111 行）的路径中添加 `trellis/`。

**设计洞察**：`configure()` 和 `collectTemplates()` 使用不对称机制产生相同的文件集 — 一个递归复制目录树，另一个手动列出 `files.set()` 调用。这种不对称使得结构迁移期间路径漂移成为可能。迁移目录结构时，始终检查两条路径。

**回归测试**：`regression.test.ts` 现在验证所有带命令的平台在其 `collectTemplates` 路径中使用 `/commands/trellis/`。

---

## 参考 PR

| PR | 平台 | 模式 | 备注 |
|----|----------|---------|-------|
| #22 | iFlow CLI | 标准（钩子 + 代理）| 完整平台带 Python 钩子 |
| feat/gemini branch | Gemini CLI | TOML 命令仅 | 第一个非 Markdown 命令格式，Cursor 级最小 |
| main | Antigravity | 工作流（从 Codex 派生）| 无物理模板 — 从 Codex 技能运行时适配 |
| #71 | Qoder | 技能（像 Codex/Kiro）| 带 YAML frontmatter 的技能；Trae 被放弃（仅 IDE，无确定性调用触发）|
