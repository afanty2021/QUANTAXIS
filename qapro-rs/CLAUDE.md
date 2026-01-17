# qapro-rs 文档

[根目录](../CLAUDE.md) > **qapro-rs**

---

> **版本**: 0.1.1
> **作者**: @yutiansut @quantaxis
> **语言**: Rust

---

## 变更记录 (Changelog)

### 2026-01-17 09:13:03 CST
- 📈 初始化模块文档

---

## 模块职责

qapro-rs 是 **QUANTAXIS 的 Rust 实现**，提供高性能核心组件：

- **QAMarket**: 市场数据结构和订单管理
- **QAData**: 数据结构和存储
- **高性能计算**: 利用 Rust 的性能优势

---

## 入口与启动

### 构建

```bash
cd qapro-rs
cargo build --release
```

### 作为 Python 扩展使用

```bash
# 安装 PyO3 绑定
pip install -e .

# 在 Python 中使用
import qars3
```

---

## 对外接口

### Python 绑定

```python
# QARS3 (PyO3)
from qars3 import QA_QIFIAccount

account = QA_QIFIAccount(
    account_cookie="test",
    init_cash=1000000,
)

# 交易接口与 QARSAccount 兼容
account.buy("000001", 10.5, "2025-01-15", 1000)
```

---

## 关键依赖

```toml
[dependencies]
serde = "1.0"           # 序列化
mongodb = "1.1"          # MongoDB
actix-web = "4.0"        # Web 框架
polars = { git = "..." } # DataFrame
clickhouse-rs = "1.0"   # ClickHouse
```

---

## 数据模型

### QIFI 账户结构 (Rust)

```rust
pub struct QIFIAccount {
    pub account_cookie: String,
    pub portfolio: String,
    pub balance: f64,
    pub available: f64,
    pub margin: f64,
    pub positions: HashMap<String, Position>,
    pub orders: HashMap<String, Order>,
    pub trades: HashMap<String, Trade>,
}
```

---

## 相关文件清单

```
qapro-rs/
├── Cargo.toml          # 项目配置
├── src/
│   ├── qamarket/       # 市场模块
│   ├── qadata/         # 数据模块
│   └── lib.rs          # 库入口
├── src/qamarket/
│   └── qahexos/        # HEXOS 交易所
└── readme.md           # 模块说明
```

---

## 下一步

- 查看: [QARSBridge](../QUANTAXIS/QARSBridge/CLAUDE.md) | [QADataBridge](../QUANTAXIS/QADataBridge/CLAUDE.md)
