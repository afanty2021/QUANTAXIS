# QAStrategy 模块文档

[根目录](../../CLAUDE.md) > [QUANTAXIS](../) > **QAStrategy**

---

> **版本**: 2.0.0
> **作者**: @yutiansut @quantaxis
> **语言**: Python

---

## 变更记录 (Changelog)

### 2026-01-17 09:36:13 CST
- 📈 添加 QAStrategyCtaBase 详细的 API 文档
- 📈 添加完整的策略示例代码
- 📈 添加回测流程文档
- ✨ 新增事件回调详解
- ✨ 新增订单管理 API 文档

### 2026-01-17 09:13:03 CST
- 📈 初始化模块文档

---

## 模块职责

QAStrategy 提供**策略回测框架**：

- **QAStrategyCtaBase**: CTA 策略基类 (单标的)
- **QAMultiBase**: 多标的策略基类
- **QAHedgeBase**: 对冲/套利策略基类
- **QAFactorBase**: 因子策略基类

---

## 入口与启动

```python
from QUANTAXIS.QAStrategy import QAStrategyCtaBase
from QUANTAXIS.QAStrategy.util import QA_data_futuremin_resample
```

---

## QAStrategyCtaBase - CTA 策略基类详解

### 类初始化

```python
class QAStrategyCtaBase:
    def __init__(
        self,
        code='rb2005',           # 交易代码 (字符串或列表)
        frequence='1min',        # 数据频率 ('1min', '5min', 'day', 'tick')
        strategy_id='QA_STRATEGY', # 策略ID
        risk_check_gap=1,        # 风险检查间隔 (秒)
        portfolio='default',     # 投资组合名称
        start='2020-01-01',      # 回测开始日期
        end='2020-05-21',        # 回测结束日期
        init_cash=1000000,       # 初始资金
        send_wx=False,           # 是否发送微信通知
        data_host=eventmq_ip,    # 数据源主机
        data_port=eventmq_port,  # 数据源端口
        trade_host=eventmq_ip,   # 交易主机
        trade_port=eventmq_port, # 交易端口
        model='py'               # 模型 ('py' 或 'rust')
    ):
```

### 核心属性

| 属性 | 类型 | 说明 |
|-----|------|------|
| `code` | str/list | 交易代码 |
| `frequence` | str | 数据频率 |
| `strategy_id` | str | 策略唯一标识 |
| `market_type` | MARKET_TYPE | 市场类型 (STOCK_CN/FUTURE_CN) |
| `running_mode` | str | 运行模式 ('backtest'/'sim') |
| `init_cash` | float | 初始资金 |
| `acc` | QIFI_Account | 账户对象 |
| `positions` | QA_Position | 持仓对象 |

### 市场数据属性

| 属性 | 类型 | 说明 |
|-----|------|------|
| `market_data` | DataFrame | 市场数据 |
| `latest_price` | dict | 最新价格 {code: price} |
| `running_time` | str | 当前运行时间 |
| `bar_id` | int | 当前 bar ID |

---

## 事件回调详解

### 生命周期回调

#### `user_init()`

用户自定义初始化，在策略启动时调用一次。

```python
def user_init(self):
    """策略初始化时调用"""
    # 初始化指标参数
    self.fast_period = 5
    self.slow_period = 20

    # 初始化状态变量
    self.bar_count = 0
    self.is_long = False
```

#### `on_dailyopen()`

每日开盘时调用。

```python
def on_dailyopen(self):
    """每日开盘回调"""
    self.daily_high = None
    self.daily_low = None
```

#### `on_dailyclose()`

每日收盘时调用。

```python
def on_dailyclose(self):
    """每日收盘回调"""
    # 保存每日数据
    self.plot('daily_close', self.acc.balance, 'value')
```

### 数据回调

#### `on_bar(self, bar)`

每个 K 线触发。

```python
def on_bar(self, bar):
    """
    K线回调

    Args:
        bar: pd.DataFrame 或 dict，包含:
            - code: 代码
            - datetime: 时间
            - open: 开盘价
            - high: 最高价
            - low: 最低价
            - close: 收盘价
            - volume: 成交量
    """
    # 获取当前价格
    close = bar.get('close', bar['close'] if isinstance(bar, dict) else bar.close)

    # 策略逻辑
    if self.should_buy():
        self.send_order('BUY', 'OPEN', price=close, volume=1)
    elif self.should_sell():
        self.send_order('SELL', 'CLOSE', price=close, volume=1)
```

#### `on_tick(self, tick)`

每个 tick 触发 (仅 tick 模式)。

```python
def on_tick(self, tick):
    """
    Tick回调

    Args:
        tick: dict，包含:
            - symbol: 代码
            - last_price: 最新价
            - bid_price_1: 买一价
            - ask_price_1: 卖一价
            - volume: 成交量
            - datetime: 时间
    """
    price = tick['last_price']

    # 基于 tick 的策略逻辑
    if price > self.upper_limit:
        self.send_order('SELL', 'OPEN', price=price, volume=1)
```

#### `on_1min_bar(self)`

每分钟触发 (无论什么频率)。

```python
def on_1min_bar(self):
    """每分钟回调"""
    # 更新计时器
    self.bar_count += 1

    # 执行定时任务
    if self.bar_count % 60 == 0:
        self.rebalance()
```

### 交易回调

#### `on_deal(self, order)`

订单成交回调。

```python
def on_deal(self, order):
    """
    成交回调

    Args:
        order: dict，包含:
            - order_id: 订单ID
            - code: 代码
            - price: 成交价
            - amount: 成交量
            - trade_time: 成交时间
            - towards: 方向
    """
    print(f"成交: {order['code']} {order['towards']} "
          f"价格={order['price']} 数量={order['amount']}")

    # 更新策略状态
    if order['towards'] in ['BUY_OPEN', 'SELL_CLOSE']:
        self.is_long = True
    else:
        self.is_long = False
```

#### `on_ordererror(self, direction, offset, price, volume)`

订单错误回调。

```python
def on_ordererror(self, direction, offset, price, volume):
    """订单错误回调"""
    print(f"订单错误: {direction} {offset} @ {price} x {volume}")
    # 处理错误逻辑
```

---

## 订单管理 API

### 发送订单

#### `send_order()`

```python
def send_order(
    self,
    direction='BUY',      # 方向: 'BUY' 或 'SELL'
    offset='OPEN',        # 开平: 'OPEN' 或 'CLOSE'
    price=3925,          # 价格
    volume=10,           # 数量
    order_id='',         # 订单ID (可选，自动生成)
    code=None            # 代码 (可选，默认 self.code)
):
    """
    发送订单

    Args:
        direction: 买卖方向
        offset: 开仓/平仓
        price: 价格 (float 或 pd.Series)
        volume: 数量
        order_id: 自定义订单ID
        code: 交易代码
    """
```

**示例**:

```python
# 买入开仓
self.send_order('BUY', 'OPEN', price=4500.0, volume=2)

# 卖出平仓
self.send_order('SELL', 'CLOSE', price=4520.0, volume=1)

# 获取当前价格后下单
current_bar = self.get_current_marketdata()
price = current_bar['close'].iloc[0]
self.send_order('BUY', 'OPEN', price=price, volume=1)
```

### 订单检查

#### `check_order()`

```python
def check_order(self, direction, offset, code=None):
    """
    检查订单是否有效

    期货市场: 同方向不开仓
    股票市场: 无限制

    Args:
        direction: 方向
        offset: 开平
        code: 代码

    Returns:
        bool: True=有效, False=无效
    """
```

---

## 查询 API

### 账户查询

#### `get_cash()`

获取可用资金。

```python
cash = self.get_cash()
print(f"可用资金: {cash}")
```

#### `get_positions(code)`

获取持仓。

```python
positions = self.get_positions('000001')
print(f"多头持仓: {positions.volume_long}")
print(f"空头持仓: {positions.volume_short}")
print(f"持仓均价: {positions.open_price_long}")
print(f"浮动盈亏: {positions.float_profit}")
```

#### `get_account_info()`

获取账户信息。

```python
info = self.acc.account_msg
print(f"总资产: {info.get('balance')}")
print(f"可用: {info.get('available')}")
print(f"持仓市值: {info.get('hold')}")
```

### 市场数据查询

#### `get_current_marketdata()`

获取当前 bar 数据。

```python
bar = self.get_current_marketdata()
print(bar)
#              open   high    low  close  volume
# datetime
# 2024-01-15  10.50  10.80  10.45  10.75   5000
```

#### `get_code_marketdata(code)`

获取指定代码的历史数据。

```python
data = self.get_code_marketdata('000001')
print(data.tail(10))  # 最近10根K线
```

---

## 完整策略示例

### 双均线策略

```python
from QUANTAXIS.QAStrategy import QAStrategyCtaBase
from QUANTAXIS import QA

class MAStrategy(QAStrategyCtaBase):
    """
    双均线 CTA 策略

    策略逻辑:
    - 快线上穿慢线: 买入开仓
    - 快线下穿慢线: 卖出平仓
    """

    def user_init(self):
        """初始化参数"""
        self.fast_period = 5   # 快线周期
        self.slow_period = 20  # 慢线周期

        # 状态变量
        self.is_long = False
        self.entry_price = 0
        self.entry_bar = 0

    def on_bar(self, bar):
        """K线回调"""
        # 获取历史数据
        data = self.market_data

        # 计算均线
        if len(data) < self.slow_period:
            return

        fast_ma = data['close'].iloc[-self.fast_period:].mean()
        slow_ma = data['close'].iloc[-self.slow_period:].mean()

        # 前一根K线的均线
        prev_fast_ma = data['close'].iloc[-self.fast_period-1:-1].mean()
        prev_slow_ma = data['close'].iloc[-self.slow_period-1:-1].mean()

        current_price = bar['close']

        # 金叉买入
        if prev_fast_ma <= prev_slow_ma and fast_ma > slow_ma:
            if not self.is_long:
                self.send_order('BUY', 'OPEN', price=current_price, volume=1)
                self.is_long = True
                self.entry_price = current_price
                self.entry_bar = self.bar_id

        # 死叉卖出
        elif prev_fast_ma >= prev_slow_ma and fast_ma < slow_ma:
            if self.is_long:
                self.send_order('SELL', 'CLOSE', price=current_price, volume=1)
                self.is_long = False

        # 止损
        elif self.is_long:
            loss_ratio = (current_price - self.entry_price) / self.entry_price
            if loss_ratio < -0.05:  # 5%止损
                self.send_order('SELL', 'CLOSE', price=current_price, volume=1)
                self.is_long = False

    def on_deal(self, order):
        """成交回调"""
        direction = order.get('trade_towards', '')
        price = order.get('trade_price', 0)
        amount = order.get('trade_amount', 0)

        if direction in ['BUY_OPEN', 'SELL_CLOSE']:
            print(f"多头成交: {price} x {amount}")
        else:
            print(f"空头成交: {price} x {amount}")
```

### 期货策略示例

```python
class FutureBreakoutStrategy(QAStrategyCtaBase):
    """
    期货突破策略

    突破前N根K线最高价买入开仓
    突破前N根K线最低价卖出开仓
    """

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.lookback = 20  # 回看周期

    def user_init(self):
        self.breakout_long = False
        self.breakout_short = False

    def on_bar(self, bar):
        data = self.market_data

        if len(data) < self.lookback + 1:
            return

        # 计算突破区间
        lookback_data = data.iloc[-self.lookback-1:-1]
        high_channel = lookback_data['high'].max()
        low_channel = lookback_data['low'].min()

        current_high = bar['high']
        current_low = bar['low']
        current_close = bar['close']

        # 向上突破
        if current_high > high_channel and not self.breakout_long:
            self.send_order('BUY', 'OPEN', price=current_close, volume=1)
            self.breakout_long = True
            self.breakout_short = False

        # 向下突破
        elif current_low < low_channel and not self.breakout_short:
            self.send_order('SELL', 'OPEN', price=current_close, volume=1)
            self.breakout_short = True
            self.breakout_long = False
```

---

## 运行回测

### 方式1: 使用 QARSBridge

```python
from QUANTAXIS.QAStrategy import QAStrategyCtaBase
from QUANTAXIS.QAData import QA_DataStruct_Stock_day

# 1. 创建策略
strategy = MAStrategy(
    code='000001',
    frequence='day',
    start='2020-01-01',
    end='2024-12-31',
    init_cash=1000000
)

# 2. 运行回测
strategy.run_backtest()

# 3. 保存结果
strategy.acc.save()

# 4. 风险分析
from QUANTAXIS.QAAnalysis import QA_Risk
risk = QA_Risk(strategy.acc)
risk.save()
```

### 方式2: 模拟交易

```python
# 初始化模拟环境
strategy.debug_sim()

# 启动订阅和事件循环
strategy.run_sim()
```

---

## 关键依赖

```python
# 核心依赖
QUANTAXIS.QIFI
QUANTAXIS.QAData
QUANTAXIS.QAEngine

# 消息队列 (模拟交易需要)
pika >= 1.3.2
```

---

## 策略类型对比

| 基类 | 适用场景 | 市场类型 | 代码示例 |
|-----|---------|---------|---------|
| QAStrategyCtaBase | 单标的 CTA | 股票/期货 | 双均线、突破策略 |
| QAMultiBase | 多标的轮动 | 股票组合 | 行业轮动、多因子 |
| QAHedgeBase | 对冲/套利 | 期货/期权 | 跨期套利、价差交易 |
| QAFactorBase | 因子策略 | 全市场 | 多因子选股 |

---

## 相关文件清单

```
QUANTAXIS/QAStrategy/
├── __init__.py        # 模块入口
├── qactabase.py       # CTA 基类
├── qamultibase.py     # 多标的基类
├── qahedgebase.py     # 对冲基类
├── qafactorbase.py    # 因子基类
├── util.py            # 工具函数
└── syncoms.py         # 同步通信
```

---

## 常见问题 (FAQ)

### Q: 如何调整仓位?

```python
# 计算仓位比例
cash = self.get_cash()
price = bar['close']
position_ratio = 0.3  # 30%仓位

# 计算可买数量
max_volume = int(cash * position_ratio / price)

# 下单
self.send_order('BUY', 'OPEN', price=price, volume=max_volume)
```

### Q: 如何同时交易多个品种?

```python
# 创建策略时传入代码列表
strategy = MyStrategy(
    code=['000001', '600036', '000002'],
    frequence='1min'
)

# 在策略中处理
def on_bar(self, bar):
    code = bar.get('code', bar.name[1] if hasattr(bar, 'name') else self.code)

    # 对每个代码执行策略逻辑
    if code == '000001':
        # 000001 的逻辑
        pass
    elif code == '600036':
        # 600036 的逻辑
        pass
```

### Q: 如何记录策略日志?

```python
# 使用 QA 的日志工具
import QUANTAXIS as QA

QA.QA_util_log_info("策略开始运行")
QA.QA_util_log_info(f"当前价格: {bar['close']}")

# 或者使用 print
print(f"账户余额: {self.acc.balance}")
```

---

## 下一步

- 查看: [QARSBridge](../QARSBridge/CLAUDE.md) | [QAFactor](../QAFactor/CLAUDE.md) | [QIFI](../QIFI/CLAUDE.md)
