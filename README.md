# 🐕 小白 - 智能实验助手

基于 OpenClaw 的智能体，专为物理实验数据处理和报告生成设计。

## 功能

### Skill 1: 实验数据提取 (`experiment-data-assistant-pro`)
- 识别实验数据图片（手写/打印）
- 提取表格数据
- 生成 Excel 文件
- 数据分析

### Skill 2: 实验报告生成 (`experiment-report-generator`)
- 根据实验指导书自动生成完整实验报告（Word）
- 支持自动公式插入
- 支持图表生成（matplotlib）
- 实验步骤描述（"做了什么 → 得到什么"）
- 误差分析与总结

### Skill 3: Hello World (`hello-world`)
- 入门示例 skill

## 智能体配置

- `IDENTITY.md` — 智能体身份（名字、性格）
- `SOUL.md` — 行为准则与红线
- `AGENTS.md` — 工作规则
- `USER.md` — 用户信息

## 使用的技术

- **OCR**: GLM-5V-Turbo / GLM-4.6V 视觉模型识别图片
- **Excel**: openpyxl
- **Word 报告**: python-docx
- **图表**: matplotlib
- **数据处理**: numpy, scipy

## 环境

- OpenClaw 智能体框架
- Python 3.12
- WSL2 (Ubuntu)
