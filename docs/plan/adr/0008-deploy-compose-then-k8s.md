# ADR-0008：部署演进 —— 先 Docker Compose，再 Kubernetes

- 状态：Accepted
- 日期：2026-07-16

## 背景

需要可演示、可压测、可讲清的云原生部署能力（Docker / K8s / CI/CD），同时不能让部署体系拖垮前两年业务交付。

## 决策

1. **P0–P5**：以 **Docker + Compose** 为本地/staging 标准交付形态；所有应用多阶段构建镜像。  
2. **P6–P7**：将业务微服务迁到 **Kubernetes**（演示或单机 kind/k3d/云厂商均可）；具备 Deployment、探针、Ingress、Secret、滚动发布与回滚。  
3. 中间件：Compose 阶段全家桶；K8s 阶段允许「业务在集群、中间件 Compose/托管并存」，面试讲清取舍。  
4. CI：GitHub Actions（或同等）负责构建推送镜像；部署 staging 自动化，生产级需人工门禁。

## 备选方案

1. **第一天就上 K8s**：运维噪声大，早期收益低 → 拒绝作为默认。  
2. **只 Compose 不上 K8s**：省事但缺少弹性/滚动/探针等面试与工程深度 → 2 年周期下拒绝。  
3. **Serverless 为主**：与 Java 微服务叙事不完全契合 → 不做主路径。

## 后果

- 前期交付快；后期有完整容器编排故事。  
- 需维护两套编排（Compose 与 K8s），用同一 Dockerfile 降低分叉。

## 验证方式

M4 Compose 演示；M21 K8s 烟雾测试；M22 滚动发布回滚演练。
