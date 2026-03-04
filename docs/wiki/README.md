# AxonHub Repo Wiki

## 1. 项目概览

AxonHub 是一个 all-in-one AI 开发平台，提供统一的 API 网关与双向数据转换管道，使客户端无需更换 SDK 即可接入多家 AI 提供商。

核心能力概述：
- 统一 API 层：兼容 OpenAI / Anthropic 等接口格式
- 转换管道：请求与响应的双向标准化与适配
- 渠道管理：多提供商与负载均衡
- 追踪与可观测性：线程级请求链路
- 权限系统：基于角色的细粒度访问控制

参考：
- [中文 README](../../README.zh-CN.md)
- [英文 README](../../README.md)

## 2. 架构与数据流

AxonHub 采用双向转换管道进行请求处理：
- 入站处理：解析、验证、标准化
- 核心处理：渠道选择、负载均衡、故障转移
- 出站处理：提供商适配、协议映射、格式转换

参考：
- [转换流程架构](../zh/development/transformation-flow.md)
- [开发指南](../zh/development/development.md)

## 3. 目录结构（核心路径）

### 后端
- [cmd/axonhub/main.go](../../cmd/axonhub/main.go)：应用入口
- [internal/server/](../../internal/server/)：HTTP 服务器与路由
- [internal/server/biz/](../../internal/server/biz/)：业务逻辑
- [internal/server/api/](../../internal/server/api/)：REST/GraphQL API
- [internal/llm/](../../internal/llm/)：模型适配与转换
- [internal/ent/](../../internal/ent/)：Ent ORM 与数据层

### 前端
- [frontend/src/](../../frontend/src/)：前端代码
- [frontend/src/features/](../../frontend/src/features/)：功能模块
- [frontend/src/routes/](../../frontend/src/routes/)：路由
- [frontend/src/locales/](../../frontend/src/locales/)：i18n

## 4. 快速开始

### Docker Compose（推荐）
参考：
- [快速入门](../zh/getting-started/quick-start.md)

### 二进制下载
参考：
- [中文 README](../../README.zh-CN.md)

## 5. 开发指南

### 环境要求
- Go 1.24+
- Node.js 18+
- pnpm

### 启动后端
```bash
make build-backend
./axonhub

# 或使用 air 热重载
air
```

### 启动前端
```bash
cd frontend
pnpm install
pnpm dev
```

参考：
- [开发指南](../zh/development/development.md)

## 6. 配置与部署

### 配置
- 使用 `config.yml` 与环境变量
- 配置优先级：环境变量 > 配置文件 > 默认值

参考：
- [配置指南](../zh/deployment/configuration.md)

### 部署
- Docker Compose
- Helm / Kubernetes
- 虚拟机部署

参考：
- [部署配置说明](../zh/deployment/configuration.md)

## 7. API 与系统管理

### API 参考
- [OpenAI API](../zh/api-reference/openai-api.md)
- [Anthropic API](../zh/api-reference/anthropic-api.md)
- [Gemini API](../zh/api-reference/gemini-api.md)
- [Embedding API](../zh/api-reference/embedding-api.md)
- [Image Generation](../zh/api-reference/image-generation.md)
- [Rerank API](../zh/api-reference/rerank-api.md)
- [Search API](../zh/api-reference/search-api.md)

### 系统与权限
- [权限指南](../zh/guides/permissions.md)
- [追踪指南](../zh/guides/tracing.md)

## 8. 渠道与模型管理

- [渠道配置指南](../zh/guides/channel-management.md)
- [模型管理指南](../zh/guides/model-management.md)
- [API Key Profiles](../zh/guides/api-key-profiles.md)
- [请求覆盖](../zh/guides/request-override.md)
- [负载均衡](../zh/guides/load-balance.md)

## 9. 测试与质量

### 后端测试
```bash
go test ./...
```

### E2E 测试
```bash
bash ./scripts/e2e/e2e-test.sh
```

参考：
- [开发指南 - 测试与质量](../zh/development/development.md)

## 10. 常见问题与排查

- [FAQ](../zh/faq/faq.md)
- [快速入门 - 故障排除](../zh/getting-started/quick-start.md)

## 11. 相关文档导航

- [开发文档目录](../zh/development/development.md)
- [部署文档目录](../zh/deployment/configuration.md)
- [指南目录](../zh/guides/)
- [API 参考目录](../zh/api-reference/)
