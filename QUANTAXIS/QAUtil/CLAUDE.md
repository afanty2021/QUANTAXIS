# QAUtil 模块文档

[根目录](../../CLAUDE.md) > [QUANTAXIS](../) > **QAUtil**

---

## 模块职责

工具函数和资源管理器

---

## 入口与启动

```python
from QUANTAXIS.QAUtil import (
    # 交易时间
    QA_util_get_trade_date,
    QA_util_get_trade_range,
    QA_util_if_trade,

    # 资源管理器 (v2.1 新增)
    QAMongoResourceManager,
    QARabbitMQResourceManager,
    QAClickHouseResourceManager,
    QARedisResourceManager,
    get_mongo_resource,
    get_rabbitmq_resource,
)
```

---

## 相关文件

- `QUANTAXIS/QAUtil/`
- [examples/resource_manager_example.py](../../examples/resource_manager_example.py)
