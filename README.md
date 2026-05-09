# AI Knowledge Base V4

AI/LLM/Agent 领域技术情报平台 — 每日自动采集、多 Agent 协作分析、多渠道分发、交互式问答。

## 架构

```
                         ┌──────────────────────────────────────┐
                         │        OpenClaw 消息网关 :3000        │
                         │  ┌─────────┐  ┌───────────────────┐  │
  Telegram / 飞书 ──────▶│  │ 路由匹配 │─▶│ knowledge-query   │  │
                         │  │         │  │ daily-briefing    │  │
                         │  │         │  │ subscription-mgr  │  │
                         │  └─────────┘  └───────────────────┘  │
                         └──────────────┬───────────────────────┘
                                        │ 读 knowledge/articles/
                                        │
  ┌─────────────────────────────────────┼───────────────────────┐
  │                         Stage A: LangGraph 工作流            │
  │                                                              │
  │  plan ──▶ collect ──▶ analyze ──▶ review                    │
  │                                      │                       │
  │                    ┌─────────────────┼─────────────────┐     │
  │                    ▼                 ▼                 ▼     │
  │               organize          revise (循环)     human_flag │
  │              (通过入库)         (未通过,重试)      (超次人工)  │
  │                    │                                         │
  │                    ▼                                         │
  │          knowledge/articles/*.json                           │
  └────────────────────────┬────────────────────────────────────┘
                           │
  ┌────────────────────────▼────────────────────────────────────┐
  │                    Stage B: 分发层                            │
  │                                                              │
  │  formatter.py ──▶ publisher.py ──┬── Telegram 频道           │
  │  (Markdown/飞书/               ├── 飞书群组 Webhook          │
  │   Telegram 格式)               └── 本地 JSON 文件            │
  └─────────────────────────────────────────────────────────────┘
```

## 快速开始

### 1. 环境准备

```bash
cd v4-production
cp .env.example .env
# 编辑 .env，至少填入 LLM_API_KEY、LLM_BASE_URL、LLM_MODEL
pip install -r requirements.txt
```

### 2. 运行流水线（采集 → 分析 → 整理）

```bash
# 精简模式（5 条，约 ¥0.005）
PLANNER_TARGET_COUNT=5 python -m pipeline.pipeline --no-publish

# 标准模式（10 条）
python -m pipeline.pipeline --no-publish
```

### 3. 完整运行（流水线 + 多渠道分发，需配置 Telegram/飞书）

```bash
python -m pipeline.pipeline
```

### 4. 仅发布简报（不运行采集）

```bash
python daily_digest.py
```

### 5. Docker 部署

```bash
cp .env.example .env  # 编辑填写必要配置
./scripts/deploy.sh    # docker compose up -d
```

三个服务自动启动：`pipeline`（Cron 定时采集） + `bot`（交互机器人） + `openclaw`（消息网关）。
