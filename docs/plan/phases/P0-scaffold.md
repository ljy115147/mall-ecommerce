# P0 — 中台脚手架（Month 1）

## 目标

搭好后续 23 个月都能复用的工程基座：模块边界、Outbox→Kafka、幂等、缓存、观测、AI 空服务、双前端空壳。

## 前置

- 本机 Docker / JDK17 / Maven / Node20 / Python3.11 可用
- 已阅读 `02-tech-stack.md`、`03-architecture.md`

## 周计划

### Week 1 — 后端多模块 + 规范

- [ ] 创建 `mall-backend` 父 POM 与子模块（common/bootstrap/api-app/api-admin）
- [ ] 统一 `Result`、错误码枚举、`GlobalExceptionHandler`
- [ ] 参数校验（Jakarta Validation）
- [ ] Flyway 接入；示例表 `demo_ping`
- [ ] SpringDoc OpenAPI
- [ ] 日志 JSON pattern + MDC `traceId` 过滤器

### Week 2 — Kafka Outbox + 幂等 + 缓存

- [ ] 表：`outbox_event`、`idempotent_record`
- [ ] Outbox Publisher（事务后或同事务写表）
- [ ] Relay 定时/轮询发送 Kafka（注意并发与状态：NEW/SENT/FAIL）
- [ ] 示例 Consumer + 幂等拦截
- [ ] `@Idempotent` 或拦截器（Header `Idempotent-Key`）
- [ ] `CacheTemplate`：Caffeine + Redis，统一 key 前缀 `mall:`



### Week 3 — Compose 全家桶 + 观测 + 镜像骨架

- [ ] `deploy/docker-compose.yml`：MySQL、Redis、Kafka、MinIO、Prometheus、Grafana、SkyWalking OAP+UI、Kafka UI、XXL-JOB Admin
- [ ] `deploy/docker/Dockerfile.backend`、`Dockerfile.ai` 多阶段骨架（可先跑通）
- [ ] Spring Boot Actuator + Micrometer 暴露
- [ ] Grafana 导入 JVM/Kafka 基础看板
- [ ] SkyWalking agent 本地启动说明

### Week 4 — 前端空壳 + mall-ai + 文档 + CI

- [ ] `mall-frontend` / `mall-admin` Vite+Vue3+TS 路由与布局
- [ ] Axios 封装
- [ ] `mall-ai`：`GET /health`，`POST /embeddings`（mock 固定维度向量）
- [ ] 根 README 启动步骤（含 Compose）
- [ ] ADR-0001 Kafka；ADR-0002 Outbox；ADR-0008 部署演进
- [ ] CI：PR 编译 + 单测
- [ ] 演示录屏 3 分钟



## 交付物


| 交付      | 路径建议                          |
| ------- | ----------------------------- |
| Compose | `deploy/docker-compose.yml`   |
| 后端可运行   | `mall-backend`                |
| AI      | `mall-ai`                     |
| ADR     | `docs/plan/adr/`              |
| 验收勾选    | `docs/plan/checklists/M01.md` |




## 面试可讲难点（P0 就埋钩子）

1. 为什么 Outbox 而不是先发 Kafka 再写库？
2. Relay 怎么保证不丢不重？（至少一次 + 幂等）
3. traceId 如何跨线程/跨异步传递？
4. 本地缓存与 Redis 如何避免惊群？



## 完成定义（DoD）

- `docker compose up` 后健康检查通过  
- 调用示例 API 能产生 Outbox → Kafka → Consumer 日志  
- Grafana 能看到服务 up 与消息指标  
- 新人按 README 30 分钟内启动成功

