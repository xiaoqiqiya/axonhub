# AxonHub 后端学习向导（Go）

> 目标：面向不熟悉 Go 的读者，快速理解 AxonHub 后端的结构、请求链路、LLM 转换管道、权限与数据层，并给出可跟读的代码入口。

---

## 1. 后端总体结构（从入口到业务）

### 1.1 入口与依赖注入（fx）
- 入口启动：`server.Run()` 负责组装 fx 依赖并启动 HTTP 服务。
  - 代码位置：[internal/server/server.go:80-117](../../internal/server/server.go#L80-L117)
- fx 模块注册：`dependencies.Module` + `biz.Module` + `api.Module` 等。
- 生命周期：GC worker 在 `fx.Lifecycle` 中启动/停止。

### 1.2 路由与中间件（Gin）
- 路由集中定义于 `SetupRoutes`：
  - 代码位置：[internal/server/routes.go:50-224](../../internal/server/routes.go#L50-L224)
- 中间件链：
  - AccessLog / Ent Client / Logging & Tracing / Metrics
  - 代码入口：[internal/server/routes.go:54-58](../../internal/server/routes.go#L54-L58)

### 1.3 业务服务层（biz）
- 业务服务统一放在 `internal/server/biz/`。
- 对应 API handler 只做“参数解析 + 调用业务服务 + 返回响应”。
- 示例：`AuthService`、`RequestService`、`ChannelService`。

---

## 2. 请求链路：一次 OpenAI Chat 请求的完整流程

以 `/v1/chat/completions` 为例：

### 2.1 路由入口
- 路由注册：
  - [internal/server/routes.go:157-169](../../internal/server/routes.go#L157-L169)
- 目标 handler：`OpenAIHandlers.ChatCompletion`
  - [internal/server/api/openai.go:159-161](../../internal/server/api/openai.go#L159-L161)

### 2.2 认证与上下文
- API Key 认证：`WithAPIKeyAuth`
  - 代码位置：[internal/server/middleware/auth.go:18-58](../../internal/server/middleware/auth.go#L18-L58)
- 认证成功后会把 `APIKey`、`ProjectID` 写入 context。

### 2.3 进入编排器（Orchestrator）
`OpenAIHandlers` 会构造 `ChatCompletionOrchestrator`：
- 代码位置：[internal/server/api/openai.go:55-96](../../internal/server/api/openai.go#L55-L96)
- 依赖注入：ChannelService / ModelService / RequestService / SystemService 等

> 学习提示：编排器是请求进入 LLM pipeline 的入口，负责把“HTTP 请求”变成“统一 LLM 请求”。

---

## 3. LLM 转换管道（Inbound → Unified → Outbound）

### 3.1 管道入口
- 核心处理逻辑在 `llm/pipeline/pipeline.go`：
  - [llm/pipeline/pipeline.go:222-365](../../llm/pipeline/pipeline.go#L222-L365)
- 关键步骤：
  1. Inbound Transformer → `llm.Request`
  2. Middlewares（统一请求前处理）
  3. Outbound Transformer → 上游请求
  4. 执行器发送请求
  5. 响应/流式处理

### 3.2 Inbound / Outbound Transformer 示例（OpenAI）
- Inbound（HTTP → 统一请求）：
  - [llm/transformer/openai/inbound.go:30-76](../../llm/transformer/openai/inbound.go#L30-L76)
- Outbound（统一请求 → 上游）：
  - [llm/transformer/openai/outbound.go:112-177](../../llm/transformer/openai/outbound.go#L112-L177)

### 3.3 Middlewares（管道中间件）
- 定义接口：[llm/pipeline/middleware.go:11-58](../../llm/pipeline/middleware.go#L11-L58)
- 允许在请求/响应/流式阶段拦截和处理。

---

## 4. 渠道与模型选择（ChannelService）

### 4.1 渠道与模型映射
- 入口：`ChannelService.buildChannelWithTransformer`
  - [internal/server/biz/channel_llm.go:151-757](../../internal/server/biz/channel_llm.go#L151-L757)
- 这里负责根据 `channel.Type` 选择不同 Outbound Transformer。

### 4.2 Model 映射规则
- `GetModelEntries` 会统一：
  - SupportedModels
  - ExtraModelPrefix
  - AutoTrimedModelPrefixes
  - ModelMappings
- 代码位置：[internal/server/biz/channel_llm.go:804-907](../../internal/server/biz/channel_llm.go#L804-L907)

---

## 5. 权限与认证体系

### 5.1 JWT 认证（管理端）
- `AuthService.GenerateJWTToken` & `AuthenticateJWTToken`
  - [internal/server/biz/auth.go:83-184](../../internal/server/biz/auth.go#L83-L184)

### 5.2 API Key 认证（API 端）
- `AuthService.AnthenticateAPIKey`
  - [internal/server/biz/auth.go:186-208](../../internal/server/biz/auth.go#L186-L208)
- 中间件：`WithAPIKeyAuth`
  - [internal/server/middleware/auth.go:18-58](../../internal/server/middleware/auth.go#L18-L58)

### 5.3 Ent 权限策略
- 示例：Channel schema policy
  - [internal/ent/schema/channel.go:186-197](../../internal/ent/schema/channel.go#L186-L197)

---

## 6. 数据层（Ent ORM）

### 6.1 Schema 定义
- Channel schema 入口：
  - [internal/ent/schema/channel.go:35-142](../../internal/ent/schema/channel.go#L35-L142)

### 6.2 Request 与执行记录
- RequestService 负责写入请求与执行记录：
  - [internal/server/biz/request.go:111-342](../../internal/server/biz/request.go#L111-L342)
- 支持：
  - DB 或外部存储
  - 流式 chunk 持久化
  - trace 维度查询

---

## 7. GraphQL（管理端）

- Schema：`internal/server/gql/*.graphql`
- Resolvers：`internal/server/gql/*.resolvers.go`
- 路由：
  - `/admin/graphql`
  - `/openapi/v1/graphql`
  - `/agent/v1/graphql`
- 路由定义：
  - [internal/server/routes.go:91-145](../../internal/server/routes.go#L91-L145)

---

## 8. 建议的学习路径（逐步上手）

### Step 1：跑通一次请求
1. 读路由定义：[internal/server/routes.go](../../internal/server/routes.go)
2. 读 handler：`OpenAIHandlers.ChatCompletion`
3. 追踪到 Orchestrator 与 pipeline

### Step 2：理解管道与转换器
1. Inbound / Outbound
2. pipeline 中间件链
3. retry / channel switch 机制

### Step 3：理解权限与数据层
1. AuthService
2. Ent schema + policy
3. RequestService

---

## 9. 常用扩展点

### 9.1 新增渠道
- 更新枚举：`internal/ent/schema/channel.go`
- 注册 transformer：`ChannelService.buildChannelWithTransformer`
- 更新前端配置（不在本文范围）

### 9.2 新增 API
- 路由：`internal/server/routes.go`
- handler：`internal/server/api/*.go`
- biz 服务：`internal/server/biz/*.go`

---

## 10. 常用调试入口

- 认证失败：
  - [internal/server/middleware/auth.go](../../internal/server/middleware/auth.go)
- 请求链路失败：
  - [llm/pipeline/pipeline.go](../../llm/pipeline/pipeline.go)
- 渠道与模型映射：
  - [internal/server/biz/channel_llm.go](../../internal/server/biz/channel_llm.go)

---

## 11. 进一步阅读

- [开发指南](../zh/development/development.md)
- [转换流程架构](../zh/development/transformation-flow.md)
- [渠道配置指南](../zh/guides/channel-management.md)
- [配置指南](../zh/deployment/configuration.md)
