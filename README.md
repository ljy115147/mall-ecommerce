# Mall Ecommerce — 拼多多社招向迷你电商中台

面向 **拼多多社招** 的前后端分离电商平台：从可上线模块化单体演进到 Spring Cloud 微服务，覆盖交易中台、大促秒杀、拼团、Kafka 事件驱动、混合检索（ES + Milvus）、推荐与 RAG 智能客服等。

## 文档入口（先读这里）

| 文档 | 说明 |
|------|------|
| [docs/plan/00-README.md](docs/plan/00-README.md) | 计划总入口：如何按 24 个月执行 |
| [docs/plan/04-roadmap-24-months.md](docs/plan/04-roadmap-24-months.md) | **逐月排期**（主执行表） |
| [docs/plan/03-architecture.md](docs/plan/03-architecture.md) | 目标架构与演进 |
| [docs/plan/deep-dives/deployment-docker-k8s.md](docs/plan/deep-dives/deployment-docker-k8s.md) | Docker / K8s / CI/CD 部署专项 |
| [docs/plan/interview/hard-questions.md](docs/plan/interview/hard-questions.md) | 面试难点题库 |

## 仓库规划（后续实现时创建）

```
mall-ecommerce/
├── mall-frontend/          # C 端 Vue3
├── mall-admin/             # 管理端 / 商家端 Vue3
├── mall-backend/           # Java 模块化单体 → 微服务
├── mall-ai/                # Embedding / RAG / LLM 网关
├── deploy/                 # Docker Compose / K8s
└── docs/                   # 本计划与设计文档
```

## 当前状态

目前仅落地 **2 年详细执行计划与设计文档**。代码从 `docs/plan/phases/P0-scaffold.md` 开始实现。
