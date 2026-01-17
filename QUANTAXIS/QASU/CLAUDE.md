# QASU 模块文档

[根目录](../../CLAUDE.md) > [QUANTAXIS](../) > **QASU**

---

## 模块职责

数据保存和存储管理

---

## 入口与启动

```python
from QUANTAXIS.QASU import (
    # 股票数据
    QA_SU_save_stock_day,
    QA_SU_save_stock_min,
    QA_SU_save_stock_list,
    QA_SU_save_stock_info,

    # 指数数据
    QA_SU_save_index_day,
    QA_SU_save_index_min,
    QA_SU_save_index_list,

    # 财务数据
    QA_SU_save_financialfiles,

    # 策略保存
    QA_SU_save_strategy,
)
```

---

## 相关文件

- `QUANTAXIS/QASU/`
