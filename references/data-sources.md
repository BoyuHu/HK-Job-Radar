# 香港岗位数据源指南

> 实战经验沉淀：哪些源能验证、哪些是反爬坑、搜索词怎么写命中率最高。

## 1. 可验证源（WebFetch 能取到 JD 全文，优先使用）

| 源 | 说明 | 注意事项 |
|---|---|---|
| 公司官网 ATS 直链 | lever.co / greenhouse.io / ashbyhq.com / myworkdayjobs.com / smartrecruiters.com | **质量最高**：JD 全文、发布/截止日期齐全，优先命中 |
| openjobs.com.hk | 中文友好，多为全职岗 | 站内 keyword 搜索页 JS 渲染不全（每次只出 1-2 条）——用 `site:openjobs.com.hk` 搜索定向命中，别直抓其搜索页 |
| bebee.com | 岗位量大，更新快 | query 参数直抓易失效返回通用列表；分类页/详情页可用。**搜索摘要显示的"X天前"与详情页常不一致，以详情页为准**；大量旧岗重发，截止日期过期的照样排除 |
| recruit.com.hk | 中小公司岗 | 基本可验证 |
| techjobasia.com | 科技岗，对应届友好 | 列表页可读 |
| jobstore.com | 聚合岗（含新加坡公司驻港） | 详情页可验证 |
| headlinejobs.hk | 本地化岗位 | 可验证 |
| globalscope | 外资/金融岗 | 部分需登录 |

## 2. 反爬黑名单（默认不写入，无法验证在线状态）

```
jobsdb.com  ctgoodjobs.com  linkedin.com  jobstreet  glassdoor
trabajo.org  thebigjobsite.com  ricebowl  joblum.com  offertoday  jobleads
```

- 这些站点 WebFetch 通常返回 403/451/登录墙/空壳页，**JD 抓不到 = 未验证 = 不写入**。
- JobsDB 可以用来**发现**岗位（搜索结果里能看到 JD 摘要），但要投递/入库就去该公司官网或其他可验证源找同一条岗位的直链。

## 3. 搜索词模板

按用户的 profile.directions（方向）+ 通用模板轮换组合，避免每天同一批词：

```
Hong Kong {方向关键词} job hiring this week site:bebee.com
Hong Kong {方向关键词} job apply site:openjobs.com.hk
Hong Kong {方向关键词} graduate OR trainee OR entry apply
{公司名} careers graduate programme Hong Kong        # 路B：公司官网轮查
site:bebee.com Hong Kong {关键词} "天前" OR "days ago"  # 时效限定
```

实战技巧：
- **应届/初级岗稀缺日**：加 `fresh graduate welcome`、`entry level`、`trainee` 变体。
- 搜索结果里的岗位先在脑内比对已入库清单再验证，省 WebFetch 次数。
- 每日命中率低是常态：优先换 site 限定词和源，而不是加大次数硬搜。

## 4. 路 B：公司官网轮查

1. 维护目标公司清单（profile 的 `companies`，含公司名 + careers 页链接 + 上次检查日期）。
2. 每次运行取「上次检查日期最旧」的 2-5 家，WebFetch 其 careers 页/ATS 列表。
3. 命中匹配岗位 → 走正常验证与写入流程；回写该公司「上次检查日期」。
4. 路A 已凑满当日配额可跳过路B。
