# 技术栈与中间件清单（已定）

版本号为建议区间，落地时锁定到具体 patch 并写入各服务 `README`。

## 1. 前端

| 项 | 选型 |
|----|------|
| C 端 | Vue 3.4+ / Vite / TypeScript / Pinia / Vue Router |
| 管理端 / 商家端 | Vue 3 + Element Plus + TypeScript |
| HTTP | Axios；统一错误码与鉴权拦截 |
| 工程 | ESLint + Prettier；环境变量分 env |

## 2. 后端（Java）

| 项 | 选型 |
|----|------|
| 语言 | Java 17 / 21（建议 17 LTS） |
| 框架 | Spring Boot 3.2+ |
| 安全 | Spring Security + JWT |
| ORM | MyBatis-Plus |
| 迁移 | Flyway |
| 文档 | SpringDoc OpenAPI |
| 工具 | Lombok、MapStruct、Hutool（克制使用） |
| 微服务 | Spring Cloud Alibaba：Nacos、Gateway、OpenFeign、Sentinel |
| 分布式事务 | **默认** Outbox + Kafka 最终一致；**局部试点** Seata AT（仅普通下单占用库存，见 ADR-0007） |
| 分片 | ShardingSphere-JDBC |
| 任务 | XXL-JOB |
| 缓存 | Caffeine + Spring Data Redis |
| 消息 | Spring Kafka |

## 3. AI（Python）

| 项 | 选型 |
|----|------|
| 服务 | FastAPI + uvicorn |
| Embedding | BGE-small-zh（本地）或 OpenAI-compatible embeddings |
| LLM | OpenAI-compatible API（云端/本地均可切） |
| 向量 | pymilvus |
| 编排 | 自研轻量 RAG Pipeline（检索 → 组装 Prompt → LLM → 引用） |

## 4. 数据与中间件

| 组件 | 用途 |
|------|------|
| MySQL 8 | 主业务库；后期主从 + 分库分表 |
| Redis 7 | 缓存、库存预减、限流计数、分布式锁、推荐列表 |
| Kafka 3.x（KRaft） | 领域事件、秒杀命令、行为流、商品变更、DLQ |
| Elasticsearch 8 | 商品关键词搜索、运营查询 |
| Milvus 2.x | 商品/知识库向量 |
| Canal | binlog → ES / Kafka |
| MinIO | 图片与静态资源 |
| Seata Server | P6：仅普通下单「订单+库存占用」AT 试点（见 ADR-0007，非全链路） |
| Prometheus + Grafana | 指标 |
| SkyWalking | 链路追踪 |
| Nginx | 静态资源与反向代理 |

可选增强（M15 后按需）：

| 组件 | 用途 |
|------|------|
| Flink | 实时特征 / 行为窗口（推荐实时化） |
| ClickHouse 或 Doris | 行为与对账分析、简易数仓 |
| Apache SeaTunnel / DataX | 同步作业 |
| OpenTelemetry | 与 SkyWalking 二选一做深化时可统一 |

## 5. Kafka Topic 清单（基准）

| Topic | Key | 说明 |
|-------|-----|------|
| `mall.order.events` | orderId | 订单生命周期 |
| `mall.inventory.events` | skuId | 库存流水事件 |
| `mall.seckill.commands` | userId 或 skuId | 秒杀异步下单 |
| `mall.groupon.events` | groupId | 拼团事件 |
| `mall.payment.events` | paymentId | 支付/退款 |
| `mall.behavior.events` | userId | 曝光点击加购下单 |
| `mall.product.changed` | productId | 触发 embedding/缓存 |
| `mall.search.index` | productId | 可选：索引任务 |
| `mall.*.dlq` | 原 key | 死信 |

分区策略：按关键吞吐预估从 3–12 分区起步；秒杀命令 Topic 单独扩大分区。

## 6. Milvus Collection

| Collection | 向量来源 | 用途 |
|------------|----------|------|
| `product_emb` | 标题+卖点+类目文本 | 语义搜索、向量召回 |
| `kb_emb` | FAQ/政策/商品说明切片 | RAG 客服 |
| `user_emb`（可选） | 近行为聚合向量 | 个性化召回进阶 |

## 7. 部署与云原生（已定）

详见 [deep-dives/deployment-docker-k8s.md](./deep-dives/deployment-docker-k8s.md)、[ADR-0008](./adr/0008-deploy-compose-then-k8s.md)。

| 技术 | 用途 | 阶段 |
|------|------|------|
| Docker + BuildKit | 多阶段构建（backend/frontend/admin/ai） | P0 起 |
| Docker Compose | 本地/staging 中间件 + 应用编排 | P0–P5 主力 |
| 镜像仓库 | GHCR / Harbor / Docker Hub（选一写清） | M4 起 |
| Nginx | 静态资源、反向代理、入口限流 | P1 |
| Kubernetes（kind/k3d 或云 K8s） | 微服务 Deployment/Service/Ingress | P6 起 **必做** |
| 探针 + 滚动发布 | liveness/readiness；`rollout undo` 回滚 | P6–P7 |
| HPA | Gateway / 秒杀 Consumer / AI 弹性（演示级） | P6–P7 |
| Kustomize | sit/staging/prod 环境叠加 | P6 |
| Helm | 可选：中间件与业务 chart 打包 | P7 加码 |
| GitHub Actions（或 GitLab CI） | lint → test → build 镜像 → 推送 → 部署 staging | P0 末起 |
| Secret 管理 | 环境变量 / K8s Secret /（可选）密封密钥；禁止进 Git | 全程 |

部署目录约定：`deploy/docker-compose*.yml`、`deploy/docker/Dockerfile.*`、`deploy/k8s/`、`deploy/nginx/`。

## 8. 本地依赖（开发机）

- Docker Desktop / Docker Engine（含 Compose）
- kubectl + kind 或 k3d（P6 前装好即可）
- JDK 17+、Maven 3.9+、Node 20+、pnpm 或 npm
- Python 3.11+、Poetry 或 uv
- JMeter 或 k6（压测）
- 可选：Lens / Kafka UI / Redis Insight / k9s

## 9. 环境分层

| Env | 编排 | 用途 |
|-----|------|------|
| local | Compose | 开发，Mock 支付与 Mock LLM 可开 |
| sit | Compose 或轻量 K8s | 联调，真实中间件，沙箱支付 |
| staging | Compose（前期）→ K8s（P6 后） | 预发、压测、对外演示 |
| prod | K8s（若真实上线） | 密钥与配额独立 |

所有密钥走环境变量或配置中心 / K8s Secret，**禁止进 Git**。
