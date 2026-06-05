---
name: logic-review-universal
description: 通用代码逻辑校验与审查规范。用于代码评审、重构后检查、上线前风险扫描，聚焦逻辑正确性、状态流一致性、边界条件和回归风险。
license: MIT
metadata:
  author: team-custom
  version: "1.0.0"
---

# Logic Review Universal Workflow

Use this skill to run a reusable, framework-agnostic logic review. Prioritize correctness and regression risk over style.

## Core Goal

- 识别会导致错误行为的数据流和分支问题
- 发现“能运行但逻辑不稳”的隐患
- 输出有证据、可执行、可验收的修复清单
- 固化统一审查格式，降低回答漂移

## Inputs (required)

在开始前先明确以下输入：

- 审查范围（目录、模块、PR、提交区间）
- 业务目标（该功能应如何工作）
- 关键链路（新增、编辑、删除、审核、导入导出等）
- 风险偏好（保守/平衡/激进）

若用户未提供上述信息，先在审查结论中显式标记假设。

## Review Procedure (must follow in order)

### 1) Build execution map

- 识别入口：页面事件、接口入口、任务入口、定时任务
- 识别主路径：正常流程从输入到输出的关键步骤
- 识别副路径：异常、回滚、重试、取消、并发

### 2) Validate contract and mapping

- 校验请求参数字段名、响应字段名、类型是否一致
- 校验 DTO/VO/Model 转换是否有遗漏或错误映射
- 校验枚举/状态值是否统一，避免魔法值漂移

### 3) Validate state transitions

- 校验 loading、disabled、锁状态是否可正确收敛
- 校验状态切换是否存在“跳步、漏步、重复触发”
- 校验重置/分页/筛选/路由切换是否互相污染

### 4) Validate branch completeness

- 校验 if/else、switch、guard clause 是否覆盖所有合法输入
- 校验空值、极值、非法值、重复提交、并发触发
- 校验编辑态与新增态、首屏态与回填态的一致性

### 5) Validate side effects

- 校验副作用触发时机：请求、缓存、日志、埋点、消息提示
- 校验失败分支是否有补偿或可恢复策略
- 校验异步竞争条件（后返回覆盖先返回）

### 6) Validate consistency and reuse

- 标记复制逻辑是否已出现分叉（同逻辑多处实现）
- 标记同名字段异义、异名字段同义问题
- 标记跨层泄漏（展示层耦合底层细节）

### 7) Output actionable plan

- 只给有证据结论，不给“猜测式”结论
- 每个问题给出最小修复建议
- 每个修复建议附回归验证点

### 8) Persist regression checklist to file (required)

- 审查完成后，必须创建或更新独立的“逻辑回归清单文件”
- 若用户未指定路径，默认使用 `docs/logic-review-backlog.md`
- 每次审查都要增量更新，不覆盖历史任务（除非用户明确要求）
- 清单用于后续改造跟踪与验收闭环
- 若本步骤未执行，则本次审查视为未完成，不可交付
- 即使本次“无问题”，也要在清单文件追加一条“本次审查无新增问题”的记录（含日期与范围）

## Severity Model

- `P0`：错误数据、主流程失败、权限/资金/订单等高风险错误
- `P1`：逻辑不稳定、条件遗漏、状态不一致、易回归
- `P2`：可维护性问题、轻微冗余、命名或结构噪音

## Evidence Rule

每条发现必须包含：

- 现象（What）
- 影响（Impact）
- 触发条件（When）
- 证据（Where: 文件路径 + 关键代码）
- 最小修复建议（How）

## Required Output Format

每次审查使用以下结构：

1. 审查范围与假设
2. 主要发现
   - P0
   - P1
   - P2
3. 证据
   - 文件路径 + 片段 + 触发路径
4. 最小修复清单（按优先级）
5. 回归验证清单
   - 功能回归点
   - 数据一致性检查点
   - 类型/lint/check 检查点
6. 回归清单文件
   - 文件路径
   - 本次新增/更新任务数
7. 残余风险与待确认项

## Regression Backlog File Schema (required)

回归清单文件中的每条任务必须包含：

- `任务编号`（例如 `LR-001`）
- `问题级别`（P0/P1/P2）
- `问题摘要`
- `目标文件`
- `触发条件`
- `修复建议`
- `回归验证点`
- `状态`（todo/doing/done）
- `负责人`（可选）
- `备注`（可选）

## Guardrails

- 不做大而空重构建议；优先最小可交付修复
- 不把风格问题当作逻辑问题上升优先级
- 先修正确性，再优化抽象
- 用户只要结论时输出精简版；要落地时输出文件级步骤
- 未创建/更新回归清单文件，禁止输出“审查完成”

## Quick Checklist

- [ ] 主路径与异常路径都被验证
- [ ] 契约字段与类型全部对齐
- [ ] 状态切换无卡死、无遗漏、无覆盖污染
- [ ] 关键分支覆盖空值/极值/并发场景
- [ ] 每条结论均有证据和修复建议
- [ ] 已给出可执行的回归清单
