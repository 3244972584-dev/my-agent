---
name: experiment-plotter
description: |
  根据实验数据自动生成图表（折线图、散点图、线性拟合图等）。
  当用户需要以下任一操作时触发：
  (1) 画图 (2) 绘制图表 (3) 生成曲线 (4) 数据可视化 (5) 作图 (6) 画散点图/折线图
---

# 实验图表生成器

## 触发条件

用户提供实验数据，要求生成图表。

## 绘图流程

### Step 1: 确定图表类型

根据数据特征和实验类型自动选择：

| 数据特征 | 推荐图表 | 典型场景 |
|---------|---------|---------|
| 两组连续变量 | 散点图 + 拟合线 | U-I 曲线、lg Ie'~√Ua |
| 自变量等间距 | 折线图 | 温度-时间曲线 |
| 需要外推/拟合 | 散点图 + 线性拟合 | 里查孙直线、肖特基修正 |
| 多组数据对比 | 同一图多曲线 | 不同 If 下的 Ua-Ie' |
| 分类对比 | 柱状图 | 不同材料对比 |

### Step 2: 生成图表

**环境准备：**
```bash
# 如未安装
python3 -m pip install --user --break-system-packages matplotlib numpy scipy
```

**核心代码模板：**

```python
import matplotlib
matplotlib.use('Agg')  # 无图形界面模式
import matplotlib.pyplot as plt
import numpy as np
from scipy.stats import linregress

# 中文字体（WSL 环境可能不支持，优先用英文标签）
plt.rcParams['axes.unicode_minus'] = False
```

#### 散点图 + 线性拟合

```python
def plot_scatter_with_fit(x, y, xlabel, ylabel, title, filename,
                          fit=True, extrapolate_to=0, label='Data'):
    """散点图 + 可选线性拟合 + 可选外推"""
    fig, ax = plt.subplots(figsize=(8, 6))
    ax.scatter(x, y, color='blue', zorder=5, s=60, label=label)

    if fit and len(x) >= 2:
        slope, intercept, r_val, _, _ = linregress(x, y)
        x_min = min(extrapolate_to, min(x)) if extrapolate_to is not None else min(x)
        x_fit = np.linspace(x_min, max(x) * 1.05, 100)
        y_fit = slope * x_fit + intercept
        ax.plot(x_fit, y_fit, 'r-', linewidth=1.5,
                label=f'Fit: y={slope:.4f}x+{intercept:.4f} (R²={r_val**2:.4f})')

        if extrapolate_to is not None:
            y_ext = slope * extrapolate_to + intercept
            ax.plot(extrapolate_to, y_ext, 'go', markersize=10, zorder=6,
                    label=f'Extrapolate: y={y_ext:.4f}')

    ax.set_xlabel(xlabel, fontsize=12)
    ax.set_ylabel(ylabel, fontsize=12)
    ax.set_title(title, fontsize=14)
    ax.legend(fontsize=10)
    ax.grid(True, alpha=0.3)
    fig.savefig(filename, dpi=150, bbox_inches='tight')
    plt.close()
    return filename
```

#### 多曲线对比图

```python
def plot_multi_curve(data_sets, xlabel, ylabel, title, filename):
    """多组数据在同一图上对比
    data_sets: [(x1, y1, label1), (x2, y2, label2), ...]
    """
    fig, ax = plt.subplots(figsize=(8, 6))
    colors = ['blue', 'red', 'green', 'orange', 'purple', 'brown']
    for i, (x, y, label) in enumerate(data_sets):
        ax.scatter(x, y, color=colors[i % len(colors)], s=40, label=label, zorder=5)
        ax.plot(x, y, color=colors[i % len(colors)], alpha=0.3)
    ax.set_xlabel(xlabel, fontsize=12)
    ax.set_ylabel(ylabel, fontsize=12)
    ax.set_title(title, fontsize=14)
    ax.legend(fontsize=10)
    ax.grid(True, alpha=0.3)
    fig.savefig(filename, dpi=150, bbox_inches='tight')
    plt.close()
```

#### 折线图

```python
def plot_line(x, y, xlabel, ylabel, title, filename, marker='o'):
    """简单折线图"""
    fig, ax = plt.subplots(figsize=(8, 6))
    ax.plot(x, y, marker=marker, linewidth=1.5, markersize=6)
    ax.set_xlabel(xlabel, fontsize=12)
    ax.set_ylabel(ylabel, fontsize=12)
    ax.set_title(title, fontsize=14)
    ax.grid(True, alpha=0.3)
    fig.savefig(filename, dpi=150, bbox_inches='tight')
    plt.close()
```

### Step 3: 插入到 Word 报告

如果需要插入报告：
```python
from docx.shared import Inches
from docx.enum.text import WD_ALIGN_PARAGRAPH

doc.add_picture(filename, width=Inches(5.5))
doc.paragraphs[-1].alignment = WD_ALIGN_PARAGRAPH.CENTER
p = doc.add_paragraph()
p.alignment = WD_ALIGN_PARAGRAPH.CENTER
p.add_run('图X  图表标题').font.size = Pt(10)
```

### Step 4: 保存

图表保存位置：
- 临时图片：`~/.openclaw/workspace/report_images/`
- 最终报告：`/mnt/c/Users/sdg/Desktop/`

## 作图规范

1. **坐标轴**：标注名称和单位，如 "Ua (V)"、"lg Ie'"
2. **数据点**：用蓝色圆点标记，大小适中
3. **拟合线**：红色实线，标注拟合公式和 R²
4. **外推点**：绿色圆点，醒目标注
5. **网格**：浅灰色，alpha=0.3
6. **图例**：标注清晰，字号 10-11
7. **分辨率**：dpi=150
8. **尺寸**：figsize=(8,6) 或 (7,5)
