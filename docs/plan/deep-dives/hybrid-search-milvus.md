# Deep Dive：ES + Milvus 混合检索

## 1. 为什么双引擎

| 引擎 | 擅长 | 不擅长 |
|------|------|--------|
| ES | 关键词、过滤、聚合、可解释 | 同义/意图（「冬天保暖」） |
| Milvus | 语义近邻 | 精确过滤、强业务规则 |

混合：召回并集或双路 TopK → 融合 → 业务过滤（上架、库存、价格）。

## 2. 管道

```text
商品变更 → Kafka → AI Embed → upsert Milvus
         → Canal → upsert ES
查询 → 并行 ES & Milvus → fuse → hydrate 商品卡片
```

## 3. 融合策略（可先简单）

- 分数 min-max 归一化  
- `score = w1 * es + w2 * vec`  
- 过滤无货/下架  
- 可加业务加权：销量、好评（后期）  

## 4. 降级

- Milvus/AI 超时 → 仅 ES  
- ES 失败 → 可选 DB like（极弱）或报错  
- 开关强制降级用于演练  

## 5. 索引与重刷

- Embedding 模型升级 → 全量 re-embed Job  
- ES mapping 变更 → reindex  
- 对账：DB on_sale 集合 vs ES/Milvus 集合差异  

## 6. 可深挖问题

1. IVF_FLAT vs HNSW 的召回/延迟/内存？  
2. 向量维度与距离度量（IP/L2/Cosine）怎么选？  
3. 如何防止「语义相关但类目错误」？  
4. 查询侧缓存是否该做？键是什么？  
5. 多语言/错别字如何处理？
