---
name: project-rating
description: Web3 项目综合评级报告生成。输入项目名称，自动收集多维度数据并按评级框架（A/A-/B+/B-）输出结构化评级报告。数据来源：RootData API（基本信息/融资/团队）、官网/Medium（产品进展）、DefiLlama API（TVL/Fees/Revenue）、CoinMarketCap（用户数/交易所预期）。触发词：评级、rating、帮我评级、项目评分、对XX做评级。
---

# Project Rating Skill

## 概述

对 Web3 项目进行 A / A- / B+ / B- 四级评估，输出带数据来源标注的完整评级报告。

## 执行流程

### Step 1：RootData API 获取基本信息

读取 `references/config.md` 获取 API Key。

```bash
# 搜索项目 ID
curl -s -X POST "https://api.rootdata.com/open/ser_inv" \
  -H "apikey: <API_KEY>" -H "language: cn" -H "Content-Type: application/json" \
  -d '{"query":"<项目名>","include_tags":1}'

# 获取项目详情（融资/投资人/社交链接/标签）
curl -s -X POST "https://api.rootdata.com/open/get_item" \
  -H "apikey: <API_KEY>" -H "language: cn" -H "Content-Type: application/json" \
  -d '{"project_id":<ID>,"include_team":1,"include_investors":1}'
```

获取字段：`tags`、`social_media`（website/X/medium）、`investors`（Tier/lead）、`token_symbol`、`establishment_date`

### Step 2：产品进展 & 技术架构

用 `web_fetch` 依次抓取：
1. **官网** `<website>` → 路线图、用户规模声称数据
2. **Medium** `medium.com/@<handle>` → 产品上线公告、融资公告、TGE 信息、用户数据

### Step 3：链上数据（DefiLlama API）

```bash
# Derivatives/DEX 交易量
curl -s "https://api.llama.fi/overview/derivatives?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyVolume" \
  | python3 -c "import sys,json; d=json.load(sys.stdin); [print(json.dumps(p,indent=2)) for p in d.get('protocols',[]) if '<项目名>' in p.get('name','').lower()]"

# Fees（7D / 30D）
curl -s "https://api.llama.fi/overview/fees?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyFees" \
  | python3 -c "import sys,json; d=json.load(sys.stdin); [print('fees 7d:',p.get('total7d'),'30d:',p.get('total30d')) for p in d.get('protocols',[]) if '<项目名>' in p.get('name','').lower()]"

# Revenue
curl -s "https://api.llama.fi/overview/fees?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyRevenue" \
  | python3 -c "import sys,json; d=json.load(sys.stdin); [print('rev 7d:',p.get('total7d')) for p in d.get('protocols',[]) if '<项目名>' in p.get('name','').lower()]"
```

关键指标：`total24h`、`total7d`、`total30d`、`change_7d`

### Step 4：交易所预期

用 `web_fetch` 抓取 `https://coinmarketcap.com/currencies/<slug>/`，获取 Pre-market 状态及交易所上线信息。

### Step 5：按评级框架打分并输出报告

参考 `references/rating-framework.md` 中的评级标准，逐项打分后给出综合评级。

---

## 报告输出模板

```
📊 <项目名> 项目评级报告

🔧 数据来源说明
• 📋 基本信息/赛道/投资人：RootData API
• 🏗️ 产品/路线图：官网 + Medium
• 💰 Fees/Revenue/Volume：DefiLlama API
• 📈 用户数/交易量：CoinMarketCap
• 🏦 交易所预期：CoinMarketCap Pre-market
• ⚠️ 推特粉丝/Discord/TG：平台拦截，需手动核查

---

📋 基本信息
现阶段：[主网/测试网/早期]
赛道：[来自 RootData 标签]
官网 / X / Medium / Discord

---

🚀 产品进展
[路线图关键节点 + 完成状态]
[核心技术亮点]
[用户/交易量数据（官方声称）]

---

📊 链上数据（DefiLlama）
| 指标 | 数值 |
| 24h Volume | |
| 7D Volume | |
| 7D Fees | |
| 7D Revenue | |
| 7D 环比 | |

---

💰 融资情况（RootData）
| 机构 | 级别 | 角色 |

---

👥 团队背景（RootData）
| 成员 | 职位 |

---

📊 社群热度
[X 粉丝 / Discord / TG / KOL 情况]

---

🏦 交易所预期
[CMC/已上/待确认]

---

🎯 评级逐项打分
| 维度 | 判断 | 等级 |
| ① 基本面&产品 | | |
| ② 赛道风口 | | |
| ③ 融资规模 | | |
| ④ 团队背景 | | |
| ⑤ 社群热度 | | |
| ⑥ 交易所预期 | | |

---

🏆 综合评级：[🟢 A / A-  |  🟡 B+  |  🟠 B-]
升级缺口：[列出待确认项]
```

---

## 注意事项

- RootData API Key 存于 `references/config.md`，不可提交到公开仓库
- DefiLlama 直接调 REST API，无需登录；Dune/X/Discord 受平台限制，标注"需手动核查"
- 投资人 Tier 定义见 `references/rating-framework.md`
- 若 DefiLlama 未收录该项目，TVL/Fees 标注 N/A，改从官方公告获取
