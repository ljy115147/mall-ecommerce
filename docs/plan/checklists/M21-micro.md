# Month 21 验收清单（微服务 + K8s）

- [ ] 核心服务均独立库独立部署
- [ ] Gateway 路由与鉴权正确
- [ ] 下单/支付/秒杀/拼团/搜索/推荐在微服务下通过
- [ ] 停止 AI/Recommend（含杀 Pod）后下单仍成功（录屏）
- [ ] Sentinel 规则对秒杀/LLM 生效
- [ ] 灰度策略可演示
- [ ] K8s：Deployment/Service/Ingress/探针/Secret 齐全
- [ ] 滚动发布与 `rollout undo` 演练通过
- [ ] CI：镜像构建 → staging 部署 → smoke test
- [ ] 拆分前后对比文档完成
