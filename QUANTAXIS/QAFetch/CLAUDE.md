# QAFetch 模块文档

[根目录](../../CLAUDE.md) > [QUANTAXIS](../) > **QAFetch**

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

QAFetch 提供**统一的多市场数据获取接口**：

- **多数据源支持**: TDX、Tushare、同花顺、东方财富、和讯网
- **多市场覆盖**: 股票、期货、期权、港股、美股、数字货币
- **统一接口**: 所有数据源使用相同的 API

---

## 入口与启动

```python
import QUANTAXIS.QAFetch as QAFetch

# 使用指定数据源
engine = QAFetch.use('tdx')  # 或 'tushare', 'ths'
```

---

## 对外接口

### 股票数据

```python
# 日线数据
data = QAFetch.QA_fetch_get_stock_day(
    package='tdx',      # 数据源
    code='000001',      # 代码
    start='2024-01-01',
    end='2024-12-31',
    if_fq='00',         # '00'=前复权 '01'=后复权
)

# 分钟数据
data = QAFetch.QA_fetch_get_stock_min(
    package='tdx',
    code='000001',
    start='2024-01-01 09:30:00',
    end='2024-01-01 15:00:00',
    level='1min',       # 1min/5min/15min/30min/60min
)

# 实时行情
data = QAFetch.QA_fetch_get_stock_realtime('tdx', '000001')

# 股票列表
data = QAFetch.QA_fetch_get_stock_list('tdx', type_='stock')

# 除权数据
data = QAFetch.QA_fetch_get_stock_xdxr('tdx', '000001')
```

### 期货数据

```python
# 日线数据
data = QAFetch.QA_fetch_get_future_day(
    package='tdx',
    code='RB2512',
    start='2024-01-01',
    end='2024-12-31',
    frequence='day',
)

# 分钟数据
data = QAFetch.QA_fetch_get_future_min(
    package='tdx',
    code='RB2512',
    start='2024-01-01 09:00:00',
    end='2024-01-01 15:00:00',
    frequence='1min',
)

# 期货列表
data = QAFetch.QA_fetch_get_future_list('tdx')

# 实时行情
data = QAFetch.QA_fetch_get_future_realtime('tdx', 'RB2512')
```

### 指数数据

```python
# 日线
data = QAFetch.QA_fetch_get_index_day('tdx', '000001', start, end)

# 分钟
data = QAFetch.QA_fetch_get_index_min('tdx', '000001', start, end, level='1min')

# 指数列表
data = QAFetch.QA_fetch_get_index_list('tdx')
```

### 其他市场

```python
# 港股
QAFetch.QA_fetch_get_hkstock_day(package, code, start, end)
QAFetch.QA_fetch_get_hkstock_list(package)

# 美股
QAFetch.QA_fetch_get_usstock_day(package, code, start, end)
QAFetch.QA_fetch_get_usstock_list(package)

# 期权 (与期货接口相同)
QAFetch.QA_fetch_get_option_day(package, code, start, end)
```

### 成交数据

```python
# 股票成交
data = QAFetch.QA_fetch_get_stock_transaction(
    package='tdx',
    code='000001',
    start='2024-01-01 09:30:00',
    end='2024-01-01 15:00:00',
    retry=2,
)

# 实时成交
data = QAFetch.QA_fetch_get_stock_transaction_realtime('tdx', '000001')
```

---

## 关键依赖

```python
# 必需
pytdx >= 1.72          # 通达信
tushare >= 1.4.0       # Tushare Pro

# 可选
requests
lxml
beautifulsoup4
```

---

## 数据模型

### 返回格式

所有返回数据为 **Pandas DataFrame**，包含字段：

**日线数据**:
- date, open, high, low, close, volume, amount

**分钟数据**:
- datetime, open, high, low, close, volume, amount

**实时行情**:
- 代码、名称、最新价、买卖价、成交量等

---

## 相关文件清单

```
QUANTAXIS/QAFetch/
├── __init__.py          # 统一接口
├── QATdx.py             # 通达信数据源
├── QATushare.py         # Tushare 数据源
├── QAThs.py             # 同花顺数据源
├── QAEastMoney.py       # 东方财富数据源
├── QAfinancial.py       # 财务数据
├── QABinance.py         # 币安数据源
├── QAhuobi.py           # 火币数据源
└── QAQuery.py           # 数据库查询
```

---

## 下一步

- 查看: [QAData](../QAData/CLAUDE.md) | [QASU](../QASU/CLAUDE.md)
