---
name: experiment-data-validator
description: |
  检查实验数据质量，包括异常值检测、单位一致性、趋势合理性分析。
  当用户需要以下任一操作时触发：
  (1) 检测数据是否合理 (2) 发现异常点 (3) 分析误差来源 (4) 数据质量检查 (5) 验证实验数据
---

# 实验数据验证器

## 触发条件

用户提供实验数据，要求检查数据质量、发现异常点或分析误差来源。

## 验证流程

### Step 1: 确定实验类型

根据用户描述或数据特征判断实验类型：

| 类型 | 典型数据特征 |
|------|------------|
| 电学 | 电压(V)、电流(A/mA/μA)、电阻(Ω/kΩ) |
| 热学 | 温度(K/°C)、时间(s)、热量(J) |
| 化学 | 浓度(mol/L)、质量(g)、体积(mL) |
| 力学 | 位移(m)、时间(s)、力(N)、质量(kg) |
| 光学 | 角度(°)、波长(nm)、折射率(无量纲) |

### Step 2: 异常值检测

**方法一：统计检测**
```python
import numpy as np

def detect_outliers_zscore(values, threshold=2.5):
    """Z-score 法检测异常值"""
    mean = np.mean(values)
    std = np.std(values)
    outliers = []
    for i, v in enumerate(values):
        z = abs(v - mean) / std if std > 0 else 0
        if z > threshold:
            outliers.append({'index': i, 'value': v, 'z_score': round(z, 2)})
    return outliers

def detect_outliers_iqr(values, factor=1.5):
    """IQR 法检测异常值"""
    q1, q3 = np.percentile(values, [25, 75])
    iqr = q3 - q1
    lower = q1 - factor * iqr
    upper = q3 + factor * iqr
    outliers = []
    for i, v in enumerate(values):
        if v < lower or v > upper:
            outliers.append({'index': i, 'value': v, 'lower_bound': round(lower, 4), 'upper_bound': round(upper, 4)})
    return outliers
```

**方法二：趋势偏差检测**
- 对数据做线性/多项式拟合
- 计算每个点的残差
- 残差显著偏大的点标记为异常

```python
from scipy.stats import linregress

def detect_trend_outliers(x, y, threshold=3.0):
    """基于拟合残差的异常值检测"""
    slope, intercept, _, _, _ = linregress(x, y)
    y_pred = slope * np.array(x) + intercept
    residuals = np.array(y) - y_pred
    std_res = np.std(residuals)
    outliers = []
    for i, r in enumerate(residuals):
        if std_res > 0 and abs(r) / std_res > threshold:
            outliers.append({
                'index': i, 'x': x[i], 'y': y[i],
                'predicted': round(y_pred[i], 4),
                'residual': round(r, 4),
                'sigma': round(abs(r) / std_res, 2)
            })
    return outliers
```

### Step 3: 单位与数量级检查

对每组数据检查：

| 检查项 | 方法 |
|--------|------|
| 数量级合理性 | 与同列其他值比较，偏差超过 10 倍则标记 |
| 单位一致性 | 同一物理量的数值应在同一量级范围 |
| 常见单位错误 | mA 写成 A（偏大 1000 倍）、mV 写成 V（偏大 1000 倍） |
| 正负号 | 电流、电压等是否出现不合理负值 |

**常见物理量合理范围：**

| 物理量 | 典型范围 | 实验类型 |
|--------|---------|---------|
| 阳极电流 Ie' | 0.01 μA ~ 10 mA | 逸出功 |
| 加速电压 Ua | 1 V ~ 200 V | 逸出功 |
| 灯丝电流 If | 0.1 A ~ 1 A | 逸出功 |
| 温度 T | 300 K ~ 3000 K | 热学 |
| 电阻 R | 1 Ω ~ 10 MΩ | 电学 |

### Step 4: 趋势合理性分析

根据实验类型验证数据趋势：

| 实验类型 | 预期趋势 | 违反标志 |
|---------|---------|---------|
| 逸出功 | Ie' 随 Ua 单调递增 | Ie' 下降 |
| 逸出功 | lg Ie' 与 √Ua 近似线性 | 严重非线性 |
| 伏安法 | U 与 I 线性（欧姆定律） | 非线性偏移 |
| 单摆 | T² 与 L 线性 | 严重偏离 |

### Step 5: 数据完整性检查

- 缺失值：空行、NaN、0（可能表示未测量）
- 重复值：完全相同的行
- 数据点数量：是否与实验要求一致

### Step 6: 输出验证报告

以结构化方式输出检查结果：

```
📋 数据验证报告

⚠️ 异常值（共 N 处）：
  - 第 X 行，字段 Y：值 Z（偏离均值 A 个标准差）
  - ...

✅ 单位检查：通过 / 发现问题
  - ...

✅ 趋势分析：合理 / 发现异常
  - ...

⚠️ 完整性：
  - 缺失值：X 处
  - 重复值：X 处

💡 修正建议：
  1. ...
  2. ...
```

## 注意事项

- 标记异常时给出具体行号和数值，方便用户定位
- 区分"确实异常"和"合理但意外"的数据
- 提供修正建议而非直接修改数据
- 如果数据来自图片识别，提醒用户可能是 OCR 误差
