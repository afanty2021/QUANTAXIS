# QIFI 模块文档

[根目录](../../CLAUDE.md) > [QUANTAXIS](../) > **QIFI**

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

QIFI (QuantAxis Interoperable Financial Interface) 是 **统一账户协议**，提供跨语言 (Python/Rust/C++) 和跨系统的账户一致性：

- **QIFI_Account**: 标准账户实现
- **QIFI_Manager**: 多账户管理系统
- **QIFI_SManager**: 策略账户管理器

---

## 入口与启动

```python
from QUANTAXIS.QIFI import (
    QIFI_Account,        # 账户类
    QA_QIFIMANAGER,      # 账户管理器
    QA_QIFISMANAGER,     # 策略管理器
)
```

---

## 对外接口

### QIFI_Account - 账户类

```python
account = QIFI_Account(
    account_cookie: str,
    init_cash: float = 1000000,
)

# 交易接口 (与 QARSAccount 一致)
account.buy(code, price, datetime, amount)
account.sell(code, price, datetime, amount)
account.buy_open(code, price, datetime, amount)
account.sell_open(code, price, datetime, amount)
# ...

# 查询接口
account.get_positions()
account.get_account_info()
account.get_qifi()
```

### QA_QIFIMANAGER - 账户管理器

```python
manager = QA_QIFIMANAGER()

# 添加账户
manager.add_account(account)

# 获取账户
manager.get_account(account_cookie)

# 查询所有账户
manager.accounts
```

---

## 关键依赖

```python
QUANTAXIS.QAMarket.QAOrder
QUANTAXIS.QAMarket.QAPosition
```

---

## 数据模型

### QIFI 账户结构

与 QARSBridge 相同，参见 [QARSBridge/CLAUDE.md](../QARSBridge/CLAUDE.md)

---

## 常见问题 (FAQ)

### Q: QIFI_Account 和 QARSAccount 如何选择？

A: API 完全一致，QARSAccount 性能更高 (100x)，QIFI_Account 不需要额外安装。

---

## 相关文件清单

```
QUANTAXIS/QIFI/
├── __init__.py        # 模块入口
├── QifiAccount.py     # 账户实现
├── QifiManager.py     # 管理器实现
└── qifi.md           # QIFI 协议说明
```

---

## 下一步

- 查看: [QARSBridge](../QARSBridge/CLAUDE.md) | [QAMarket](../QAMarket/CLAUDE.md)
