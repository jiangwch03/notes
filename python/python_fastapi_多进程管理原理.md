# FastAPI 多进程原理详解

FastAPI 本身是一个 ASGI 框架，**它本身并不直接负责多进程管理**，多进程能力来自于运行它的 ASGI 服务器（如 Uvicorn、Gunicorn）。下面结合项目 `main.py` 中 `workers` 参数的实际使用场景来讲解。

---

## 一、核心原理：Pre-fork 多进程模型

Uvicorn / Gunicorn 采用经典的 **Pre-fork Worker 模型**：

```
        ┌─────────────────┐
        │   Master 主进程  │  ← 监听端口、管理 worker
        │   (Supervisor)  │
        └────────┬────────┘
                 │ fork()
       ┌─────────┼─────────┬─────────┐
       ▼         ▼         ▼         ▼
   ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐
   │Worker1│ │Worker2│ │Worker3│ │Worker4│  ← 各自独立的 Python 进程
   └───────┘ └───────┘ └───────┘ └───────┘
       │         │         │         │
       └─────────┴────┬────┴─────────┘
                      ▼
                共享同一个 Socket
              (SO_REUSEPORT / fd继承)
```

### 关键流程

1. **Master 进程启动**：绑定 `host:port`，创建监听 Socket。
2. **Fork 子进程**：根据 `workers=N` 派生 N 个 worker 子进程，子进程**继承 socket 文件描述符**。
3. **内核负载均衡**：操作系统内核在 `accept()` 系统调用时，把新连接分发给空闲的 worker（惊群问题由内核 `SO_REUSEPORT` 或互斥锁解决）。
4. **Worker 独立运行**：每个 worker 内部跑一个完整的 **asyncio 事件循环**，处理被分配到的请求。
5. **主进程监控**：worker 异常退出时，master 自动拉起新的 worker。

---

## 二、为什么需要多进程？—— 绕过 GIL

Python 的 **GIL（全局解释器锁）** 导致单进程内多线程无法真正并行执行 CPU 密集型代码。

| 模型 | 并发能力 | 并行能力 | 适用场景 |
|------|---------|---------|---------|
| 单进程 + asyncio | 高（单核） | 否 | I/O 密集（DB、网络） |
| 多进程 + asyncio | 高（多核） | 是 | CPU + I/O 混合 |
| 多线程 | 受限于 GIL | 否 | 不推荐 |

**FastAPI 的最佳实践 = 多进程（突破 GIL） + 每进程内 asyncio（高并发 I/O）**

---

## 三、项目代码中 `workers` 的行为

`src/main.py` 中：

```python
uvicorn.run(
    app='knowledge_rag.server.server:create_app',
    workers=AppConfig.app_workers,
    reload=AppConfig.app_reload,
    factory=True,
)
```

需要注意 Uvicorn 的几个**关键限制**：

### 1. `workers` 与 `reload` 互斥
- `reload=True` 时，`workers` 参数会被忽略（reload 模式只能单进程）。
- 生产环境必须 `reload=False` 才能启用多进程。

### 2. `factory=True` 的意义
- 每个 worker 进程启动后，会**各自调用一次** `create_app()` 工厂函数。
- 这意味着：**每个进程拥有独立的 FastAPI 实例、独立的内存空间、独立的连接池**。

### 3. 跨进程不共享状态

| 资源 | 是否共享 |
|------|---------|
| Python 对象、全局变量 | ❌ 各进程独立 |
| 数据库连接池（SQLAlchemy） | ❌ 每进程独立池 |
| 内存缓存（dict、lru_cache） | ❌ 不共享 |
| Redis、数据库 | ✅ 外部共享 |
| 监听 Socket | ✅ 内核层共享 |

---

## 四、多进程下的注意事项

### 1. 连接池规模
假设 `workers=4`，每个 SQLAlchemy 引擎 `pool_size=10`，**实际数据库连接数 = 4 × 10 = 40**，需评估 DB 承载力。

### 2. 启动钩子会执行 N 次
FastAPI 的 `lifespan` / `startup` 事件在**每个 worker 都会触发一次**，初始化逻辑要保证幂等。

### 3. 单例陷阱
进程内的 `@lru_cache`、单例对象在不同 worker 之间是**完全独立**的副本，需要全局一致性时应使用 Redis 等外部存储。

### 4. 生产推荐：Gunicorn + UvicornWorker
对于更复杂的进程管理（优雅重启、信号处理、worker 超时），生产环境通常用：

```bash
gunicorn knowledge_rag.server.server:create_app \
    --workers 4 \
    --worker-class uvicorn.workers.UvicornWorker \
    --bind 0.0.0.0:8000
```

Gunicorn 提供更成熟的 master 进程管理，Uvicorn 负责 ASGI 协议解析。

---

## 五、请求处理的完整链路

```
客户端请求
    ↓
内核 Socket 队列
    ↓ (内核分发)
Worker 进程 N
    ↓
Uvicorn (HTTP 协议解析)
    ↓
asyncio 事件循环
    ↓
FastAPI 路由 → 依赖注入 → 业务函数
    ↓
返回响应
```

---

## 六、总结

- **多进程 = OS 层 fork**，由 Uvicorn/Gunicorn 实现，FastAPI 无感知。
- **目的是绕过 GIL**，充分利用多核 CPU。
- **每个进程内仍是 asyncio 单线程事件循环**，负责高并发 I/O。
- **进程间不共享内存**，跨进程状态必须依赖 Redis、DB 等外部组件。
- 经验值：`workers = (2 × CPU核数) + 1`（Gunicorn 官方推荐起点）。
