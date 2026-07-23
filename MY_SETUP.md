# wjk-dot 的 daily_stock_analysis 配置说明

Fork 自 [ZhuLinsen/daily_stock_analysis](https://github.com/ZhuLinsen/daily_stock_analysis)。

## ✅ 第一次配置（5 分钟）

### 1) 开启 Actions

GitHub 默认禁用 fork 仓库的 Actions。

进 https://github.com/wjk-dot/daily_stock_analysis/actions
→ 顶部黄色横幅点 `I understand my workflows, go ahead and enable them`

### 2) 填 Secrets

路径：https://github.com/wjk-dot/daily_stock_analysis/settings/secrets/actions → `New repository secret`

#### 必填（4 个）

| Name | Value |
|---|---|
| `OPENAI_API_KEY` | `sk-cp-...agP4`（hermes 同款 MiniMax key） |
| `OPENAI_BASE_URL` | `https://api.minimaxi.com/v1` |
| `OPENAI_MODEL` | `MiniMax-M2.5` |
| `STOCK_LIST` | `600519,000001,002415`（自选股，逗号分隔） |

#### 强烈推荐（1 个 — 让结果推送到飞书）

| Name | Value |
|---|---|
| `FEISHU_WEBHOOK_URL` | 飞书群机器人的 webhook URL（见下文） |

### 3) 找飞书群机器人 webhook URL

飞书群 → 设置 → 群机器人 → 添加机器人 → **自定义机器人**

- 名称：`股票分析`
- 安全设置：勾"签名校验"（更安全）或"自定义关键词"（关键词填 `股票`，否则 MiniMax 推送会失败）

加完后 webhook 长这样：
```
https://open.feishu.cn/open-apis/bot/v2/hook/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

整条 URL 复制到 `FEISHU_WEBHOOK_URL` secret。

### 4) 第一次手动跑（不要等定时）

https://github.com/wjk-dot/daily_stock_analysis/actions
→ 左侧选 `00 Daily Analysis`
→ 右侧 `Run workflow` → 选 `main` → 绿色按钮

跑 3-5 分钟，看：
- 绿色 ✓ = 成功，看飞书有没有收到报告
- 红色 ✗ = 看日志找错（最常见：OPENAI_API_KEY 没填对 / STOCK_LIST 格式错）

## ✅ 后续维护

### 修改自选股

`STOCK_LIST` 是 secret，每次改都要进 Settings。也可以改成 **Repository Variables**（`vars.STOCK_LIST`）让浏览器直接改：

路径：https://github.com/wjk-dot/daily_stock_analysis/settings/variables/actions → `New repository variable`

```yaml
# 00-daily-analysis.yml 里的读取顺序（已经支持）：
STOCK_LIST_CONFIG: ${{ vars.STOCK_LIST || secrets.STOCK_LIST }}
```

意思是：优先读 vars，没有就 fall back 到 secret。**推荐用 vars**，改起来方便还不算"敏感配置"。

### 改定时时间

`.github/workflows/00-daily-analysis.yml` 里已经有：
```yaml
schedule:
  - cron: '0 10 * * 1-5'     # 北京时间 18:00，周一到周五
```

要改就编辑这个文件 + commit。

⚠️ GitHub Actions 定时有 5-30 分钟漂移。

### 切 LLM provider

填不同的 secret 即可（按优先级）：

| 优先级 | 用的 provider |
|---|---|
| 1 | `GEMINI_API_KEY`（最便宜最快） |
| 2 | `AIHUBMIX_KEY`（国内友好） |
| 3 | `OPENAI_API_KEY` + `OPENAI_BASE_URL`（你 hermes 同款） |

## ✅ 看历史报告

https://github.com/wjk-dot/daily_stock_analysis/actions
→ 选 `00 Daily Analysis` → 列表里看每次 run 的 artifacts

artifacts 里通常有：
- `report-xxx.md`（完整决策报告）
- `analysis-results.json`（结构化结果）
- `notification-log.txt`（推送日志）

## ⚠️ 风险提醒

- **不是投资建议** — 推送结果仅供参考
- **免费 LLM 有超时风险** — MiniMax M2.5 推理慢，可能要换 Gemini 2.5 Flash 更稳
- **A 股市场风险** — 投资决策需独立判断
- **GitHub Actions 限制** — 每月 2000 分钟免费额度，每天 3-5 分钟够用

## ✅ 与 alphasift / hermes 的关系

```
DSA (这个 repo, GitHub Actions 云端)
  → 每日推送决策报告到飞书

alphasift (你已装到 ~/alphasift-venv)
  → 跑因子筛选 + LLM 排序，按需出表

hermes (飞书 chatbot)
  → LLM 通了以后，agent 可以调 alphasift，DSA 在 alphasift 背后
```
