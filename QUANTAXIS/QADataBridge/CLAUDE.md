# QADataBridge 模块文档

[根目录](../../CLAUDE.md) > [QUANTAXIS](../) > **QADataBridge**

---

> **版本**: 2.1.0.alpha2
> **作者**: @yutiansut @quantaxis
> **语言**: Python/Rust

---

## 变更记录 (Changelog)

### 2026-01-17 09:13:03 CST
- 📈 初始化模块文档
- 🔗 添加接口文档和示例链接

---

## 模块职责

QADataBridge 是 **跨语言零拷贝数据交换桥接层**，基于 QADataSwap 提供 Python/Rust/C++ 之间的高效数据传输：

- **零拷贝转换**: Pandas ↔ Polars ↔ Arrow
- **共享内存通信**: 跨进程数据传输 (7x 加速)
- **自动回退**: QADataSwap 未安装时使用标准序列化

**性能对比**:
- Pandas → Polars: 2.5x 加速
- 共享内存传输: 7x 加速
- 零拷贝转换: 5-10x 加速

---

## 入口与启动

### 模块入口

```python
from QUANTAXIS.QADataBridge import (
    # 检测支持
    has_dataswap_support,
    HAS_DATASWAP,

    # 转换函数
    convert_pandas_to_polars,
    convert_polars_to_pandas,
    convert_pandas_to_arrow,
    convert_arrow_to_pandas,

    # 共享内存
    SharedMemoryWriter,
    SharedMemoryReader,
)
```

### 检测 QADataSwap 支持

```python
from QUANTAXIS.QADataBridge import has_dataswap_support

if has_dataswap_support():
    print("零拷贝通信可用")
else:
    print("使用传统数据传输")
```

---

## 对外接口

### 数据转换

#### Pandas ↔ Polars

```python
import pandas as pd
from QUANTAXIS.QADataBridge import convert_pandas_to_polars, convert_polars_to_pandas

# Pandas → Polars (零拷贝)
df_pandas = pd.DataFrame({'price': [10.5, 20.3], 'volume': [1000, 2000]})
df_polars = convert_pandas_to_polars(df_pandas)

# Polars → Pandas
df_pandas_back = convert_polars_to_pandas(df_polars)
```

#### Pandas ↔ Arrow

```python
from QUANTAXIS.QADataBridge import convert_pandas_to_arrow, convert_arrow_to_pandas

# Pandas → Arrow (零拷贝)
table = convert_pandas_to_arrow(df_pandas)

# Arrow → Pandas
df_back = convert_arrow_to_pandas(table)
```

#### Polars ↔ Arrow

```python
from QUANTAXIS.QADataBridge import convert_polars_to_arrow, convert_arrow_to_polars

# Polars → Arrow (零拷贝)
table = convert_polars_to_arrow(df_polars)

# Arrow → Polars
df_back = convert_arrow_to_polars(table)
```

### 共享内存通信

#### SharedMemoryWriter - 写入数据

```python
from QUANTAXIS.QADataBridge import SharedMemoryWriter

# 创建写入器
writer = SharedMemoryWriter(
    name: str,              # 共享内存名称
    size_mb: int = 50,      # 大小 (MB)
)

# 写入数据
writer.write(
    data,                   # Pandas/Polars DataFrame
    timeout_ms: int = 5000, # 超时时间
)

# 关闭
writer.close()
```

#### SharedMemoryReader - 读取数据

```python
from QUANTAXIS.QADataBridge import SharedMemoryReader

# 创建读取器
reader = SharedMemoryReader(
    name: str,              # 共享内存名称
)

# 读取数据
data = reader.read(
    timeout_ms: int = 5000, # 超时时间
)

# 关闭
reader.close()
```

---

## 关键依赖与配置

### 依赖

```python
# 必需
qadataswap >= 0.1.0  # 跨语言零拷贝通信

# 回退时使用
polars  # Polars DataFrame
pyarrow  # Apache Arrow
```

### 安装 QADataSwap

```bash
# 方式1: 通过 extras 安装
pip install quantaxis[rust]

# 方式2: 手动安装
cd /home/quantaxis/qars2/libs/qadataswap
pip install -e .

# 验证安装
python -c "from QUANTAXIS.QADataBridge import has_dataswap_support; print(has_dataswap_support())"
```

---

## 数据模型

### 支持的数据格式

- **Pandas DataFrame**: 标准格式
- **Polars DataFrame**: 高性能格式
- **Arrow Table**: 跨语言格式

---

## 测试与质量

### 单元测试

```bash
# 运行测试
pytest QUANTAXIS/QADataBridge/tests/
```

### 手动测试

```bash
# 运行示例
python examples/qadatabridge_example.py
```

---

## 常见问题 (FAQ)

### Q1: QADataSwap 未安装会怎样？

A: 系统会自动回退到标准转换，功能完全一致，只是性能较低。

### Q2: 何时使用共享内存？

A: 需要在进程间传输大量数据时，如实时行情分发、策略间数据共享。

### Q3: Polars 和 Pandas 性能对比？

A: Polars 在大数据量下性能更优，建议数据量 > 100万行时使用。

---

## 相关文件清单

```
QUANTAXIS/QADataBridge/
├── __init__.py              # 模块入口
├── arrow_converter.py       # Arrow 格式转换
└── shared_memory.py         # 共享内存通信
```

---

## 示例代码

完整示例: [examples/qadatabridge_example.py](../../examples/qadatabridge_example.py)

---

## 下一步

- 查看其他模块: [QARSBridge](../QARSBridge/CLAUDE.md) | [QAData](../QAData/CLAUDE.md)
