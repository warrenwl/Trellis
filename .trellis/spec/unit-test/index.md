# 单元测试指南

> 本项目的测试约定和模式。

---

## 概述

本项目使用 **Vitest** 和 TypeScript ESM。测试集中在 `test/` 目录中，镜像 `src/` 结构。目标是快速、可重现的测试，最小化 mock。

---

## 指南目录

| 指南 | 描述 | 状态 |
|-------|-------------|--------|
| [约定](./conventions.md) | 文件命名、结构、断言模式、何时编写测试 | 已完成 |
| [Mock 策略](./mock-strategies.md) | mock 什么、如何 mock 以及最小化 mock 原则 | 已完成 |
| [集成模式](./integration-patterns.md) | 命令的函数级集成测试 | 已完成 |

---

## 快速参考

```bash
# 运行所有测试
pnpm test

# 监听模式
pnpm test:watch

# 运行特定测试文件
pnpm test test/commands/init.integration.test.ts

# 带覆盖率报告运行（终端 + HTML）
pnpm test:coverage
```

---

## 代码覆盖率

覆盖率通过 `@vitest/coverage-v8` 自动生成。配置在 `vitest.config.ts` 中。

- **终端**：`pnpm test:coverage` 打印每个文件的覆盖率表
- **HTML 报告**：`./coverage/index.html`（gitignored，按需生成）
- **源范围**：`src/**/*.ts`（排除 `src/cli/index.ts`）

**不要**维护手动覆盖率表 — 始终运行 `pnpm test:coverage` 获取真实数字。

---

## CI / 管道策略

| 阶段 | 运行什么 | 原因 |
|-------|---------|-----------|
| **pre-commit** (husky) | `lint-staged` (eslint + prettier) | 保持快速；不要在这里添加测试，否则开发者会用 `--no-verify` 跳过 |
| **CI** (GitHub Actions, PR gate) | `pnpm lint` → `pnpm build` → `pnpm test` | 完整套件；~312 个测试在 ~1s 内运行，没有理由拆分 |

**何时重新考虑**：如果总测试时间超过 5 分钟，拆分为快速（单元）和慢速（集成）阶段。目前不必要。

---

**语言**：所有文档应使用**中文**编写。
