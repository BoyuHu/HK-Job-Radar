# HK Job Radar 🇭🇰

每日香港岗位雷达：一个可直接装进 AI Agent 的开源 Skill。首次安装通过 12 题访谈生成你的专属求职画像，之后每天自动**搜索 → 验证在线 → 抓取 JD → 按你的规则筛选 → 去重 → 写入飞书多维表格**，并在对话里输出一份求职日报。

> A daily Hong Kong job radar as an installable agent skill. Onboarding interview → daily search & verify & filter → Feishu bitable + chat report. English intro below · 中文文档为主。

## 为什么不是又一个 job scraper

| 常见爬虫脚本 | HK Job Radar |
|---|---|
| 抓到什么写什么，大量过期/已下架岗位 | **验证优先**：写入前逐条访问链接，确认在招 + JD 到手 + 发布日期，三样缺一不写 |
| 反爬站（JobsDB/LinkedIn 等）抓到残缺文本 | 内置香港**可验证源清单 + 反爬黑名单**（几十次实战踩坑沉淀） |
| 筛选规则写死在代码里 | **零预设价值观**：薪资门槛、是否排除销售岗、行业偏好、方向分类……全部由你在配置里决定 |
| 跑一次是一锤子买卖 | 每日配额 + 防重复 + **断点自动补足**：当天没抓满，下次运行自动补差额 |
| 结果散落在控制台 | 结构化写入飞书多维表格（含 JD 全文、发布时间、备注），每天对话输出日报 |

## 快速上手（3 步）

1. **安装**：把本仓库放进你的 agent
   - WorkBuddy：复制到 `~/.workbuddy/skills/hk-job-radar/`
   - 其他 agent（GPTs / Claude 等）：把 `SKILL.md` 全文粘贴进 instructions（需 agent 具备联网搜索、网页抓取、命令执行能力）
2. **接入飞书**（10 分钟）：按 `docs/setup-guide.md` 建表并填入凭证
3. **对话触发**：对 agent 说「抓岗位」→ 它会先做安装访谈，生成 `profile.yaml`，之后每天定时（或你手动说一声）自动运行

## 配置即一切

所有规则都在 `profile.yaml`，改完即生效：

```yaml
rules:
  salary_min: 0              # 月薪下限（HKD），0 = 不设
  max_years: 5               # JD 要求经验超过几年排除
  exclude_pure_sales: false  # 是否排除纯销售岗 —— 你自己定
  permanent_only: true       # 只要全职 permanent
  exclude_industries: []     # 行业排除，空 = 不限
  freshness_days: 92         # 发布超过 3 个月的不写入
schedule:
  daily_target: 10           # 每日新岗位目标条数
  search_budget: 6           # 单次运行最多搜索次数
```

## 每日日报长什么样

每天运行结束，agent 在对话里输出：

1. **今日概览** — 新增 N 条 / 跳过重复 K 条 / 方向分布
2. **重点岗位 Top 3-5** — 公司 + 岗位 + 为什么匹配你的背景
3. **岗位职责类型分布** — 方案交付 / 售前 / 需求分析 / 客户成功…
4. **JD 高频技能** — SQL / Python / LLM / 粤语…
5. **经验门槛** — 应届友好几条 / 1-3 年几条 / 经验边缘几条
6. **行动建议** — 最值得优先投的 1-2 条及理由

## 项目结构

```
hk-job-radar/
├── SKILL.md                  # Agent 执行手册（核心，自包含，可单文件使用）
├── references/
│   ├── data-sources.md       # 香港可验证源清单 + 反爬黑名单 + 搜索词模板
│   └── verification.md       # 在线验证 / JD 抓取 / 发布日期规则
├── templates/
│   ├── profile.yaml          # 求职画像配置模板（含注释）
│   └── feishu-fields.json    # 飞书建表字段模板
├── docs/
│   └── setup-guide.md        # 飞书接入步骤（lark-cli / 自建应用两条路）
└── LICENSE
```

## 要求与限制

- 宿主 agent 需具备：联网搜索、网页抓取、命令执行（无命令执行能力可用降级模式，见 SKILL.md §6）
- v1 仅支持飞书多维表格作为存储后端；v2 计划支持 Notion / Google Sheets / 本地 CSV
- v1 仅内置香港数据源 know-how；其他城市可通过 profile 自行扩展搜索词
- 你的 profile 与凭证只存本地（已在 .gitignore），不会进入本仓库

## 隐私

仓库内不含任何使用者的求职数据。安装访谈产生的画像、抓取的岗位、飞书凭证全部保存在你自己的环境里，是否联网、传给哪家模型由你选择的宿主 agent 决定。

## License

MIT
