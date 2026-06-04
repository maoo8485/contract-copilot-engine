# DOCX 输出管线

## 执行规则

本文是 Step 4 的完整指令。**输入**调用 1-3 的全部输出，**产出**两个 DOCX 文件。

## 输出物

| 文件 | 内容 | 格式要求 |
|:---|:---|:---|
| `[合同名]_审核修订版.docx` | 合同原文 + 修订痕迹 + 批注 | OOXML track changes + comments |
| `[合同名]_审查报告.docx` | 审查意见书（8 节） | 正式法律文书版式 |

## 处理流程

### 1. 转换合同为 DOCX

```bash
pandoc [合同名].md -o [合同名].docx
```

若合同已是 DOCX 则跳过。

### 2. 生成审查计划 JSON

将调用 2 的问题清单 + 调用 3 的深度分析输出，结构化写入 `review-plan.json`。

**字段要求**：
- 每个问题必须有 `risk_name / risk_level / action / target_text`
- `action` 取值：
  - `replace`：确定性问题（笔误、术语统一、直接改文的整改）→ 直接修订
  - `comment`：需谈判/待确认/需保留弹性的问题 → 仅批注
  - `report-only`：程序优化型、说明型、低必要性问题 → 仅入报告，不在 Word 留痕
- `replacement_text`：replace 项必须提供可落文的替换文本
- `comment`：comment 项必须提供批注文本（多行结构：风险等级 / 风险点 / 条款位置 / 说明 / 修改建议 / 可选建议措辞）

### 3. 执行修订批注

```bash
CONTRACT_COPILOT_CONFIG_DIR="[可写临时目录]" \
PYTHONPATH="[contract-copilot 上级目录]" \
python3 -m contract-copilot.scripts.review.apply_review_plan \
  --input [原始合同].docx \
  --plan review-plan.json \
  --output [合同名]_审核修订版.docx \
  --report-docx [合同名]_审查报告.docx \
  --author "[审查人姓名]" \
  --organization "[机构名称]" \
  --no-archive
```

**修订策略（revise-first）**：
- 确定性问题 → 直接修订正文（显示为 track changes）
- 涉及商务谈判、事实缺口、需保留弹性的问题 → 批注
- 重大直接修订（P0/P1）→ 保留简短批注说明修改原因
- 程序优化型、说明型、低必要性问题 → 仅写入审查报告，不在 Word 正文留痕

### 4. 失败降级

以下任一条件触发降级：

- Python 脚本执行失败（ImportError / AttributeError / PermissionError）
- `config/reviewer_profile.json` 不存在且无法自动创建
- 依赖缺失（defusedxml / lxml / python-docx）
- 沙箱限制导致无法写入 `.cc-switch` 路径

**降级方案**：

1. **仍产出审查报告 DOCX**——用 pandoc 将审查报告 Markdown 转为 DOCX（不含修订痕迹，但报告本身完整可用）
2. **仍产出修订建议 Markdown**——在合同 Markdown 中直接写入修改，用 `[审查批注: 风险等级/问题/建议]` 标记需要人工处理的项
3. **明确告知阻塞点**——"DOCX 修订批注版未生成，原因：[具体错误]。修订建议已写入 Markdown 文件 [路径]，审查报告 DOCX 已生成 [路径]。"

## 审查报告结构（8 节）

1. **审查意见书抬头**——致/审查人/日期/合同概况
2. **综合审查意见**——结论（可签/有条件可签/不建议签）+ 核心风险摘要
3. **重要风险提示表**——P0/P1/P2 分级 + 签约前是否必须处理
4. **详细审查意见**——逐项表格（风险概述、原条款、建议修改、法律依据）
5. **审批触发汇总**（如有调用 3）
6. **谈判优先级**（如有调用 3）
7. **签署前先决事项**——Checklist 格式
8. **声明**——审查范围、法律依据、不构成法律意见

报告版式：深蓝标题、仿宋正文、浅底元信息卡、紧凑表格、页脚页码。

## 审查计划 JSON 模板

```json
{
  "meta": {
    "author": "[审查人]",
    "organization": "[机构]",
    "contract_type": "[从调用1路由卡]",
    "edit_policy": "revise-first"
  },
  "findings": [
    {
      "risk_name": "[名称]",
      "risk_level": "P0",
      "action": "replace",
      "target_text": "[合同中的原文]",
      "replacement_text": "[替换后的文本]",
      "comment": "P0 | [风险名称] | [条款位置]\n[修改说明]\n> 法律依据：[法条]"
    }
  ]
}
```

## 完成

Step 4 完成。产物检查清单：
- [ ] `_审核修订版.docx` 已生成（或降级为修订建议 Markdown）
- [ ] `_审查报告.docx` 已生成
- [ ] 文件路径已明确告知用户
