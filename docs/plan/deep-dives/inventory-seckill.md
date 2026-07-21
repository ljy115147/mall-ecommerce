# Deep Dive：库存中心与秒杀防超卖

## 1. 库存模型

```text
sellable  可售
locked    下单占用未支付/未确认
sold      已确认销售
```

不变式：`sellable + locked + sold = total`（若有 total）或至少 `sellable/locked/sold >= 0`。

所有变更写 `stock_log`，带 `biz_no` 唯一。

## 2. 普通下单防超卖

```sql
UPDATE stock SET sellable = sellable - #{n}, locked = locked + #{n}, version = version + 1
WHERE sku_id = #{id} AND sellable >= #{n}
```

影响行数=0 → 库存不足。比 `select for update` 更易扩展（仍要注意热点 SKU）。

## 3. 秒杀路径

```text
预热：total → 分桶 Redis
请求：限流 → 风控 → 限购 → DECR 桶 → 成功则发 Kafka
消费者：创建订单 + DB 流水（可能异步把 locked/sold 对齐）
售罄：所有桶 ≤0 且无释放
```

分桶目的：降低单 key 热点；接受轻微的「局部售罄但其它桶仍有」——用随机桶或双检。

## 4. Redis 成功、下游失败

- 排队单状态：ACCEPTED → ORDER_CREATED / ORDER_FAILED  
- FAILED 可触发 Redis 回补（Lua INCR）需幂等  
- 对账 Job：Redis 剩余 + 已售 vs 活动总量  

## 5. 与支付时点

两种策略（选一写 ADR）：

| 策略 | 说明 |
|------|------|
| A 下单占用 | 体验好；关单释放；本项目默认 |
| B 支付扣减 | 超卖窗口不同；恶意占库存少 |

秒杀常走「先扣 Redis，后建单」，更接近 B 的强占用。

## 6. 压测断言 SQL 思路

- 活动销量 ≤ 配置库存  
- 无负库存  
- 同一用户限购不超过 N  
- 订单与流水 biz_no 一一对应  

## 7. 可深挖问题

1. Lua 与管道区别？为何扣库存常用 Lua？  
2. 热点 key 还有哪些解法（本地缓存、多级、副本）？  
3. 如何发现「超卖 1 件」的线上问题？  
4. 分桶导致少卖怎么办？  
5. 时钟回拨对活动开始时间的影响？
