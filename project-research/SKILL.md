---
name: project-research
description: Web3 项目调研报告生成。当用户输入项目名称并要求调研时激活。自动从 RootData API、官方文档、Medium 博客等来源收集：项目阶段、赛道标签、官网/推特/Discord/TG 链接、TGE 时间、用户数据、融资情况、团队背景、空投信息、社区热度，并按标准模板输出完整调研报告。触发词：调研、research、项目调研、帮我查、帮我调研。
---

# Project Research Skill

## 概述

按照标准模板，对 Web3 项目进行全面调研并输出结构化报告。

## 执行流程

### Step 1：RootData API 搜索项目

使用 `exec` 调用 RootData API，API Key 从用户提供或检查 references/config.md。

**搜索接口（获取项目 ID）：**
```bash
curl -s -X POST "https://api.rootdata.com/open/ser_inv" \
  -H "apikey: <API_KEY>" \
  -H "language: cn" \
  -H "Content-Type: application/json" \
  -d '{"query":"<项目名>","include_tags":1}'
```

**项目详情接口（获取融资、投资人、标签、社交链接）：**
```bash
curl -s -X POST "https://api.rootdata.com/open/get_item" \
  -H "apikey: <API_KEY>" \
  -H "language: cn" \
  -H "Content-Type: application/json" \
  -d '{"project_id":<ID>,"include_team":1,"include_investors":1}'
```

返回字段：`total_funding`、`tags`、`social_media`（website/X/discord/telegram）、`investors`（lead_investor=1 为领投）、`token_symbol`、`establishment_date`

### Step 2：抓取补充信息

从 RootData 获取社交链接后，依次抓取：

1. **官方文档**：`docs.<domain>` 或 gitbook 链接 → 获取产品功能描述
2. **Medium 博客**：`medium.com/@<项目名>` → 获取融资公告、产品上线、TGE、空投信息、用户数据
3. **官网**：`<website>` → 获取用户规模口号数据

抓取时用 `web_fetch`，重点文章：融资公告、产品上线公告、空投说明、TGE 公告。

### Step 3：按模板输出报告

见下方「报告模板」。

---

## 报告模板

```
📋 <项目名> 项目调研报告

现阶段：[测试网 / 主网上线 / 公测阶段 / 早期 / ToB业务]
赛道：[来自 RootData 标签，如：基础设施 / Layer1 / AI 代理 / AI]

官网：<website>
官方文档：<docs链接>
推特：<X链接>
Discord：<discord链接>
TG：<telegram链接，若无则注明暂未公开>

---

🚀 项目进展

TGE：[时间+是否完成]；<链接>
[重大事件1，附链接]
[重大事件2，附链接]
用户数据：[DAU/MAU/付费用户/开发者数等]
产品交易量：[如有]
奖项：[如有]

---

🎯 是否明牌空投
✅/❌ [说明 + 链接]

---

💡 一句话项目概述
<一句话>

---

💰 融资情况
累计融资 XXX 万美金
[时间，轮次，金额，机构*（*=领投），链接]

---

👥 团队亮点
[姓名]：[职位]。此前在 [公司/机构] 任职；[学术背景]
[姓名]：[职位]。...

---

📊 社交媒体热度
推特 @<handle>：[粉丝数，近期阅读量，KOL 讨论情况]
Discord：[用户数，在线人数]
TG：[用户数，在线人数]
[是否有负面舆论]
```

---

## 阶段判断规则

| 条件 | 阶段判断 |
|------|---------|
| 代币已上线交易所 | 主网上线 / TGE 已完成 |
| 产品上线但代币未发 | 公测阶段 |
| L1/协议处于 TestNet | 测试网 |
| 仅有融资公告，无产品 | 早期 |
| 无 C 端产品，主做企业服务 | ToB 业务 |

---

## 数据来源优先级

1. **RootData API**（结构化数据：融资额、投资人、标签、社交链接）
2. **Medium 官方博客**（时间线事件：融资公告、产品上线、TGE、空投）
3. **官方文档**（产品功能描述）
4. **官网**（用户规模数据）
5. **X/Discord/TG**（社区热度，因登录限制可能无法自动抓取，标注"需手动核查"）

---

## 注意事项

- RootData API 当前套餐支持：`ser_inv`（搜索）、`get_item`（详情+投资人）
- 推特/Discord/TG 实时在线数据受平台登录限制，无法自动抓取，需提示用户手动核查
- 融资轮次明细若 API 未返回，从 Medium 融资公告文章补充
- 团队信息若 API `team` 字段为空，从 Medium 创始人文章或融资公告中提取
- 所有关键信息必须附带原始链接
- 参考配置文件：`references/config.md`
