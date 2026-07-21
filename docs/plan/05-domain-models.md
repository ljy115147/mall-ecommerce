# 领域模型、状态机与核心表

## 1. 限界上下文

| 上下文 | 职责 | 关键聚合 |
|--------|------|----------|
| Identity | 用户账号、登录、地址 | User, Address |
| Merchant | 商家、店铺、入驻 | Merchant, Shop |
| Catalog | 类目、SPU/SKU、上下架 | Category, Product, Sku |
| Cart | 购物车 | Cart, CartItem |
| Trade | 订单履约主状态 | Order, OrderItem |
| Inventory | 可售/占用/已售 | Stock, StockLog |
| Promotion | 券、满减、拼团、秒杀活动 | Coupon, Groupon, SeckillActivity |
| Payment | 支付单、退款单、回调 | Payment, Refund, PayNotifyLog |
| AfterSale | 售后单 | AfterSaleOrder |
| Search | 检索门面 | （无强一致写模型） |
| Recommend | 行为、召回结果 | BehaviorEvent, RecResult |
| Risk | 规则与名单 | RiskRule, Blacklist |
| Knowledge | 客服知识库 | KbDoc, KbChunk |

## 2. 订单状态机

```text
CREATED(待支付)
  ├─ pay_success → PAID
  ├─ user_cancel / timeout → CANCELLED
PAID
  ├─ ship → SHIPPED
  ├─ refund_only → REFUNDING → REFUNDED
SHIPPED
  ├─ confirm / auto_confirm → COMPLETED
  ├─ aftersale → AFTER_SALE_*（与售后单协作）
CANCELLED / REFUNDED / COMPLETED = 终态（视规则）
```

规则：

- 任何流转写 `order_state_log(order_id, from, to, reason, operator, ts)`。
- 非法流转抛领域错误，不静默吞。

## 3. 支付单状态机

```text
INIT → PAYING → SUCCESS
              ↘ FAIL
SUCCESS → REFUNDING → REFUNDED（部分退可扩展 PARTIAL）
```

回调必须：验签 → 幂等键（渠道流水号）→ 更新支付单 → 推订单。

## 4. 拼团状态机

```text
OPEN → SUCCESS（满员）
     → FAILED（超时/主动）
成员：JOINED → 随团成功/失败
```

临界区：最后名额用 DB 条件更新或分布式锁 + 版本号，成团只触发一次 Outbox。

## 5. 售后状态机（仅退款优先）

```text
APPLIED → APPROVED → REFUNDING → DONE
        ↘ REJECTED
```

未发货仅退款：同意后创建退款单，成功后回补库存占用/已售（按支付时是否扣减策略一致）。

## 6. 库存模型

```text
stock(sku_id, sellable, locked, sold, version)
stock_log(id, sku_id, change_type, delta, biz_no, ts)
```

`change_type`：LOCK / UNLOCK / DEDUCT / SECKILL_DECR / RECONCILE_ADJUST…

所有变更带 `biz_no` 保证幂等。

## 7. 核心表清单（逻辑）

### 用户/商家

- `user`, `user_address`
- `merchant`, `shop`, `shop_user_rel`

### 商品

- `category`, `product_spu`, `product_sku`, `product_image`
- `product_status` 或字段：`ON_SALE/OFF_SALE`

### 交易

- `cart_item`
- `order`, `order_item`, `order_state_log`
- `order_fee_snapshot`（应付、优惠、运费、实付等分项）

### 库存/促销/支付

- `stock`, `stock_log`
- `coupon_template`, `coupon_user`
- `promotion_rule`
- `groupon_group`, `groupon_member`
- `seckill_activity`, `seckill_sku`, `seckill_order_queue`
- `payment`, `refund`, `pay_notify_log`

### 基础设施

- `outbox_event`(id, aggregate_type, aggregate_id, topic, key, payload, status, created_at, sent_at)
- `idempotent_record`(ikey, scope, created_at)
- `job_execution_log`

### 搜推/AI

- `behavior_event`
- `item_similarity`(item_a, item_b, score) 或只放离线文件/Redis
- `kb_document`, `kb_chunk`
- `ai_chat_session`, `ai_chat_message`（可选）

### 对账

- `recon_batch`, `recon_diff`

## 8. 金额与时间

- 金额：`BIGINT` 分；汇率不做。
- 时间：统一 UTC 存储或统一业务时区（选一写 ADR），API 返回 ISO8601。

## 9. ID 策略

| 实体 | 策略 |
|------|------|
| 用户/商品 | 号段或雪花 |
| 订单 | **含分片信息的订单号**（分库分表后必须） |
| 支付单 | 雪花；渠道侧另有 transaction_id |
| 幂等键 | 客户端传入或服务端生成 `Idempotent-Key` |

分片后订单号设计见 `deep-dives/sharding-reconciliation.md`。
