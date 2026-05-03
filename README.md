# 🐕 小白 - 智能实验助手

基于 OpenClaw 的智能体，专为物理实验数据处理和报告生成设计。

## Skills（8个）

### 🔁 experiment-auto-pipeline
一键完成实验数据处理全流程：识别数据图片 → 提取表格 → 数据验证 → 曲线拟合 → 生成图表 → 生成Excel+报告

### 📊 experiment-data-assistant-pro
识别实验数据图片（手写/打印），提取表格数据，生成 Excel 文件，数据分析

### 📝 experiment-report-generator
根据实验指导书自动生成完整实验报告（Word），支持公式、图表、步骤描述

### ✅ experiment-data-validator
检查实验数据质量：异常值检测、单位一致性、趋势合理性、数据完整性

### 📈 experiment-curve-fitting
曲线拟合（线性/多项式/指数/对数），返回拟合方程、参数和R²

### 🎨 experiment-plotter
根据实验数据自动生成图表（散点图、折线图、线性拟合图等）

### 💬 experiment-qa-assistant
基于实验指导书回答实验相关问题（类似助教）

### 📋 excel-xlsx
Excel 文件创建、编辑、格式化

## 智能体配置

- `IDENTITY.md` — 智能体身份（名字、性格）
- `SOUL.md` — 行为准则与红线
- `AGENTS.md` — 工作规则
- `USER.md` — 用户信息

## 技术栈

- **图像识别**: GLM-5V-Turbo / GLM-4.6V 视觉模型
- **Excel**: openpyxl
- **Word 报告**: python-docx
- **图表**: matplotlib
- **数据处理**: numpy, scipy
