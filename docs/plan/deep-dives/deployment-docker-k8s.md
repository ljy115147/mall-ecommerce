# Deep Dive：部署体系（Docker / K8s / CI/CD / 发布）

## 1. 演进策略（已定）

```text
M1–M4    Docker Compose：中间件 + 应用本地/演示一键起
M4       应用全部容器化（多阶段构建）；Nginx + Compose staging
M8+      镜像规范、健康检查、资源限制成习惯
M18–M21  Kubernetes 演示/预发集群：微服务工作负载
M21–M22  HPA、滚动发布、探针、ConfigMap/Secret、Ingress
M22–M24  可选：Helm、GitOps 雏形、全链路压测流量隔离
```

原则：**先 Docker 标准化镜像与编排，再上 K8s**；避免一上来连业务都没有就折腾集群。

## 2. 技术清单

| 技术 | 用途 | 阶段 |
|------|------|------|
| Docker / BuildKit | 多阶段构建 Java/Node/Python 镜像 | P0 起 |
| Docker Compose | 本地与 staging 中间件+应用编排 | P0–P5 主力 |
| 镜像仓库 | Harbor 自建或 GHCR/Docker Hub | M4 起 |
| Nginx | 静态资源、反向代理、限流入口 | P1 |
| Kubernetes | Deployment/StatefulSet/Service/Ingress | P6 |
| metrics-server + HPA | AI、秒杀 Consumer、Gateway 弹性 | P6–P7 |
| Helm（可选） | 打包中间件与业务 chart | P7 加码 |
| GitHub Actions / GitLab CI | build → test → 推镜像 → deploy | P0 末起逐步完善 |
| Kustomize | 多环境叠加（sit/staging/prod） | P6 |
| cert-manager（可选） | TLS | 有域名时 |
| Ingress Controller（Nginx/Traefik） | 七层入口 | P6 |

中间件在 K8s 上的策略（已定）：

- **演示/学习**：Kafka、MySQL、Redis、Milvus、ES 可用官方 Operator 或 Bitnami Chart，**单副本可接受**。  
- **叙事上要分清**：生产级中间件常有云托管；面试讲「业务工作负载上 K8s，有状态组件可托管/独立」。

## 3. 仓库目录约定

```text
deploy/
  docker-compose.yml              # 本地全家桶
  docker-compose.staging.yml      # 演示叠加
  docker/
    Dockerfile.backend
    Dockerfile.frontend
    Dockerfile.admin
    Dockerfile.ai
  k8s/
    base/                         # Deployment/Service 通用
    overlays/sit/
    overlays/staging/
    ingress.yaml
    hpa.yaml
    seata.yaml                    # 若试点
  helm/                           # 可选
  nginx/
  prometheus/
  grafana/
  perf/
scripts/
  build-images.sh
  deploy-k8s.sh
  smoke-test.sh
```

## 4. 镜像规范（面试常问）

- 多阶段构建：Maven/Node 构建阶段与运行阶段分离  
- 非 root 用户运行  
- 只带 JRE/必要依赖；`.dockerignore` 排除无用文件  
- 标签：`gitSha` + `semver`；禁止长期只依赖 `latest`  
- 健康检查：`/actuator/health` 或自定义 `/health`  
- 资源：Compose/`requests`/`limits` 都要写（哪怕演示环境数值很小）

## 5. Kubernetes 对象最小集（业务服务）

每个微服务至少具备：

1. `Deployment`（副本、滚动策略 `maxUnavailable`/`maxSurge`）  
2. `Service`（ClusterIP）  
3. `liveness` + `readiness` 探针（区分「进程活着」与「能接流量」）  
4. `ConfigMap`（非密钥配置）+ `Secret`（JWT/支付/LLM Key）  
5. 可选 `HPA`（CPU/自定义指标后期）  
6. `Ingress` 路由到 Gateway 或 Nginx  

有状态：MySQL/Kafka 优先 StatefulSet 或外部托管；**不要把业务 Deployment 当数据库用。**

## 6. 发布与回滚

| 策略 | 本项目落地 |
|------|------------|
| 滚动发布 | K8s 默认；先 staging 验证 |
| 灰度 | Gateway 按 header/`userId` hash（应用层）+ 可选双 Deployment |
| 回滚 | `kubectl rollout undo`；镜像 tag 可回退 |
| 数据库 | Flyway **向前兼容**；禁止发布依赖「先改代码再改不兼容库」而无回滚方案 |

大促前：固定版本、关自动 HPA 激进扩缩或预设副本、开启限流。

## 7. CI/CD 流水线（目标形态）

```text
push/PR
  → lint + unit test
  → build images (Buildx)
  → push registry
  → (staging) kubectl/helm apply
  → smoke test（下单/搜索健康）
  → 人工确认后（可选）升预发标签
```

密钥：CI 用 OIDC/Registry Token；**绝不把 kubeconfig 明文进仓库**。

## 8. 可观测与部署的衔接

- 容器日志 stdout → 可接 Loki/EFK（P7 加码）  
- Prometheus 抓取注解或 ServiceMonitor  
- SkyWalking agent 以 sidecar 或 Java agent 环境变量注入  

## 9. 面试可深挖问题

1. 为什么 readiness 和 liveness 要分开？探活失败会怎样？  
2. 滚动发布时旧副本没退完，如何保证兼容？  
3. 有状态中间件上 K8s 的风险？  
4. HPA 根据 CPU 扩秒杀服务够不够？该看什么指标？  
5. Compose 与 K8s 各自适合什么阶段？  
6. 镜像里打 JDK 全家桶有什么问题？  
7. 配置是打进镜像还是运行时注入？为什么？  
8. 如何做「一键回滚」同时避免 DB 不兼容？

## 10. 与业务阶段的绑定验收

| 里程碑 | 部署验收 |
|--------|----------|
| M1 | Compose 起中间件 |
| M4 | 前后端+后端镜像 Compose 演示通过 |
| M8 | 压测环境容器化、资源限制生效 |
| M21 | 核心微服务服务跑在 K8s；Ingress 可访问；HPA 或手动扩缩演示 |
| M22 | 滚动发布 + 回滚演练录屏 |
| M24 | `scripts/deploy-*.sh` + 文档陌生人可部署 |
