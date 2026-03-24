# 配置文件

## RootData API Key

```
<YOUR_ROOTDATA_API_KEY>
```

> 获取方式：前往 https://www.rootdata.com/zh → 登录 → 个人中心 → API 申请
> ⚠️ 请勿将真实 API Key 提交到公开 GitHub 仓库，仅在本地 config.md 中填入。

## 可用接口

| 接口 | 方法 | 说明 |
|------|------|------|
| `/open/ser_inv` | POST | 搜索项目/人物，返回 id、名称、简介 |
| `/open/get_item` | POST | 项目详情，支持 include_team、include_investors |

## 请求头

```
apikey: <API_KEY>
language: cn
Content-Type: application/json
```

## 已知限制

- `hot_projects` 等高级接口需升级套餐（返回"权限不足"）
- `team` 字段有时返回空数组，需从 Medium 文章补充
- 融资轮次明细接口暂未找到有效端点
