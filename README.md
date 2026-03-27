# AI Shopping Assistant

基于 LangChain 1.x 与 LangGraph 1.x 构建的单入口 ReAct 智能导购 Agent。用户只需输入一句自然语言，Agent 会自主完成意图识别、工具调用、观察压缩、循环控制和结果生成。

## 项目亮点

- 单入口 `/api/v1/chat`：统一接收推荐、对比、详情、价格等问题
- ReAct 执行链：思考、行动、观察、再决策
- 原生 Tool Calling / Function Calling：基于 LangChain 工具绑定执行
- 结构化输出：Pydantic 模型约束意图、决策与 trace steps
- 上下文管理：滑动窗口 + 历史摘要 + 关键状态持久化
- 循环控制：最大步数、重复 thought 检测、无进展提前终止、best-effort 收束
- 观测压缩：按工具类型对 observation 做规范化与摘要
- 可观测性：返回 `trace`、`steps`、`done_reason`、`state_delta`、工具调用统计

## 技术栈

- FastAPI
- LangChain 1.1.3
- LangGraph 1.0.5
- LangChain OpenAI 1.1.3
- SQLAlchemy Async + aiomysql
- Redis（可选缓存，不可用时自动降级）

## 主要能力

1. 商品推荐
2. 商品对比
3. 商品详情查询
4. 价格与价格快照查询
5. 完整 ReAct steps 追踪

## API 说明

### 1. Health Check

`GET /health`

### 2. 单入口 Agent 对话

`POST /api/v1/chat`

请求示例：

```json
{
  "session_id": "s1",
  "user_query": "我想买一台适合办公的轻薄本，预算5000左右"
}
```

响应特点：

- `reply`：面向用户的最终回答
- `trace`：摘要版执行路径与工具统计
- `steps`：完整 ReAct 调试步骤，包含 thought、action、observation、done_reason、state_delta
- `recommendation` / `comparison`：结构化业务结果

### 3. 调试用辅助接口

- `POST /api/v1/recommend`
- `POST /api/v1/compare`

这两个接口主要用于回归测试和单功能验证，项目主体形态仍然是 `/api/v1/chat` 单入口 Agent。

## 快速启动

### 1. 安装依赖

```bash
pip install -r requirements.txt
```

### 2. 配置环境变量

项目需要 MySQL、OpenAI 兼容 LLM API，Redis 为可选。

`app/core/config.py` 会从 `.env` 读取配置。

### 3. 启动 API

```bash
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload
```

### 4. 测试接口

```bash
curl.exe -X POST "http://127.0.0.1:8000/api/v1/chat" -H "Content-Type: application/json" -d "{\"session_id\":\"s1\",\"user_query\":\"我想买一台适合办公的轻薄本，预算5000左右\"}"
```

## 项目结构

```text
app/
  agent/         # LangGraph Agent 主链、prompt、intent、context、termination
  tools/         # LangChain tools：搜索、筛选、推荐、对比、详情、价格
  services/      # 领域服务与 response composer
  api/           # FastAPI 路由与 schema
  db/            # SQLAlchemy 模型与 repository
```

## 适合展示的 Agent 能力

- ReAct Agent 设计与 LangGraph 状态图编排
- Tool Calling / Function Calling
- Pydantic 结构化输出
- 意图识别与任务驱动终局判定
- 上下文管理、observation 压缩、循环控制
- 可观测性与 trace/steps 调试能力

