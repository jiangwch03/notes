# notes

个人学习笔记仓库，按主题分类存放，涵盖 AI 大模型、Java、Python、数据库、中间件、数据结构与算法、网络与架构等多个方向。

## 目录结构

```
notes/
├── AI/                        # 大模型与 AI 应用
│   ├── 01 模型基座相关/       # LLM 简介、原理入门、项目研发流程
│   ├── 02 prompt/             # 提示词工程、Markdown 语法笔记
│   ├── 03 agent/              # Agent、Haeness 工程、MCP、Skill、上下文工程、长期记忆
│   ├── 04 Rag/                # RAG 流程、Embeddings、Milvus
│   └── 05 可观测性和rag测评/  # Langfuse、Ragas
├── java/                      # Java 集合、JVM（上/中/下篇课件与思维导图）、并发面试
├── python/                    # Python 基础、FastAPI（含多进程管理原理）
├── 数据库/                    # MySQL（课件+笔记）、分库分表、Seata 分布式事务
├── 中间件/                    # Nginx、Kafka、Redis
├── 数据结构/                  # 数据结构与算法总结、LeetCode 重点题
├── 网络相关/                  # 网络基础、WebSocket
├── 架构/                      # 并发数 & RT & 吞吐
├── knowledge项目/             # MinerU、crawl4ai、OpenSpec 和 Superpowers
├── README.md
└── github使用.mmap
```

## 内容概览

| 模块 | 主要内容 |
|------|----------|
| AI / 大模型 | LLM 原理与研发流程、提示词工程、Agent（MCP / Skill / 上下文与长期记忆）、RAG（Embeddings / Milvus）、可观测性与评测（Langfuse / Ragas）、LangChain / LangGraph 笔记 |
| Java | Java 集合、JVM 内存与垃圾回收、字节码与类加载、性能监控与调优、并发面试总结 |
| Python | 基础语法、PIP 命令、import 机制、FastAPI 及多进程管理 |
| 数据库 | MySQL 课件笔记、主从同步、ShardingSphere 分片与读写分离、Seata 分布式事务 |
| 中间件 | Nginx 基础与高并发架构、Kafka 重点、Redis 重点 |
| 数据结构与算法 | 数据结构总结、重点算法思考、LeetCode、算法工具方法 |
| 网络与架构 | 网络基础、WebSocket、并发数 & RT & 吞吐 |
| 知识项目 | MinerU、crawl4ai、OpenSpec 与 Superpowers 学习记录 |

## Markdown 笔记

可直接阅读的 Markdown 文件：

- [Markdown 语法笔记](AI/02%20prompt/markdown.md)
- [PIP 命令](python/PIP命令.md)
- [Python import 机制](python/python-import-essence.md)
- [FastAPI 多进程管理原理](python/python_fastapi_多进程管理原理.md)

## 文件格式说明

- `.mmap` 为 MindManager 思维导图文件，可用 [MindManager](https://www.mindmanager.com/) 打开
- `.doc` / `.docx` 可用 Word / WPS 打开
- `.pdf` / `.pptx` / `.txt` / `.png` 为课件、图片或文本笔记

## 说明

笔记会持续补充，内容以个人理解为主，如有疏漏欢迎指正。
