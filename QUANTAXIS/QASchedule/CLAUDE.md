# QASchedule 模块文档

[根目录](../../CLAUDE.md) > [QUANTAXIS](../) > **QASchedule**

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

QASchedule 提供**任务调度和后台作业管理**：

- **动态任务调度**: 基于 APScheduler
- **定时任务**: Cron 表达式支持
- **远程任务**: 支持远程任务调度
- **自动运维**: 数据更新、监控等

---

## 入口与启动

```python
from QUANTAXIS.QASchedule import QAScheduler
```

---

## 对外接口

### QAScheduler

```python
scheduler = QAScheduler()

# 添加定时任务
scheduler.add_job(
    func=my_function,
    trigger='cron',
    hour=15,
    minute=0,
    id='daily_update',
)

# 添加间隔任务
scheduler.add_job(
    func=my_function,
    trigger='interval',
    seconds=60,
    id='minute_task',
)

# 启动
scheduler.start()

# 移除任务
scheduler.remove_job('daily_update')
```

---

## 关键依赖

```python
apscheduler >= 3.10.0
```

---

## 示例

```bash
# 运行调度服务器
python examples/scheduleserver.py
```

---

## 相关文件清单

```
QUANTAXIS/QASchedule/
└── __init__.py    # 模块入口
```

---

## 下一步

- 查看: [QAWebServer](../QAWebServer/CLAUDE.md) | [QAEngine](../QAEngine/CLAUDE.md)
