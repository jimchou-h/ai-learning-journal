---
name: project-structure-audit
description: 通用项目结构与页面架构审计 Skill。用于按统一标准分析页面分层、复用率、类型健康度、风险点，并输出可执行重构计划。
license: MIT
metadata:
  author: team-custom
  version: "1.2.0"
---

# Project Structure Audit Workflow

Use this skill to run a reusable architecture audit for any project. Focus on evidence-based findings and actionable refactor plans.

## Core Goal

- 快速识别“能跑但难维护”的结构性问题。
- 输出统一格式的审计报告，减少回答风格漂移。
- 给出按优先级排序的最小改造步骤，便于分批落地。
- 审计后生成可执行的优化清单并持续更新状态。
- 在改造过程中强制对齐 `vue-best-practices` 关键规范。

## Inputs (required)

在开始前先明确以下输入信息。若用户未提供，应先询问：

- 审计范围（目录或模块），例如：`src/views/integration-bank`
- 技术栈（Vue/React/Node 等）
- 目标（提效、重构、防回归、接手项目）
- 是否有项目级配置文件（如 `project-audit.config.json`）

## Audit Procedure (must follow in order)

### 1) Scope and inventory

- 列出范围内页面、组件、composable、service、types 的主要文件。
- 标记路由入口页和核心弹窗/抽屉页。
- 识别高复杂度文件（长文件、状态变量密集、模板分支多）。

### 2) Architecture boundary check

检查是否存在边界混乱：

- 页面是否同时承担“数据编排 + 大量 UI + API 细节 + 文案提示”
- composable 是否混入 UI 行为（如直接 `message`、`modal`）
- service 是否承担了展示层逻辑
- 类型定义是否分散且重复

### 3) Reuse and abstraction check

检查复用机会和过度复制：

- 相同筛选区、表格列、分页逻辑是否跨页面重复
- 相同上传、预览、弹窗流程是否有统一封装
- 同构页面数量 >= 2 时，是否有共享 composable 或基础组件

### 4) Type safety and API contract check

- 标记 `any` / `Record<string, any>` / 隐式类型转换风险
- 标记状态魔法值（`0/1/2`）是否缺枚举
- 检查字段兼容逻辑是否分散在页面（建议汇总到 mapper/adapter）

### 5) Risk and maintainability check

- 标记硬编码阈值（如 `size: 1000`、固定默认 orgId、固定字符串规则）
- 标记可能的功能 bug（模板表达式错误、状态切换错误、分页覆盖等）
- 标记重复代码导致的“改一处漏一处”风险

### 6) Output plan

必须输出分级计划：

- `P0`：线上风险/明显 bug/数据错误
- `P1`：结构性问题（边界不清、重复度高）
- `P2`：代码味道（命名、注释、轻微冗余）

并给出“按文件”的最小改造清单，确保可以逐步提交。

### 7) Create optimization backlog file (required)

审计完成后，必须创建或更新“优化清单文件”，用于分步实施与验收。若项目有分支文档规范，优先写入对应分支文档；否则使用独立清单文件。

清单必须包含：

- `任务编号`（例如 `S1-01`）
- `问题级别`（P0/P1/P2）
- `目标文件`
- `改造目标`
- `验收标准`
- `状态`（todo/doing/done）
- `关联回归点`

### 8) Vue best-practices compliance gate (required for Vue projects)

若技术栈包含 Vue，改造计划和每一步执行都必须经过以下门禁：

- 是否遵循 Composition API 与 `script setup`（若项目未明确例外）
- 页面是否过重，是否按职责拆分为页面编排、业务 composable、展示组件
- 是否遵循 props down / events up 的数据流
- 是否减少模板复杂分支，将派生逻辑上移到 `computed` 或函数
- 是否控制 `any` 与魔法值，优先使用类型与枚举
- composable 是否避免直接承担 UI 通知职责（尽量返回结果给页面层处理）
- 组件抽离并回引后，是否已补全 `import` 与注册引用（禁止仅改模板/JSX 不补 import）

### 9) Type-check execution gate (required before final delivery)

在输出最终改造结果前，必须执行类型检查：

- 优先执行 `npm run type-check`
- 若 `package.json` 中无 `type-check` 脚本，先补充脚本（Vue 项目推荐 `vue-tsc --noEmit`），再执行
- 仅当 `type-check` 与 `lint` 均通过时，才可标记任务为可交付；否则需回到清单继续修复

## Required Output Format

每次审计必须使用以下结构：

1. `审计范围`
2. `主要发现`
   - P0
   - P1
   - P2
3. `证据`
   - 文件路径 + 关键代码片段（必要时）
4. `最小改造清单（按优先级）`
   - 第一步（当天可完成）
   - 第二步（1-2 天）
   - 第三步（长期优化）
5. `回归验证清单`
   - 功能回归点
   - 类型与 lint 检查点
   - 抽离组件 import 完整性检查点（原页面是否补全 import）
6. `优化清单文件`
   - 文件路径
   - 本次新增/更新任务数
7. `Vue 规范对齐检查（仅 Vue 项目）`
   - 已满足项
   - 待修正项

## Optional Project Config

若项目提供 `project-audit.config.json`，优先读取并使用其中规则。若无配置，使用默认策略：

- route/view 文件 > 300 行：建议评估拆分
- 同构页面 >= 2：建议抽共享 composable/组件
- composable 直接操作 UI 提示：标记边界风险
- 发现 `any`：标记类型债务
- 审计后必须落地优化清单文件
- Vue 项目默认启用 `vue-best-practices` 对齐检查

## Guardrails

- 不做无证据的泛泛建议；每条发现要有文件依据。
- 不追求一次性“大重构”；优先给出最小可交付路径。
- 优先稳定行为，再做抽象优化。
- 若用户只要结论，输出精简版；若用户要落地，输出文件级步骤。
- 审计结论必须可执行：没有优化清单视为未完成闭环。
