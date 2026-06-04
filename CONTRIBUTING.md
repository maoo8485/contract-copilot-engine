# 贡献指南

## 如何贡献

1. **Issue**：提交缺陷报告、功能建议、文档改进
2. **Pull Request**：代码改动、新功能、缺陷修复

## 开发环境

```bash
git clone https://github.com/YOUR_USERNAME/contract-copilot-engine.git
cd contract-copilot-engine
python3 -m pip install -r scripts/requirements.txt
```

## 贡献范围

本仓库仅包含**审查引擎框架**。贡献方向：

- 管线架构改进
- DOCX 生成和修订工具优化
- 合同类型路由和匹配逻辑
- Playbook JSON Schema 设计
- 文档和示例

**不在范围内的贡献**：行业特定的条款标准、风险阈值或审查准则。这些属于 Playbook JSON 的私有定制内容，不进入本仓库。

## Pull Request 流程

1. Fork 本仓库并创建功能分支
2. 完成修改
3. 如有必要，更新文档
4. 提交 PR 并附清晰说明

## 致谢

本项目灵感来源于 [cat-xierluo/legal-skills](https://github.com/cat-xierluo/legal-skills) 项目中的 [`contract-copilot`](https://github.com/cat-xierluo/legal-skills/tree/main/skills/contract-copilot) skill（作者：[杨卫薪律师](https://github.com/cat-xierluo)，协议 CC BY-NC-SA 4.0）。该项目提出的合同审查三层扫描框架和 Word 修订批注一体化输出思路，对本引擎的管线架构设计产生了重要影响。

本引擎为独立重实现（MIT 协议），非引用项目的衍生作品。致谢杨卫薪律师的开源贡献。
