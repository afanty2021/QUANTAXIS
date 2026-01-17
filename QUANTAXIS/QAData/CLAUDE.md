# QAData 模块文档

[根目录](../../CLAUDE.md) > [QUANTAXIS](../) > **QAData**

---

> **版本**: 2.0.0
> **作者**: @yutiansut @quantaxis
> **语言**: Python

---

## 变更记录 (Changelog)

### 2026-01-17 09:13:03 CST
- 📈 初始化模块文档

---

## 模块职责

QAData 提供**内存数据库和数据结构**：

- **数据结构**: 日线/分钟/Tick/财务/指标等多标的数据结构
- **数据转换**: 复权、重采样、市值计算
- **数据方法**: 数据库查询封装

---

## 入口与启动

```python
from QUANTAXIS.QAData import (
    # 数据结构
    QA_DataStruct_Stock_day,
    QA_DataStruct_Stock_min,
    QA_DataStruct_Future_day,
    QA_DataStruct_Future_min,
    QA_DataStruct_Index_day,
    QA_DataStruct_Index_min,

    # 数据转换
    QA_data_stock_to_fq,
    QA_data_min_resample,
    QA_data_day_resample,

    # 数据方法
    QDS_StockDayWarpper,
    QDS_IndexDayWarpper,
)
```

---

## 对外接口

### 数据结构

#### QA_DataStruct_Stock_day - 股票日线

```python
data = QA_DataStruct_Stock_day(
    code='000001',
    start='2024-01-01',
    end='2024-12-31',
    frequence='day',
    market='stock_cn',
)

# 查询
data.len()              # 数据长度
data.open               # 开盘价序列
data.close              # 收盘价序列
data.data               # 完整数据

# 添加数据
data.append(new_data)

# 合并
new_data = data.concat(other_data)
```

#### QA_DataStruct_Future_min - 期货分钟

```python
data = QA_DataStruct_Future_min(
    code='RB2512',
    start='2024-01-01 09:00:00',
    end='2024-01-01 15:00:00',
    frequence='1min',
    market='future_cn',
)
```

### 数据转换

#### 复权

```python
# 前复权
fq_data = QA_data_stock_to_fq(
    data,
    method='qfq',  # 'qfq'=前复权 'hfq'=后复权
)
```

#### 重采样

```python
# 分钟转日线
day_data = QA_data_min_to_day(min_data)

# 分钟重采样
resampled = QA_data_min_resample(
    data,
    new_frequence='15min',
)

# Tick 转分钟
min_data = QA_data_tick_resample(tick_data)
```

#### 市值计算

```python
# 计算市值
mv_data = QA_data_calc_marketvalue(
    data,
    market_cap='circulating',  # 'total'=总市值 'circulating'=流通市值
)
```

### 数据查询

```python
# 数据库查询封装
wrapper = QDS_StockDayWarpper(
    code='000001',
    start='2024-01-01',
    end='2024-12-31',
)

# 查询
data = wrapper.data()
```

---

## 关键依赖

```python
pandas >= 2.0.0
numpy >= 1.24.0
```

---

## 数据模型

### 数据结构接口

所有数据结构实现统一接口：

```python
class QADataStruct:
    # 属性
    code: str           # 代码
    start: str          # 开始时间
    end: str            # 结束时间
    frequence: str      # 频率
    market: str         # 市场

    # 数据
    open: Series
    high: Series
    low: Series
    close: Series
    volume: Series
    amount: Series

    # 方法
    len() -> int
    append(data)
    concat(data) -> QADataStruct
    to_json() -> str
    to_df() -> DataFrame
```

---

## 相关文件清单

```
QUANTAXIS/QAData/
├── __init__.py              # 模块入口
├── QADataStruct.py          # 数据结构基类
├── QAFinancialStruct.py     # 财务数据结构
├── QAIndicatorStruct.py     # 指标数据结构
├── QASeriesStruct.py        # 序列数据结构
├── data_fq.py               # 复权
├── data_resample.py         # 重采样
├── data_marketvalue.py      # 市值计算
└── dsmethods.py             # 数据库查询
```

---

## 下一步

- 查看: [QAFetch](../QAFetch/CLAUDE.md) | [QAStrategy](../QAStrategy/CLAUDE.md)
