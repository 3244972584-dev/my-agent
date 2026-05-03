# 常见数据处理方法

## 线性拟合

```python
from scipy.stats import linregress
slope, intercept, r_value, p_value, std_err = linregress(x, y)
# slope: 斜率, intercept: 截距, r_value**2: R²
```

## 线性插值

```python
import numpy as np
T = np.interp(If_measured, If_table, T_table)
```

## 误差分析

### 直接测量误差
- 多次测量：标准差 σ = std(values)
- 单次测量：取仪器最小刻度的一半

### 间接测量误差（误差传递）
```python
# 若 f = f(a, b)，则 Δf = √((∂f/∂a · Δa)² + (∂f/∂b · Δb)²)
```

### 相对误差
```python
relative_error = abs(measured - theoretical) / theoretical * 100  # %
```

## 常见物理实验数据处理

| 实验类型 | 处理方法 | 关键步骤 | 核心图表 |
|---------|---------|---------|---------|
| 逸出功测量 | 里查孙直线法 | lg(Ie/T²) vs 1/T 拟合 | lg Ie'~√Ua, lg(Ie/T²)~1/T |
| 电阻测量 | 伏安法 | U-I 线性拟合 | U-I 曲线 |
| 单摆测 g | 周期法 | T² vs L 拟合 | T²~L |
| 牛顿环 | 等厚干涉 | D² vs m 拟合 | D²~m |
| 分光计 | 最小偏向角 | 折射率计算 | 偏向角曲线 |
| 热学实验 | 冷却法 | ln(T-T₀) vs t 拟合 | 温度-时间曲线 |

## 作图法要点

1. 选坐标：自变量为横轴，因变量为纵轴
2. 定比例：充分利用图纸，数据点分散均匀
3. 标点：用 × 或 ● 标记数据点
4. 连线：直线拟合或光滑曲线（不强制过每个点）
5. 求斜率：取直线上相距较远的两点计算
6. 标注：图号、标题、坐标轴名称和单位

## 图表生成代码模板

```python
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
import numpy as np

plt.rcParams['font.sans-serif'] = ['SimHei', 'WenQuanYi Micro Hei']
plt.rcParams['axes.unicode_minus'] = False

def plot_scatter_with_fit(x, y, xlabel, ylabel, title, filename, label='实验数据'):
    """散点图 + 线性拟合"""
    fig, ax = plt.subplots(figsize=(8, 6))
    ax.scatter(x, y, color='blue', zorder=5, label=label)
    
    # 线性拟合
    from scipy.stats import linregress
    slope, intercept, r_value, _, _ = linregress(x, y)
    x_fit = np.linspace(min(x), max(x), 100)
    y_fit = slope * x_fit + intercept
    ax.plot(x_fit, y_fit, 'r-', label=f'拟合 (R²={r_value**2:.4f})')
    
    ax.set_xlabel(xlabel, fontsize=12)
    ax.set_ylabel(ylabel, fontsize=12)
    ax.set_title(title, fontsize=14)
    ax.legend(fontsize=11)
    ax.grid(True, alpha=0.3)
    fig.savefig(filename, dpi=150, bbox_inches='tight')
    plt.close()
    return filename
```
