---
name: contract-copilot
description: >-
  合同审查与起草助手。执行 3 次 LLM 调用 + DOCX 输出的审查管线：
  Call 1 结构扫描 → Call 2 问题识别 → Call 3 深度分析（条件触发） → DOCX 修订批注与审查报告生成。
  服务房地产各类业务合同，基于四维审核法（业务风险/法律专业性/条款一致性/业务需求还原度）执行审查。
  本文件为调度入口，详细指令在 references/ 目录下的子文件中。
homepage: https://github.com/maoo8485/contract-copilot-engine
version: "3.0.0"
license: MIT
---

# Contract Copilot（合同助手）

> 审查执行入口。详细指令在 `references/` 下的四份文件中。本文件只做调度。

## 定位

合同审查与起草。服务场景：审查既有合同（识别风险+修改建议）、起草新合同（条款骨架+推荐措辞）、DOCX 批注与修订。

## 核心理念

1. **促进交易**——审查目标不是拒绝交易，是帮助交易安全落地
2. **全面思考**——同时考虑我方、对方、履行方、第三方
3. **理性决策**——高风险给明确方案，中低风险给可选方案

## 执行流程

### Step 0：准入

**必确认**：立场（甲方/乙方/中立）、口径（克制/常规/强势）。未确认则暂停。

**记忆检查**：读取 `config/review_memory.json`。命中历史记录→沿用上次配置；未命中→确认后写入。

### Step 1：调用 1 — 结构扫描（~35k tokens）

→ 读取 [`references/call1-structure-scan.md`](references/call1-structure-scan.md)，严格按其指令执行。

**输出**：路由卡 + 宏观层发现 + 中观层发现。如有历史纠正记录，自动注入。

### Step 2：调用 2 — 问题识别（~50k tokens）

→ 读取 [`references/call2-issue-identification.md`](references/call2-issue-identification.md)，严格按其指令执行。

**输入**：调用 1 的路由结果 + 合同全文 + 匹配的 Playbook JSON

**输出**：结构化问题清单。每个问题含：风险名称 / 等级（P0/P1/P2）/ 风险后果 / 判别标准 / 推荐措辞 / 法条依据 / 整改建议 / 相关条款。

**禁止在调用 2 输出**：示范文本比对、审批触发判定、谈判策略（调用 3 的事）。

### Step 3：调用 3 — 深度分析（~50k tokens）【条件触发+手动确认】

**自动触发条件**（满足任一）：P0 ≥ 3 / 金额 ≥ 1 亿 / 上市公司 / 跨境 / 对赌

满足触发条件时，**暂停并询问用户**：

> "结构扫描+问题识别完成。共 P0 X 项 / P1 X 项。是否进入深度分析（关键合同模式检查+审批触发+谈判策略）？[是/否]"

用户确认 "是" → 读取 [`references/call3-deep-analysis.md`](references/call3-deep-analysis.md)。
用户确认 "否" 或触发条件不满足 → 跳过，直接进入 Step 4。

### Step 4：DOCX 输出

→ 读取 [`references/docx-pipeline.md`](references/docx-pipeline.md)，执行 DOCX 生成。

## 输出规范速查

| 项目 | 要求 |
|:---|:---|
| 风险等级 | P0（签前必须）/ P1（优先谈判）/ P2（可优化） |
| 审查结论 | 可签 / 有条件可签 / 不建议签 + 先决事项 |
| DOCX 模式 | 修订批注一体版（确定问题直接修订；待确认/谈判项批注；低价值建议仅入报告） |
| 报告结构 | 审查意见书 8 节（详见 docx-pipeline.md） |
| 法条引用 | `> 《民法典》第X条："原文"` |

## 文件索引（按执行顺序）

| 何时读 | 文件 | 用途 |
|:---|:---|:---|
| Step 1 | `references/call1-structure-scan.md` | 结构扫描指令 |
| Step 2 | `references/call2-issue-identification.md` | 问题识别指令 |
| Step 3 | `references/call3-deep-analysis.md` | 深度分析指令 |
| Step 4 | `references/docx-pipeline.md` | DOCX 处理和报告生成 |
| Call 1 子步骤 | `references/review-framework.md` | 宏观/中观/微观检查框架 |
| Call 1 子步骤 | `references/contract-routing.md` | 12 类路由 + Playbook 映射 |
| Call 2 子步骤 | `config/playbooks/*.json` | 条款级比对标准（按需加载） |

## 冲突解决

| 冲突领域 | 以谁为准 |
|:---|:---|
| 执行流程 | 本文件 |
| 领域知识与审核标准 | 项目 CLAUDE.md |
| 条款标准立场与风险阈值 | Playbook JSON |
| 审批层级与 Tier 分级 | Playbook JSON |
| 反馈纠错记录 | Playbook JSON feedbackCorrections.records[] |
| 报告输出格式 | 本文件 + docx-pipeline.md |

## 版本

当前：3.0.0（结构重组：入口文件瘦身、3-Call 拆分为独立 reference、增加调用 3 手动确认、增加 DOCX 失败降级） / 更新：2026-06-03
