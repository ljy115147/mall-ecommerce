# 目标架构与演进

## 1. 演进三阶段

```text
阶段 A（M1–M14）  模块化单体 + 共享中间件
        │  领域包清晰、Outbox→Kafka、可上线
        ▼
阶段 B（M15–M17） 数据面强化
        │  分库分表、读写分离、对账、数仓雏形
        ▼
阶段 C（M18–M21） 微服务 + 治理
        │  独立库、独立扩缩、限流熔断灰度
        ▼
阶段 D（M22–M24） 稳定性与面试硬化
           全链路压测、混沌、材料包
```

## 2. 逻辑架构（终态）

```text
                    ┌─────────────┐  ┌─────────────┐
                    │  Vue C 端   │  │ Admin/商家  │
                    └──────┬──────┘  └──────┬──────┘
                           └────────┬───────┘
                                    ▼
                            Nginx / Gateway
                                    │
        ┌───────────┬───────────┬───┴───┬───────────┬───────────┐
        ▼           ▼           ▼       ▼           ▼           ▼
     UserSvc   ProductSvc   TradeSvc  InvSvc   PromoSvc    PaySvc
        │           │           │       │         │           │
        │           │           └───────┼─────────┘           │
        │           │                   ▼                     │
        │           │              Kafka Cluster              │
        │           │                   │                     │
        │           ▼                   ▼                     │
        │      AfterSale          Recommend / Risk            │
        │           │                   │                     │
        └───────────┴───────────────────┼─────────────────────┘
                                        ▼
                              AI Service (Embedding/RAG)
                                        │
     MySQL(分片) Redis  ES  Milvus  MinIO  XXL-JOB  Obs Stack
```

## 3. 模块化单体内部包结构（阶段 A）

```text
mall-backend/
  mall-bootstrap/          # 启动模块
  mall-common/             # 错误码、幂等、Outbox、缓存、锁
  mall-user/
  mall-merchant/           # 商家店铺
  mall-product/
  mall-cart/
  mall-trade/              # 订单
  mall-inventory/
  mall-promotion/          # 券、满减、拼团、秒杀配置
  mall-payment/
  mall-aftersale/
  mall-recommend/
  mall-risk/
  mall-search/             # 检索门面（调 ES/Milvus/AI）
  mall-job/                # 与 XXL-JOB 对接
  mall-api-admin/
  mall-api-app/
```

依赖规则：

- `trade` 可依赖 `inventory`、`promotion`、`payment` API（接口层），禁止反向依赖。
- `recommend` / `search` **不得**同步强依赖 `trade` 写路径。
- 跨模块通信优先：同进程接口 → Outbox/Kafka 事件；禁止直接写他模块表。

## 4. 关键链路简图

### 4.1 普通下单

```text
下单请求 → 鉴权 → 风控 → 算价 → 库存占用 → 写订单+支付单+Outbox
         → 提交事务 → Relay 发 Kafka → 返回 orderId
关单：Job 扫 pay_deadline → Kafka close → 幂等关单 → 释放占用
```

### 4.2 秒杀

```text
请求 → 限流 → 风控 → Redis 分桶 DECR → 写排队单/Outbox
     → Kafka seckill.commands → 消费者创建订单/落库存流水
     → 客户端轮询结果
```

### 4.3 混合搜索

```text
Query → 并行：ES 关键词 / AI Embed + Milvus TopK
      → 融合打分 → 填商品卡片（Redis/DB）
      → Milvus/AI 失败则仅 ES
```

### 4.4 RAG 客服

```text
问题 → Embed → Milvus kb_emb → 拼 Prompt(+引用) → LLM → 返回答案+citations
```

## 5. 数据架构原则

1. **金额一律分（Long）**，展示层再格式化。  
2. **状态机变更必须校验合法边**，并写状态流转日志。  
3. **费用明细下单时快照**，事后改促销不影响历史订单。  
4. **消息 at-least-once + 业务幂等**；不幻想 Kafka 恰好一次拯救一切。  
5. **每个服务/模块独立表前缀**；微服务阶段独立 Schema/库。  
6. **缓存有策略**：过期、空值、击穿（互斥/逻辑过期任选并写 ADR）。

## 6. 多级缓存

```text
请求 → Caffeine → Redis → DB
写路径：更新 DB → 删 Redis → 发 product.changed → 本地缓存短期版本号/延迟双删
```

秒杀读路径可加「活动静态配置本地缓存」，与库存扣减路径隔离。

## 7. 安全架构

- C 端 JWT；管理端 JWT + RBAC 菜单权限。  
- 商家数据隔离：所有商家 API 强制 `merchant_id` 来自登录态，禁止信任前端传商户 ID。  
- 支付回调验签 + IP 白名单（可配置）。  
- RAG/LLM：敏感信息脱敏；Prompt 注入基础防护（指令隔离）。  

## 8. 部署拓扑

### 8.1 阶段 A/B（模块化单体，M1–M17）

```text
开发机 / staging VM
  Docker Compose
    ├── mysql / redis / kafka / es / milvus / minio / ...
    ├── mall-backend
    ├── mall-ai
    ├── mall-frontend / mall-admin (或 Nginx 托管静态)
    └── nginx → 反代 API + 静态
```

### 8.2 阶段 C（微服务，M18–M24）

```text
                   Internet
                       │
                   Ingress / Nginx
                       │
                   Gateway (Pod)
          ┌────────────┼────────────┐
          ▼            ▼            ▼
     Trade Deploy  Product Deploy  AI Deploy ...
          │            │            │
     Service/DNS   Service/DNS   Service/DNS
          └────────────┼────────────┘
                       ▼
         中间件（集群内 或 旁路 Compose/托管）
         MySQL / Redis / Kafka / ES / Milvus / Nacos / Seata
```

能力要求：多阶段镜像、探针、滚动发布与回滚、ConfigMap/Secret、HPA（关键服务）、CI 构建推送。  
细节见 [deep-dives/deployment-docker-k8s.md](./deep-dives/deployment-docker-k8s.md)。
