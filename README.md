# web3Project-research Skills

Web3 项目调研 Skill 合集，适用于 [OpenClaw](https://openclaw.ai) 智能助手。

## Skills 列表

### 📋 project-research — Web3 项目调研

自动从 RootData API、Medium 博客、官方文档等来源收集项目信息，按标准模板输出调研报告。

**报告包含：**
- 项目阶段 / 赛道标签
- 官网 / 推特 / Discord / TG 链接
- TGE 时间 / 用户数据 / 融资情况
- 团队背景 / 空投信息 / 社区热度

**使用方式：**
1. 将 `project-research/` 文件夹放入 OpenClaw `workspace/skills/` 目录
2. 编辑 `project-research/references/config.md`，填入你的 RootData API Key
3. 直接对话："帮我调研 XXX 项目"

**获取 RootData API Key：** https://www.rootdata.com/zh → 登录 → 个人中心 → API

## 安装

```bash
# 方式一：直接使用 .skill 文件
cp project-research.skill ~/.openclaw/workspace/skills/

# 方式二：使用源码目录
cp -r project-research ~/.openclaw/workspace/skills/
```
