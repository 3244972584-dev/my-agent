# Word 实验报告模板代码

## 基础框架

```python
from docx import Document
from docx.shared import Pt, Cm, RGBColor, Inches
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.enum.table import WD_TABLE_ALIGNMENT
import os

doc = Document()

# === 全局样式 ===
style = doc.styles['Normal']
style.font.name = '宋体'
style.font.size = Pt(12)
style.paragraph_format.line_spacing = 1.5

# === 辅助函数 ===

def add_title(doc, text):
    """一级标题"""
    p = doc.add_paragraph()
    p.alignment = WD_ALIGN_PARAGRAPH.CENTER
    run = p.add_run(text)
    run.font.size = Pt(22)
    run.font.bold = True
    run.font.name = '黑体'

def add_heading(doc, text, level=1):
    """节标题"""
    h = doc.add_heading(text, level=level)
    for run in h.runs:
        run.font.name = '黑体'
        run.font.color.rgb = RGBColor(0, 0, 0)

def add_body(doc, text, indent=True):
    """正文段落"""
    p = doc.add_paragraph(text)
    if indent:
        p.paragraph_format.first_line_indent = Cm(0.75)
    return p

def add_table(doc, headers, rows):
    """添加表格"""
    table = doc.add_table(rows=1 + len(rows), cols=len(headers))
    table.style = 'Table Grid'
    table.alignment = WD_TABLE_ALIGNMENT.CENTER
    for i, h in enumerate(headers):
        cell = table.rows[0].cells[i]
        cell.text = h
        for p in cell.paragraphs:
            p.alignment = WD_ALIGN_PARAGRAPH.CENTER
            for run in p.runs:
                run.font.bold = True
                run.font.size = Pt(10)
    for r_idx, row in enumerate(rows):
        for c_idx, val in enumerate(row):
            cell = table.rows[r_idx + 1].cells[c_idx]
            cell.text = str(val)
            for p in cell.paragraphs:
                p.alignment = WD_ALIGN_PARAGRAPH.CENTER
                for run in p.runs:
                    run.font.size = Pt(10)

def add_formula(doc, text, num=""):
    """添加公式描述（居中斜体 + 编号）"""
    p = doc.add_paragraph()
    p.alignment = WD_ALIGN_PARAGRAPH.CENTER
    run = p.add_run(text)
    run.font.italic = True
    if num:
        run2 = p.add_run(f'    ({num})')
        run2.font.italic = False

def add_figure(doc, image_path, caption, width_inches=5):
    """插入图表"""
    doc.add_picture(image_path, width=Inches(width_inches))
    last = doc.paragraphs[-1]
    last.alignment = WD_ALIGN_PARAGRAPH.CENTER
    p = doc.add_paragraph()
    p.alignment = WD_ALIGN_PARAGRAPH.CENTER
    run = p.add_run(caption)
    run.font.size = Pt(10)

def add_step(doc, step_num, action, result):
    """添加实验步骤（做了什么 → 得到什么）"""
    p = doc.add_paragraph()
    p.paragraph_format.first_line_indent = Cm(0.75)
    run_num = p.add_run(f'（{step_num}）')
    run_num.font.bold = True
    p.add_run(action)
    run_arrow = p.add_run(' → ')
    run_arrow.font.bold = True
    run_res = p.add_run(result)
    run_res.font.color.rgb = RGBColor(0, 0, 180)

# === 报告结构 ===

add_title(doc, '实验名称')

add_heading(doc, '一、实验目的', level=1)
# 添加目的内容

add_heading(doc, '二、实验原理', level=1)
# 添加原理、公式（用 add_formula，标注编号）

add_heading(doc, '三、实验器材', level=1)
# 添加器材列表

add_heading(doc, '四、实验步骤', level=1)
# 用 add_step() 添加每一步

add_heading(doc, '五、实验数据与处理', level=1)
# 添加表格、计算过程、图表

add_heading(doc, '六、实验总结', level=1)
# 结果、误差分析、改进建议

# === 保存 ===
desktop = '/mnt/c/Users/sdg/Desktop/'
doc.save(os.path.join(desktop, '实验报告.docx'))
```

## 图表生成函数

```python
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
import numpy as np
from scipy.stats import linregress

plt.rcParams['font.sans-serif'] = ['SimHei', 'WenQuanYi Micro Hei']
plt.rcParams['axes.unicode_minus'] = False

def plot_schottky(sqrt_ua, lg_Ie_prime, T, if_val, filename):
    """肖特基修正图：lg Ie' vs √Ua"""
    fig, ax = plt.subplots(figsize=(8, 6))
    ax.scatter(sqrt_ua, lg_Ie_prime, color='blue', zorder=5, label='实验数据')
    
    slope, intercept, r_val, _, _ = linregress(sqrt_ua, lg_Ie_prime)
    x_fit = np.linspace(0, max(sqrt_ua)*1.05, 100)
    y_fit = slope * x_fit + intercept
    ax.plot(x_fit, y_fit, 'r-', label=f'拟合 (R²={r_val**2:.4f})')
    ax.plot(0, intercept, 'go', markersize=10, label=f'外推 lg Ie = {intercept:.4f}')
    
    ax.set_xlabel('√Ua (V^0.5)', fontsize=12)
    ax.set_ylabel("lg Ie'", fontsize=12)
    ax.set_title(f'肖特基修正 (If={if_val}A, T={T:.1f}K)', fontsize=14)
    ax.legend()
    ax.grid(True, alpha=0.3)
    fig.savefig(filename, dpi=150, bbox_inches='tight')
    plt.close()

def plot_richardson(inv_T, lg_Ie_T2, slope, intercept, r2, filename):
    """里查孙直线图：lg(Ie/T²) vs 1/T"""
    fig, ax = plt.subplots(figsize=(8, 6))
    ax.scatter(inv_T*1e4, lg_Ie_T2, color='blue', zorder=5, label='实验数据')
    
    x_fit = np.linspace(min(inv_T)*1e4*0.98, max(inv_T)*1e4*1.02, 100)
    y_fit = slope * (x_fit*1e-4) + intercept
    ax.plot(x_fit, y_fit, 'r-', label=f'拟合 (R²={r2:.4f})')
    
    ax.set_xlabel('1/T (×10⁻⁴ K⁻¹)', fontsize=12)
    ax.set_ylabel('lg(Ie/T²)', fontsize=12)
    ax.set_title('里查孙直线', fontsize=14)
    ax.legend()
    ax.grid(True, alpha=0.3)
    fig.savefig(filename, dpi=150, bbox_inches='tight')
    plt.close()
```

## 注意事项

- 字体：正文宋体12pt，标题黑体，行距1.5倍
- 表格：Table Grid 样式（带边框）
- 公式：居中斜体 + 编号，如 `式(1)`
- 图表：matplotlib 生成，dpi=150，居中插入，标注图号标题
- 步骤：必须体现"做了什么 → 得到什么"
