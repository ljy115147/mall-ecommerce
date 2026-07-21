# Deep Dive：拼团一致性

## 1. 业务规则（建议定稿）

- 开团人创建 `group`，状态 OPEN，目标人数 N  
- 参团占用名额；可要求预支付（含金量高）  
- 满 N → SUCCESS，只触发一次成团事件  
- 超时 → FAILED，未成团退款、释放库存  

## 2. 名额扣减

```sql
UPDATE groupon_group SET joined = joined + 1, version = version + 1
WHERE id = #{id} AND status = 'OPEN' AND joined < target
```

若 `joined+1 == target`，同事务将状态置 SUCCESS 并写 Outbox `GroupSucceeded`。  
用 CAS 保证成团事件不双发。

## 3. 支付与成团顺序（难点）

推荐叙事（预支付）：

1. 参团创建「团订单」待支付或支付中  
2. 支付成功才算有效成员  
3. 统计有效成员数达 N 成团  
4. 失败团批量退款  

若「先占座后支付」：需处理占座超时。

## 4. 并发最后一名

压测：剩余 1 名，100 并发。期望：仅 1 成功，其余明确失败码。  
检查：`GroupSucceeded` 消息仅 1 条（Outbox 唯一）。

## 5. 失败退款

- 团 FAILED → 查所有已支付成员 → 创建退款单（幂等：groupId+userId）  
- 退款成功后库存回补  

## 6. 可深挖问题

1. 成团消息重复消费怎么避免二次发货？  
2. 开团人取消如何处理已支付成员？  
3. 跨天活动库存与团生命周期？  
4. 和普通优惠券叠加规则？
