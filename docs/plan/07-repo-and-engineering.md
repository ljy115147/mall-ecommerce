# 仓库结构与工程规范

## 1. 推荐仓库布局

```text
mall-ecommerce/
├── README.md
├── docs/                      # 计划与设计（已存在）
├── mall-backend/              # Java
├── mall-frontend/             # C 端
├── mall-admin/                # 运营 + 商家（可用 monorepo 分包）
├── mall-ai/                   # FastAPI
├── deploy/
│   ├── docker-compose.yml
│   ├── docker-compose.staging.yml
│   ├── docker/                 # Dockerfile.*
│   ├── nginx/
│   ├── prometheus/
│   ├── grafana/
│   ├── kafka/
│   ├── perf/
│   ├── k8s/                    # base + overlays（P6）
│   └── helm/                   # 可选（P7）
└── scripts/
    ├── seed.sh
    ├── rebuild-es.sh
    └── reembed-products.sh
```

## 2. Git 规范

- 分支：`main`、`develop`、`feature/*`、`fix/*`、`release/*`
- Commit：约定式 `feat|fix|docs|refactor|perf|test|chore`
- PR：必须通过编译 + 单测；大功能附 ADR 链接
- Tag：`vX.Y.Z`；面试冻结 `v1.0-interview`

## 3. 代码规范

- Java：阿里巴巴规约精简版；领域层不依赖 Web
- API 响应：`{ code, message, data, traceId }`
- 异常：业务异常 vs 系统异常；不把堆栈返回给 C 端
- 禁止 `double` 算钱；禁止吞异常
- 所有外部调用（Kafka/HTTP/LLM）设超时

## 4. 测试金字塔

| 层级 | 要求 |
|------|------|
| 单测 | 状态机、算价、库存条件更新、幂等 |
| 集成测 | Testcontainers（MySQL/Redis/Kafka）核心链路 |
| 契约测 | 微服务阶段 Feign API |
| 压测 | 秒杀/下单/搜索脚本入库 |
| 安全测 | 越权用例集 |

## 5. CI/CD 流水线（建议 GitHub Actions）

**P0–P1**

1. lint + unit test  
2. 多阶段 build 镜像（Buildx）  
3. 推送 Registry（tag=`gitSha`）  

**P6+**

4. 部署 staging（kubectl/kustomize）  
5. smoke test（健康检查 + 下单/搜索探针）  
6. 失败自动停；回滚文档化  

密钥用 CI Secrets / OIDC；kubeconfig 不进仓库。详见 [deep-dives/deployment-docker-k8s.md](./deep-dives/deployment-docker-k8s.md)。

## 6. 配置与密钥

- `application-local.yml` 可进库（无密钥）
- 密钥：`.env`（gitignore）/ Nacos 加密 / K8s Secret
- LLM API Key、支付密钥、JWT secret 分离

## 7. 文档同步义务

每完成一个阶段：

1. 更新 `phases/Px` Checklist 状态  
2. 新增/更新 ADR  
3. 在 `interview/stories.md` 追加故事草稿  
4. 路线图当月勾选
