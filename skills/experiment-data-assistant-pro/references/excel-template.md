# Excel 生成代码模板

## 基础模板

```python
import openpyxl
from openpyxl.styles import Font, Alignment, Border, Side
from openpyxl.utils import get_column_letter
import os

wb = openpyxl.Workbook()
ws = wb.active
ws.title = "实验数据"

# 样式
thin = Border(left=Side('thin'), right=Side('thin'), top=Side('thin'), bottom=Side('thin'))
hf = Font(bold=True, size=11)
nf = Font(size=11)
tf = Font(bold=True, size=14)
cu = Alignment(horizontal='center', vertical='center', wrap_text=True)

# 写入表头和数据...
# 根据具体实验表格结构调整

# 保存到桌面
desktop = '/mnt/c/Users/sdg/Desktop/'
filename = '实验数据表.xlsx'
wb.save(os.path.join(desktop, filename))
print(f"✅ 已保存到桌面：{filename}")
```

## 常用操作

### 合并单元格
```python
ws.merge_cells('A1:H1')
ws['A1'] = '标题'
ws['A1'].font = tf
ws['A1'].alignment = cu
```

### 带边框的数据行
```python
for row_idx, row_data in enumerate(data, start_row):
    for col_idx, value in enumerate(row_data, 1):
        c = ws.cell(row=row_idx, column=col_idx, value=value)
        c.font = nf
        c.alignment = cu
        c.border = thin
```

### 数字格式
```python
cell.number_format = '0.00'    # 两位小数
cell.number_format = '0.000'   # 三位小数
cell.number_format = '0.0E+00' # 科学计数法
```

### 列宽
```python
ws.column_dimensions['A'].width = 14
for col in range(3, 9):
    ws.column_dimensions[get_column_letter(col)].width = 14
```
