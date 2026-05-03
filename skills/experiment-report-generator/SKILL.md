---
name: experiment-report-generator
description: |
  根据实验指导书与数据生成完整实验报告，支持自动公式、图表生成、步骤描述等高级功能。
  当用户需要以下任一操作时触发：
  (1) 生成实验报告 (2) 写实验报告 (3) 实验报告排版 (4) 把实验数据整理成报告
  需要用户提供：实验指导书（PDF/文字）+ 实验数据（图片/Excel/手动输入）
---

# 实验报告生成器（增强版）

## 触发条件

用户提供实验指导书和实验数据，要求生成实验报告。

## 整体流程

```
实验指导书 + 实验数据
    │
    ├─ Step 1: 解析实验指导书
    ├─ Step 2: 数据处理
    ├─ Step 3: 生成公式（LaTeX）
    ├─ Step 4: 生成图表
    └─ Step 5: 生成 Word 报告
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
| 注意事项 | 影响实验结论的关键点 |

如果指导书是 PDF，用 `read` 工具读取内容。

### Step 2: 数据处理

根据指导书要求选择合适的方法。参考 [references/data-processing.md](references/data-processing.md)。

**关键要求：处理过程可追溯**
- 列出所用公式（标注公式编号）
- 展示代入数值
- 给出计算结果
- 必要时附中间结果表格

### Step 3: 生成公式

在报告中插入数学公式，使用 python-docx 的 OMML（Office MathML）方式：

```python
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

def add_latex_equation(doc, latex_str, equation_num=""):
    """将 LaTeX 公式插入 Word 文档"""
    # 常用公式映射（LaTeX → Word 文本描述）
    # 复杂公式用 python-docx 的 mathML 实现
    # 简单公式用斜体居中文本
    pass
```

**常用物理公式 LaTeX 对照：**

| 含义 | LaTeX | Word 描述 |
|------|-------|-----------|
| 逸出功 | `e_0\varphi` | e₀φ |
| 里查孙公式 | `I_e = AST^2 e^{-e_0\varphi/kT}` | Ie = AS·T²·exp(-e₀φ/kT) |
| 对数关系 | `\lg(I_e/T^2) = \lg(AS) - \frac{e_0\varphi}{2.303k} \cdot \frac{1}{T}` | lg(Ie/T²) = lg(AS) - (e₀φ/2.303k)·(1/T) |

对于 Word 兼容性，优先使用**文字+符号描述**方式呈现公式（如上表右列），同时标注公式编号。
如用户需要 LaTeX 源码，可单独提供 .tex 文件。

### Step 4: 生成图表

使用 matplotlib 生成图表，保存为图片后插入 Word 报告。

```python
import matplotlib
matplotlib.use('Agg')  # 无图形界面模式
import matplotlib.pyplot as plt

plt.rcParams['font.sans-serif'] = ['SimHei', 'WenQuanYi Micro Hei']  # 中文支持
plt.rcParams['axes.unicode_minus'] = False

def plot_and_save(x, y, xlabel, ylabel, title, filename, fit_line=None):
    """生成实验图表并保存为图片"""
    fig, ax = plt.subplots(figsize=(8, 6))
    ax.scatter(x, y, color='blue', zorder=5, label='实验数据')
    if fit_line:
        ax.plot(fit_line[0], fit_line[1], 'r-', label='线性拟合')
    ax.set_xlabel(xlabel, fontsize=12)
    ax.set_ylabel(ylabel, fontsize=12)
    ax.set_title(title, fontsize=14)
    ax.legend()
    ax.grid(True, alpha=0.3)
    fig.savefig(filename, dpi=150, bbox_inches='tight')
    plt.close()
    return filename
```

**常见实验图表：**

| 图表类型 | 用途 | 示例 |
|---------|------|------|
| lg Ie' vs √Ua | 肖特基修正外推 | 逸出功实验 |
| lg(Ie/T²) vs 1/T | 里查孙直线 | 逸出功实验 |
| U-I 曲线 | 电阻/伏安特性 | 电学实验 |
| T² vs L | 单摆测重力加速度 | 力学实验 |

插入图表到 Word：
```python
from docx.shared import Inches
doc.add_picture(filename, width=Inches(5))
last_paragraph = doc.paragraphs[-1]
last_paragraph.alignment = WD_ALIGN_PARAGRAPH.CENTER
```

如 matplotlib 未安装：
```bash
python3 -m pip install --user --break-system-packages matplotlib
```

### Step 5: 生成 Word 报告

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
6. **图表和数据分析**（matplotlib 生成，插入 Word）
7. **误差分析与总结**

**格式要求：**
- 正文：宋体 12pt，行距 1.5 倍
- 标题：黑体
- 表格：Table Grid 样式（带边框）
- 公式：居中，标注编号
- 图表：居中，标注图号和标题
- 步骤：必须体现"做了什么 → 得到什么"

**保存路径：** `/mnt/c/Users/sdg/Desktop/`

## 注意事项

- 报告语言为中文，格式正式
- 图表需安装 matplotlib，公式优先用文字描述保证 Word 兼容性
- 如果视觉模型超时，压缩图片后重试
- 数据处理结果让用户确认关键数字是否正确
