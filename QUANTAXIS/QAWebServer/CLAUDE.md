# QAWebServer 模块文档

[根目录](../../CLAUDE.md) > [QUANTAXIS](../) > **QAWebServer**

---

> **版本**: 2.0.0
> **作者**: @yutiansut @quantaxis
> **语言**: Python (Tornado)

---

## 变更记录 (Changelog)

### 2026-01-17 09:36:13 CST
- 📈 添加详细的 RESTful API 文档
- 📈 添加 WebSocket 接口文档
- 📈 添加完整的代码示例
- ✨ 新增 QIFI 账户管理 API 文档

### 2026-01-17 09:13:03 CST
- 📈 初始化模块文档

---

## 模块职责

QAWebServer 提供 **Tornado Web 服务器和 RESTful API**：

- **RESTful API**: 数据查询、账户管理、策略控制
- **WebSocket**: 实时行情推送、命令执行、回测运行
- **任务调度**: 动态任务管理
- **QIFI 管理**: 账户历史、月度收益、交易记录

---

## 入口与启动

### 命令行启动

```bash
# 默认端口 8010
qawebserver

# 指定端口
qawebserver --port=8080

# 指定地址
qawebserver --address=127.0.0.1
```

### 代码启动

```python
from QUANTAXIS.QAWebServer.server import start_server

# 基础启动
start_server(handlers=None, address='0.0.0.0', port=8010)

# 自定义处理器
from myapp.handlers import CustomHandler

custom_handlers = [
    (r"/", CustomHandler),
    (r"/api/data", DataHandler),
]

start_server(handlers=custom_handlers, address='0.0.0.0', port=8080)
```

---

## RESTful API 详解

### API 路由表

| 路由 | 方法 | 处理器 | 功能 |
|-----|------|-------|------|
| `/` | GET | INDEX | 欢迎页和路由列表 |
| `/command/run` | POST | CommandHandler | 执行命令 |
| `/command/runws` | WS | CommandHandlerWS | WebSocket 命令执行 |
| `/command/runbacktest` | WS | RunnerHandler | 运行回测 |
| `/scheduler/map` | GET/POST | QASchedulerHandler | 调度管理 |
| `/scheduler/query` | GET | QAScheduleQuery | 查询调度任务 |
| `/qifi` | GET | QAQIFI_Handler | QIFI 账户查询 |
| `/qifis` | GET/POST | QAQIFIS_Handler | QIFI 账户管理 |
| `/qifirealtime` | GET/POST | QAQIFIS_REALTIME_Handler | 实时账户管理 |
| `/user` | GET | QAUserhander | 用户信息 |

### 1. 欢迎页 API

#### GET `/`

获取服务状态和可用路由列表。

**响应示例**:
```json
{
  "status": 200,
  "message": "This is a welcome page for quantaxis backend",
  "url": ["/", "/command/run", "/command/runws", ...]
}
```

### 2. QIFI 账户 API

#### GET `/qifi`

查询 QIFI 账户信息。

**参数**:
- `action` (string): 操作类型
  - `acchistory`: 账户历史资产
  - `monthprofit`: 月度收益
  - `historytrade`: 历史交易记录
  - `holdingpanel`: 持仓面板
- `account_cookie` (string): 账户 cookie
- `trading_day` (string): 交易日期 (holdingpanel 需要)

**示例 1: 账户历史资产**

```bash
GET http://127.0.0.1:8010/qifi?action=acchistory&account_cookie=KTKS_t01_au2012_5min
```

**响应**:
```json
{
  "res": {
    "2020-02-03 00:00:00": 51212,
    "2020-02-04 00:00:00": 50602,
    "2020-02-05 00:00:00": 50922,
    "2020-02-06 00:00:00": 50522
  }
}
```

**示例 2: 月度收益**

```bash
GET http://127.0.0.1:8010/qifi?action=monthprofit&account_cookie=test_account
```

**响应**:
```json
{
  "res": {
    "2020-02-29 00:00:00": -899.0,
    "2020-03-31 00:00:00": 4024.0,
    "2020-04-30 00:00:00": -7704.0,
    "2020-05-31 00:00:00": 136.0,
    "2020-06-30 00:00:00": -1847.0,
    "2020-07-31 00:00:00": 60.0
  }
}
```

**示例 3: 历史交易**

```bash
GET http://127.0.0.1:8010/qifi?action=historytrade&account_cookie=test_account
```

**响应**:
```json
{
  "res": [
    {
      "commission": 2.0,
      "direction": "SELL",
      "offset": "OPEN",
      "price": 4084.1736,
      "trade_date_time": 1579141800000000000,
      "volume": 1.0,
      "code": "a2009",
      "datetime": "2020-01-16 10:30:00"
    }
  ]
}
```

### 3. QIFI 账户管理 API

#### GET `/qifis`

查询账户列表。

**参数**:
- `action` (string): 操作类型
  - `accountlist`: 账户列表
  - `portfoliolist`: 组合列表
  - `accountinportfolio`: 组合内账户

**示例 1: 获取所有账户**

```bash
GET http://127.0.0.1:8010/qifis?action=accountlist
```

**响应**:
```json
{
  "res": ["account1", "account2", "account3"]
}
```

**示例 2: 获取组合内账户**

```bash
GET http://127.0.0.1:8010/qifis?action=accountinportfolio&portfolio=test_portfolio
```

**响应**:
```json
{
  "res": [
    {"account_cookie": "acc1", "balance": 100000, "profit": 5000},
    {"account_cookie": "acc2", "balance": 200000, "profit": -2000}
  ]
}
```

#### POST `/qifis`

执行账户管理操作。

**参数**:
- `action` (string): 操作类型
  - `drop_account`: 删除账户
  - `drop_many`: 批量删除

**示例: 删除账户**

```bash
POST http://127.0.0.1:8010/qifis
Content-Type: application/x-www-form-urlencoded

action=drop_account&account_cookie=test_account
```

**响应**:
```json
{
  "res": "Account dropped successfully",
  "status": 200
}
```

### 4. 实时账户 API

#### GET/POST `/qifirealtime`

实时账户管理（与 `/qifis` 接口相同，但使用 REALTIME 模型）。

**注意**: 此端点连接到实时交易账户，请谨慎操作。

### 5. 用户信息 API

#### GET `/user`

获取用户信息。

**响应**:
```json
{
  "status": 200,
  "result": {
    "user_cookie": "xx"
  }
}
```

---

## WebSocket API 详解

### 1. 命令执行 WebSocket

#### WS `/command/runws`

执行命令并实时返回输出。

**连接**:
```javascript
const ws = new WebSocket('ws://localhost:8010/command/runws');
```

**发送命令**:
```javascript
ws.send('python /path/to/backtest.py');
```

**接收输出**:
```javascript
ws.onmessage = (event) => {
  console.log(event.data);  // 命令输出
};

ws.onclose = () => {
  console.log('backtest run success');
};
```

### 2. 回测运行 WebSocket

#### WS `/command/runbacktest`

运行回测脚本。

**连接**:
```javascript
const ws = new WebSocket('ws://localhost:8010/command/runbacktest');
```

**发送脚本路径**:
```javascript
ws.send('/path/to/strategy.py');
```

**接收输出**:
```javascript
ws.onmessage = (event) => {
  const output = event.data;
  console.log(output);
};
```

---

## 基础处理器 (Base Handlers)

### QABaseHandler

所有 API 处理器的基类，提供 CORS 支持。

**特性**:
- 自动 CORS 头设置
- 支持 JSON/XML 响应
- OPTIONS 预检请求处理

**CORS 配置**:
```python
class QABaseHandler(RequestHandler):
    def set_default_headers(self):
        self.set_header("Access-Control-Allow-Origin", "*")
        self.set_header("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE, PUT, PATCH")
        self.set_header("Access-Control-Allow-Headers", "Content-Type, Authorization, X-Requested-With")
        self.set_header("Server", "QUANTAXISBACKEND")
```

### QAWebSocketHandler

WebSocket 处理器基类。

**特性**:
- 自动 CORS 支持
- 连接管理和心跳

**示例**:
```python
from QUANTAXIS.QAWebServer.basehandles import QAWebSocketHandler

class MyWSHandler(QAWebSocketHandler):
    def open(self):
        print("WebSocket connected")

    def on_message(self, message):
        self.write_message(f"Echo: {message}")

    def on_close(self):
        print("WebSocket closed")
```

---

## 高级功能

### 自定义处理器

```python
from QUANTAXIS.QAWebServer.basehandles import QABaseHandler
import json

class DataHandler(QABaseHandler):
    def get(self):
        code = self.get_argument('code', '000001')
        start = self.get_argument('start', '2024-01-01')
        end = self.get_argument('end', '2024-12-31')

        # 查询数据
        data = fetch_data(code, start, end)

        self.write({
            'status': 200,
            'data': data.to_dict(orient='records')
        })

    def post(self):
        body = json.loads(self.request.body)
        # 处理 POST 请求
        self.write({'status': 200, 'message': 'Success'})
```

### 添加路由

```python
from QUANTAXIS.QAWebServer.server import start_server

# 自定义路由
custom_handlers = [
    (r"/api/stock", DataHandler),
    (r"/api/future", FutureHandler),
    (r"/ws/realtime", RealtimeWSHandler),
]

start_server(handlers=custom_handlers, port=8080)
```

---

## 关键依赖

```python
# 核心依赖
tornado >= 6.4.0

# 可选依赖
gevent-websocket  # WebSocket 性能优化
pyconvert         # 数据格式转换
```

---

## 配置说明

### Tornado 配置

```python
from tornado.web import Application

apps = Application(
    handlers=handlers,
    debug=True,              # 开发模式
    autoreload=True,         # 自动重载
    compress_response=True   # 响应压缩
)
```

### 环境变量

- `mongo_ip`: MongoDB 服务器地址
- `eventmq_ip`: RabbitMQ 服务器地址
- `eventmq_port`: RabbitMQ 端口
- `eventmq_username`: RabbitMQ 用户名
- `eventmq_password`: RabbitMQ 密码

---

## 安全建议

1. **生产环境配置**:
   - 关闭 `debug` 模式
   - 配置 CORS 白名单
   - 使用 HTTPS
   - 添加认证中间件

2. **CORS 配置**:
```python
def set_default_headers(self):
    # 替换 * 为具体域名
    self.set_header("Access-Control-Allow-Origin", "https://yourdomain.com")
```

3. **认证**:
```python
class AuthenticatedHandler(QABaseHandler):
    def prepare(self):
        token = self.request.headers.get("Authorization")
        if not validate_token(token):
            self.set_status(401)
            self.finish()
```

---

## 相关文件清单

```
QUANTAXIS/QAWebServer/
├── __init__.py              # 模块入口
├── server.py                # 服务器主程序
├── basehandles.py           # 基础处理器
├── commandhandler.py        # 命令处理器
├── schedulehandler.py       # 调度处理器
├── qifiserver.py            # QIFI 服务器
└── util.py                  # 工具函数
```

---

## 下一步

- 查看: [QASchedule](../QASchedule/CLAUDE.md) | [QAPubSub](../QAPubSub/CLAUDE.md) | [QIFI](../QIFI/CLAUDE.md)
