# QARSBridge 模块文档

[根目录](../../CLAUDE.md) > [QUANTAXIS](../) > **QARSBridge**

---

> **版本**: 2.1.0.alpha1
> **作者**: @yutiansut @quantaxis
> **语言**: Python/Rust (PyO3)

---

## 变更记录 (Changelog)

### 2026-01-17 09:13:03 CST
- 📈 初始化模块文档
- 🔗 添加接口文档和示例链接
- 📊 记录性能指标

---

## 模块职责

QARSBridge 是 **QARS2 Rust 核心的 Python 桥接层**，提供 Python 友好的接口访问高性能 Rust 组件：

- **QARSAccount**: 高性能 QIFI 账户系统 (100x 加速)
- **QARSBacktest**: Rust 回测引擎 (10x 加速)
- **自动回退**: QARS2 未安装时自动使用 Python 实现

**性能对比**:
- 创建 1000 个账户: 50秒 → 0.5秒 (100x)
- 发送 10000 个订单: 50秒 → 0.5秒 (100x)
- 10年日线回测: 30秒 → 3秒 (10x)
- 内存占用: -90%

---

## 入口与启动

### 模块入口

```python
from QUANTAXIS.QARSBridge import (
    QARSAccount,      # 高性能账户
    QARSBacktest,     # 回测引擎
    has_qars_support, # 检测 Rust 支持
    HAS_QARS,         # 全局支持标志
)
```

### 检测 Rust 支持

```python
from QUANTAXIS.QARSBridge import has_qars_support

if has_qars_support():
    print("Rust 核心可用，将获得极致性能")
else:
    print("使用 Python 实现，建议安装 QARS2")
```

---

## 对外接口

### QARSAccount - 账户管理

#### 初始化

```python
account = QARSAccount(
    account_cookie: str,        # 账户唯一标识
    portfolio: str = "",        # 组合名称
    init_cash: float = 1000000, # 初始资金
    environment: str = "backtest",  # 环境: backtest/live
)
```

#### 股票交易

```python
# 买入
account.buy(
    code: str,      # 证券代码
    price: float,   # 价格
    datetime: str,  # 时间
    amount: int,    # 数量
) -> bool

# 卖出
account.sell(
    code: str,
    price: float,
    datetime: str,
    amount: int,
) -> bool
```

#### 期货交易

```python
# 买入开仓 (做多)
account.buy_open(
    code: str,
    price: float,
    datetime: str,
    amount: int,
) -> bool

# 卖出开仓 (做空)
account.sell_open(
    code: str,
    price: float,
    datetime: str,
    amount: int,
) -> bool

# 买入平仓 (平空)
account.buy_close(
    code: str,
    price: float,
    datetime: str,
    amount: int,
) -> bool

# 卖出平仓 (平多)
account.sell_close(
    code: str,
    price: float,
    datetime: str,
    amount: int,
) -> bool
```

#### 查询方法

```python
# 获取持仓 (返回 DataFrame)
positions = account.get_positions()

# 获取账户信息 (返回 dict)
info = account.get_account_info()
# {
#     'balance': 账户权益,
#     'available': 可用资金,
#     'margin': 占用保证金,
#     'float_profit': 浮动盈亏,
#     'risk_ratio': 风险度,
# }

# 导出 QIFI 格式
qifi = account.get_qifi()
```

#### 公司行为

```python
# 分红
account.receive_dividend(
    code: str,
    dividend: float,  # 每股分红
    datetime: str,
)

# 拆股
account.stock_split(
    code: str,
    split_ratio: float,  # 拆股比例 (如 2.0 表示 1 拆 2)
    datetime: str,
)
```

#### 上下文管理

```python
with QARSAccount("test", init_cash=1000000) as account:
    account.buy("000001", 10.5, "2025-01-15", 1000)
    # 退出时自动结算
```

### QARSBacktest - 回测引擎

```python
backtest = QARSBacktest(
    start: str,  # 开始日期
    end: str,    # 结束日期
)

# TODO: 添加策略接口
```

---

## 关键依赖与配置

### 依赖

```python
# 必需
qars3 >= 0.0.45  # QARS2 Rust 核心 (PyO3 绑定)

# 回退时使用
QUANTAXIS.QIFI.QifiAccount
QUANTAXIS.QABacktest
```

### 安装 QARS2

```bash
# 方式1: 通过 extras 安装
pip install quantaxis[rust]

# 方式2: 手动安装
cd /home/quantaxis/qars2
pip install -e .

# 验证安装
python -c "from QUANTAXIS.QARSBridge import has_qars_support; print(has_qars_support())"
```

### 配置

无需额外配置，模块会自动检测 QARS2 是否可用。

---

## 数据模型

### QIFI 账户结构

```python
{
    'account_cookie': str,      # 账户标识
    'portfolio': str,           # 组合名称
    'user_cookie': str,         # 用户标识
    'accounts': {
        'balance': float,       # 账户权益
        'available': float,     # 可用资金
        'margin': float,        # 占用保证金
        'frozen': float,        # 冻结资金
        'position_profit': float,  # 持仓盈亏
        'close_profit': float,  # 已实现盈亏
    },
    'positions': {
        code: {
            'volume_long': int,     # 多头持仓
            'volume_short': int,    # 空头持仓
            'open_price_long': float,   # 多头开仓价
            'open_price_short': float,  # 空头开仓价
            'margin_long': float,       # 多头保证金
            'margin_short': float,      # 空头保证金
            'float_profit_long': float, # 多头浮动盈亏
            'float_profit_short': float,# 空头浮动盈亏
        },
        ...
    },
    'orders': {
        order_id: {
            'code': str,
            'direction': str,      # BUY/SELL
            'offset': str,         # OPEN/CLOSE
            'price': float,
            'amount': int,
            'status': str,         # 订单状态
            'trade_date': str,
            'trade_time': str,
        },
        ...
    },
    'trades': {
        trade_id: {
            # 成交记录
        },
        ...
    },
}
```

---

## 测试与质量

### 单元测试

```bash
# 运行 QARSBridge 测试
pytest QUANTAXIS/QARSBridge/tests/

# 性能基准测试
pytest QUANTAXIS/QARSBridge/tests/test_performance.py -v
```

### 手动测试

```bash
# 运行示例
python examples/qarsbridge_example.py
```

### 测试覆盖

- ✅ 账户创建和初始化
- ✅ 股票买卖操作
- ✅ 期货开平仓操作
- ✅ 持仓查询
- ✅ QIFI 导出
- ✅ 分红和拆股
- ⏳ 回测引擎 (开发中)

---

## 常见问题 (FAQ)

### Q1: QARS2 未安装会怎样？

A: 系统会自动回退到纯 Python 实现，功能完全一致，只是性能较低。建议安装 QARS2 获得 100x 加速。

### Q2: 如何验证 QARS2 是否可用？

```python
from QUANTAXIS.QARSBridge import has_qars_support, get_version_info

print(has_qars_support())  # True/False
print(get_version_info())
# {
#     'bridge_version': '2.1.0.alpha1',
#     'has_qars': True,
#     'qars_version': '0.0.45',
#     'backend': 'Rust',
# }
```

### Q3: QARSAccount 和 QIFI_Account 有什么区别？

A: API 完全一致，QARSAccount 使用 Rust 实现（100x 加速），QIFI_Account 使用 Python 实现。两者数据结构（QIFI 格式）完全兼容。

### Q4: 如何从 QIFI 数据恢复账户？

```python
account = QARSAccount.from_qifi(qifi_dict)
```

---

## 相关文件清单

```
QUANTAXIS/QARSBridge/
├── __init__.py              # 模块入口，导出接口
├── qars_account.py          # QARSAccount 账户包装器
├── qars_backtest.py         # QARSBacktest 回测引擎包装器
└── QIFI_PROTOCOL.md         # QIFI 协议规范
```

---

## 示例代码

完整示例请参考:
- [examples/qarsbridge_example.py](../../examples/qarsbridge_example.py)

---

## 下一步

- 查看文档: [QIFI 协议规范](./QIFI_PROTOCOL.md)
- 学习示例: [examples/](../../examples/)
- 查看其他模块: [QADataBridge](../QADataBridge/CLAUDE.md) | [QIFI](../QIFI/CLAUDE.md)
