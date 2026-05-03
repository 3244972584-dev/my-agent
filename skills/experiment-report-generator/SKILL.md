---
name: experiment-report-generator
description: |
  根据实验指导书与数据生成完整实验报告，支持自动公式、图表生成、步骤描述等高级功能。
  当用户需要以下任一操作时触发：
  (1) 生成实验报告 (2) 写实验报告 (3) 实验报告排版 (4) 把实验数据整理成报告
  需要用户提供：实验指导书（PDF/文字）+ 实验数据（图片/Excel/手动输入）
---

# 实验报告生成器（增强版）

## 整体流程

```
实验指导书 + 实验数据
    │
    ├─ Step 1: 解析实验指导书（提取结构化信息）
    ├─ Step 2: 数据处理（调用 experiment-curve-fitting）
    ├─ Step 3: 生成图表（调用 experiment-plotter）
    └─ Step 4: 生成 Word 报告
```

### Step 1: 解析实验指导书

从指导书中提取结构化信息：

| 提取项 | 说明 |
|--------|------|
| 实验名称 | 报告标题 |
| 实验目的 | 通常指导书明确列出 |
| 实验原理 | 关键公式、定律，记录公式编号 |
| 实验器材 | 仪器名称、型号、规格 |
| 实验步骤 | 指导书要求的操作流程 |
| 数据处理方法 | 作图法、线性拟合、误差分析等 |

如果指导书是 PDF，用 `read` 工具读取内容。

### Step 2: 数据处理

根据指导书要求选择合适的方法。曲线拟合调用 `experiment-curve-fitting` skill。

处理过程要**可追溯**：每一步写明用了什么公式、代入什么数值、得到什么结果。

### Step 3: 生成图表

调用 `experiment-plotter` skill 生成实验图表，保存为图片后插入报告。

如 matplotlib 未安装：
```bash
python3 -m pip install --user --break-system-packages matplotlib
```

### Step 4: 生成 Word 报告

使用 `python-docx` 生成。如未安装：
```bash
python3 -m pip install --user --break-system-packages python-docx
```

报告结构参考 [references/report-template.md](references/report-template.md)。

**报告必须包含：**

1. **标题**（实验名称）
2. **实验目的**
3. **实验原理与公式**（带公式编号，如式(1)、式(2)）
4. **实验步骤**（体现"做了什么 → 得到什么"）
   - ❌ "测量了电压和电流"
   - ✅ "调节加速电压 Ua 从 25V 到 98V，测量对应的阳极电压 Ue'，得到 6 组 Ua-Ue' 数据对"
5. **实验数据与处理过程**（完整计算链：原始数据 → 中间计算 → 最终结果）
6. **图表和数据分析**（由 experiment-plotter 生成，插入 Word）
7. **误差分析与总结**

**格式要求：**
- 正文：宋体 12pt，行距 1.5 倍
- 标题：黑体
- 表格：Table Grid 样式（带边框）
- 公式：居中，标注编号
- 图表：居中，标注图号和标题

**保存路径：** `/mnt/c/Users/sdg/Desktop/`
