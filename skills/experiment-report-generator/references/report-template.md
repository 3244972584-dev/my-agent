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
    p = doc.add_paragraph()
    p.alignment = WD_ALIGN_PARAGRAPH.CENTER
    run = p.add_run(text)
    run.font.size = Pt(22); run.font.bold = True; run.font.name = '黑体'

def add_heading(doc, text, level=1):
    h = doc.add_heading(text, level=level)
    for r in h.runs: r.font.name = '黑体'; r.font.color.rgb = RGBColor(0,0,0)

def add_body(doc, text, indent=True):
    p = doc.add_paragraph(text)
    if indent: p.paragraph_format.first_line_indent = Cm(0.75)

def add_formula(doc, text, num=""):
    p = doc.add_paragraph()
    p.alignment = WD_ALIGN_PARAGRAPH.CENTER
    run = p.add_run(text); run.font.italic = True
    if num: p.add_run(f'          ({num})').font.italic = False

def add_table(doc, headers, rows):
    table = doc.add_table(rows=1+len(rows), cols=len(headers))
    table.style = 'Table Grid'
    table.alignment = WD_TABLE_ALIGNMENT.CENTER
    for i, h in enumerate(headers):
        c = table.rows[0].cells[i]; c.text = h
        for p in c.paragraphs:
            p.alignment = WD_ALIGN_PARAGRAPH.CENTER
            for r in p.runs: r.font.bold = True; r.font.size = Pt(10)
    for ri, row in enumerate(rows):
        for ci, val in enumerate(row):
            c = table.rows[ri+1].cells[ci]; c.text = str(val)
            for p in c.paragraphs:
                p.alignment = WD_ALIGN_PARAGRAPH.CENTER
                for r in p.runs: r.font.size = Pt(10)

def add_figure(doc, image_path, caption, width_inches=5.5):
    doc.add_picture(image_path, width=Inches(width_inches))
    doc.paragraphs[-1].alignment = WD_ALIGN_PARAGRAPH.CENTER
    p = doc.add_paragraph()
    p.alignment = WD_ALIGN_PARAGRAPH.CENTER
    p.add_run(caption).font.size = Pt(10)

def add_step(doc, step_num, action, result):
    p = doc.add_paragraph()
    p.paragraph_format.first_line_indent = Cm(0.75)
    p.add_run(f'（{step_num}）').font.bold = True
    p.add_run(action)
    p.add_run(' → ').font.bold = True
    p.add_run(result).font.color.rgb = RGBColor(0, 0, 180)

# === 保存 ===
desktop = '/mnt/c/Users/sdg/Desktop/'
doc.save(os.path.join(desktop, '实验报告.docx'))
```

## 注意事项

- 字体：正文宋体12pt，标题黑体，行距1.5倍
- 表格：Table Grid 样式
- 公式：居中斜体 + 编号
- 图表：由 experiment-plotter 生成，用 add_figure 插入
- 步骤：必须体现"做了什么 → 得到什么"
