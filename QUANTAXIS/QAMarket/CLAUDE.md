# QAMarket 模块文档

[根目录](../../CLAUDE.md) > [QUANTAXIS](../) > **QAMarket**

---

> **版本**: 2.1.0
> **作者**: @yutiansut @quantaxis
> **语言**: Python

---

## 变更记录 (Changelog)

### 2026-01-17 09:13:03 CST
- 📈 初始化模块文档

---

## 模块职责

QAMarket 提供**市场交易基础设施**：

- **MARKET_PRESET**: 市场参数预设 (合约乘数、最小跳价、保证金)
- **QA_Order**: 订单类 (QIFI 协议兼容)
- **QA_Position**: 持仓类 (多空分离、今昨分离)
- **QA_PMS**: 多标的持仓管理系统

---

## 入口与启动

```python
from QUANTAXIS.QAMarket import (
    MARKET_PRESET,    # 市场预设
    QA_Order,         # 订单
    QA_OrderQueue,    # 订单队列
    QA_Position,      # 持仓
    QA_PMS,           # 持仓管理系统
)
```

---

## 对外接口

### MARKET_PRESET - 市场预设

```python
preset = MARKET_PRESET()

# 获取合约信息
info = preset.get_code('RB')  # 螺纹钢
print(info)
# {
#     'unit_table': 10,        # 合约乘数
#     'price_tick': 1.0,       # 最小跳价
#     'buy_frozen_coeff': 0.12, # 保证金率
#     'sell_frozen_coeff': 0.12,
#     'commission_coeff': 0.0001, # 手续费率
# }
```

### QA_Order - 订单

```python
order = QA_Order(
    account_cookie: str,
    code: str,
    price: float,
    amount: int,
    order_direction: str,  # 'BUY'/'SELL'
    market_type: str,      # 'STOCK_CN'/'FUTURE_CN'
)

# 订单属性
order.order_id        # 订单ID
order.status          # 状态
order.trade_date      # 交易日期
order.trade_time      # 交易时间
```

### QA_Position - 持仓

```python
position = QA_Position(
    code: str,
    market_type: str,
)

# 期货操作
position.open_long(price, volume, datetime)   # 开多
position.close_long(price, volume, datetime)  # 平多
position.open_short(price, volume, datetime)  # 开空
position.close_short(price, volume, datetime) # 平空

# 查询
position.volume_long    # 多头持仓
position.volume_short   # 空头持仓
position.margin_long    # 多头保证金
position.float_profit_long  # 多头浮动盈亏
```

### QA_PMS - 持仓管理

```python
pms = QA_PMS(account_cookie='test')

# 更新持仓
pms.update_pos(code, volume_long=10, price=3500)

# 查询
pms.positions  # dict: code -> position
```

---

## 关键依赖

无外部依赖，纯 Python 实现。

---

## 数据模型

### 订单结构 (QIFI 协议)

```python
{
    'order_id': str,
    'account_cookie': str,
    'code': str,
    'direction': str,      # BUY/SELL
    'offset': str,         # OPEN/CLOSE
    'price': float,
    'amount': int,
    'status': str,         # NEW/FILLED/CANCELED
    'trade_date': str,
    'trade_time': str,
}
```

### 持仓结构

```python
{
    'code': str,
    'volume_long': int,
    'volume_short': int,
    'open_price_long': float,
    'open_price_short': float,
    'margin_long': float,
    'margin_short': float,
    'float_profit_long': float,
    'float_profit_short': float,
}
```

---

## 相关文件清单

```
QUANTAXIS/QAMarket/
├── __init__.py         # 模块入口
├── market_preset.py    # 市场预设
├── QAOrder.py          # 订单类
└── QAPosition.py       # 持仓类
```

---

## 下一步

- 查看: [QIFI](../QIFI/CLAUDE.md) | [QAFetch](../QAFetch/CLAUDE.md)
