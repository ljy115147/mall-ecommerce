# Deep Dive：可观测性与 SRE

## 1. 三大支柱

- Metrics：Prometheus  
- Tracing：SkyWalking  
- Logging：JSON + 集中检索（可先文件，后 ELK/Loki）  

统一 `traceId` 贯穿 Gateway → 服务 → Kafka header → 消费者。

## 2. 业务指标（比 CPU 更重要）

- `order_create_total{result}`  
- `payment_success_ratio`  
- `seckill_oversell`（必须为 0）  
- `kafka_consumer_lag{topic,group}`  
- `search_fallback_total`  
- `rec_fallback_total`  
- `rag_no_citation_ratio`  
- `recon_diff_open`  

## 3. 告警分级

| 级 | 例 | 响应 |
|----|----|------|
| P0 | 支付成功率骤降、超卖>0 | 立即止血 |
| P1 | Lag 严重、关单堆积 | 小时级 |
| P2 | 推荐降级频繁 | 日级 |

## 4. Runbook 模板

1. 现象  
2. 影响面  
3. 排查步骤（链接大盘）  
4. 止血动作（开关）  
5. 恢复验证  
6. 复盘待办  

## 5. 混沌价值

面试官听的是：**你怎么发现、怎么降级、怎么防止再发**，不是混沌工具名字。

## 6. 可深挖问题

1. 高基数 label 为什么危险？  
2. 采样率对追踪的影响？  
3. 如何区分「依赖变慢」与「自身 Bug」？  
4. Error Budget 耗尽时发布策略？  
5. 日志隐私合规怎么做？
