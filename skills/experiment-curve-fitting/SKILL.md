---
name: experiment-curve-fitting
description: |
  对实验数据进行曲线拟合（线性、多项式、指数等），返回拟合方程、参数和R²。
  当用户需要以下任一操作时触发：
  (1) 数据拟合 (2) 线性拟合 (3) 曲线拟合 (4) 求斜率/截距 (5) 拟合方程
---

# 实验数据曲线拟合

## 触发条件

用户提供数据，要求进行曲线拟合。

## 拟合流程

### Step 1: 确定拟合类型

| 数据特征 | 推荐拟合 | 典型场景 |
|---------|---------|---------|
| 散点近似直线 | 线性拟合 | U-I曲线、lg Ie'~√Ua、lg(Ie/T²)~1/T |
| 散点呈曲线 | 多项式拟合 | 非线性伏安特性 |
| 指数增长/衰减 | 指数拟合 | 放射性衰变、RC充电 |
| 对数关系 | 对数拟合 | 声音衰减 |

### Step 2: 执行拟合

```python
import numpy as np
from scipy.stats import linregress
from scipy.optimize import curve_fit

# ============================================================
# 线性拟合（最常用）
# ============================================================
def linear_fit(x, y):
    """线性拟合: y = kx + b"""
    slope, intercept, r_value, p_value, std_err = linregress(x, y)
    return {
        'type': 'linear',
        'equation': f'y = {slope:.4f}x + {intercept:.4f}',
        'slope': slope,
        'intercept': intercept,
        'r_squared': r_value**2,
        'std_err': std_err
    }

# ============================================================
# 多项式拟合
# ============================================================
def polynomial_fit(x, y, degree=2):
    """多项式拟合: y = aₙxⁿ + ... + a₁x + a₀"""
    coeffs = np.polyfit(x, y, degree)
    y_pred = np.polyval(coeffs, x)
    ss_res = np.sum((y - y_pred)**2)
    ss_tot = np.sum((y - np.mean(y))**2)
    r_squared = 1 - ss_res / ss_tot if ss_tot > 0 else 0

    terms = []
    for i, c in enumerate(coeffs):
        power = degree - i
        if power == 0:
            terms.append(f'{c:.4f}')
        elif power == 1:
            terms.append(f'{c:.4f}x')
        else:
            terms.append(f'{c:.4f}x^{power}')

    return {
        'type': f'polynomial(degree={degree})',
        'equation': ' + '.join(terms),
        'coefficients': coeffs.tolist(),
        'r_squared': r_squared
    }

# ============================================================
# 指数拟合: y = a * exp(b*x)
# ============================================================
def exponential_fit(x, y):
    """指数拟合: y = a·e^(bx)"""
    def exp_func(x, a, b):
        return a * np.exp(b * x)
    try:
        popt, pcov = curve_fit(exp_func, x, y, p0=[1, 0.1])
        a, b = popt
        y_pred = exp_func(x, a, b)
        ss_res = np.sum((y - y_pred)**2)
        ss_tot = np.sum((y - np.mean(y))**2)
        r_squared = 1 - ss_res / ss_tot if ss_tot > 0 else 0
        return {
            'type': 'exponential',
            'equation': f'y = {a:.4f}·e^({b:.4f}x)',
            'a': a, 'b': b,
            'r_squared': r_squared
        }
    except Exception as e:
        return {'type': 'exponential', 'error': str(e)}

# ============================================================
# 对数拟合: y = a * ln(x) + b
# ============================================================
def logarithmic_fit(x, y):
    """对数拟合: y = a·ln(x) + b"""
    x_pos = np.array(x)
    y_arr = np.array(y)
    mask = x_pos > 0
    if mask.sum() < 2:
        return {'type': 'logarithmic', 'error': '需要x>0的数据点'}
    ln_x = np.log(x_pos[mask])
    slope, intercept, r_value, _, _ = linregress(ln_x, y_arr[mask])
    return {
        'type': 'logarithmic',
        'equation': f'y = {slope:.4f}·ln(x) + {intercept:.4f}',
        'a': slope, 'b': intercept,
        'r_squared': r_value**2
    }
```

### Step 3: 输出拟合结果

格式化输出：

```
📈 拟合结果

  拟合类型: 线性
  拟合方程: y = -25834.15x + 1.4146
  斜率: -25834.15
  截距: 1.4146
  R²: 0.9974

  拟合质量: ★★★★★ (R² > 0.99，线性关系显著)
```

**拟合质量评价：**

| R² 范围 | 评价 | 图示 |
|---------|------|------|
| > 0.99 | 优秀 | ★★★★★ |
| 0.95 ~ 0.99 | 良好 | ★★★★ |
| 0.90 ~ 0.95 | 一般 | ★★★ |
| 0.80 ~ 0.90 | 较差 | ★★ |
| < 0.80 | 不适用该拟合 | ★ |

### Step 4: 自动推荐

当用户未指定拟合类型时：
1. 先尝试线性拟合
2. 若 R² < 0.95，尝试多项式（2阶）
3. 若仍不佳，尝试指数/对数
4. 推荐最优拟合并说明原因
