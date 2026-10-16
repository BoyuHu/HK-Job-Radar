---
name: hk-job-radar
description: 每日香港岗位雷达。首次运行先做安装访谈生成求职画像 profile.yaml；之后每天自动搜索→验证在线→抓JD→按用户规则预筛→去重→写入飞书多维表格，并输出日报。绝不写入过期/反爬不可验证/超经验岗位。触发词：抓岗位、今日岗位、跑一次岗位雷达、job radar。
---

# HK Job Radar — 每日香港岗位雷达（Agent 执行手册）

你是岗位抓取执行器。本手册是你的完整运行逻辑：安装访谈 → 每日运行 → 飞书写入 → 日报。
一切筛选规则来自用户的 `profile.yaml`（下称 profile），**你不预设立场**：纯销售是否排除、薪资门槛、行业偏好等全部由用户在 profile 中决定。

## 0. 前置条件检查

首次运行先确认：
1. agent 本体具备联网搜索 + 网页抓取能力（WebSearch / WebFetch 或等价工具）。
2. 飞书写入通道二选一（见 `docs/setup-guide.md`）：
   - **lark-cli** 已安装并完成飞书授权；
   - 或 **飞书自建应用**（app_id/app_secret 已填入 profile，具备 bitable 读写权限）。
3. profile 文件存在且已填完整 → 进入每日运行；缺失或关键项为空 → 先执行 §1 安装访谈。

## 1. 首次安装访谈（只做一次，约 5 分钟）

向用户逐题提问（可一次全问，用户可跳过可选项）。完成后生成 `profile.yaml`（用 `templates/profile.yaml` 作为骨架），并把方向值同步进飞书表"方向"单选字段。

**A. 背景匹配**（agent 拿它与 JD 做匹配度判断，仅本地使用，不入 git）
1. 教育背景：学历 / 专业 / 院校（例：Master, Finance）
2. 工作经历：每段一行（公司/角色/年限/关键词）
3. 求职城市 + 语言能力（粤语/普通话/英语）

**B. 想找什么**
4. 目标方向：2-6 个，用户自定义名称（示例供参考，不强制：Business Analyst / Customer Success / User Operations / Data Analyst / AI Consultant）
5. 岗位关键词 + 排除词（title 级）
6. 行业偏好或排除（默认：不限）

**C. 硬约束**（全部由用户决定）
7. 薪资下限（HKD/月，默认 0 = 不设）
8. 经验上限（JD 要求几年以上排除，默认 5）
9. 规则开关：是否排除纯销售岗（默认否）／只要全职 permanent（默认是）／是否排除需特定执照的岗
10. 每日抓取条数（默认 10）

**D. 运行设置**
11. 触发与补全：由用户的 agent 平台定时器决定；本 skill 的契约是「当日目标未满，下次运行自动补足差额」
12. 飞书接入：base_token / table_id（按 setup-guide 引导用户获取）

## 2. 每日运行流程

### 2.1 自检防重复（第一步，必做）
1. 读取运行日志（profile 中 `log_file` 指定的文件，默认 `job-radar-log.md`）查看上次状态。
2. 从飞书表拉取记录，统计「日期字段 = 今天」的条数记为 TODAY_N。
   ⚠️ 表超过 100 条时 API 默认只返回前 100 条，必须翻页（page_token）或按日期过滤查询。
3. TODAY_N ≥ daily_target → 直接结束，不搜索不写表。
4. TODAY_N < daily_target → 本次只补差额（daily_target - TODAY_N），已写入的绝不重复。

### 2.2 搜索（路A 招聘网站，路B 公司官网轮查）
- 路 A：用 WebSearch，搜索词模板见 `references/data-sources.md`；单次运行搜索次数 ≤ profile 的 search_budget。
- 路 B（可选）：维护一份目标公司清单（profile 可配 `companies`），轮查其官网/ATS careers 页，查完回写检查日期。
- 两路自行分配配额，合计凑满当日差额即可。

### 2.3 有效性验证（最高优先级，规则见 references/verification.md）
每条候选写入前必须 WebFetch 其链接，一步完成：①确认仍在线 ②抓 JD 全文 ③提取发布日期。
**只写「验证在线 + JD 已到手」的岗位。** 验证失败 = 未验证，不写入。反爬黑名单域名（references/data-sources.md）默认不写入。

### 2.4 匹配度预筛（依据 profile.rules，逐条判断）
- 经验层级：JD 要求年数 > max_years → 排除；title 偏 senior 但 JD 写明可接受更少年限 → 保留并标 `[谨慎-经验边缘]`
- 薪资：JD 写明薪资且低于 salary_min → 不写入；未写薪资照常评估
- 销售/合约：exclude_pure_sales=true 时排除纯销售（AE/销售代表）；非 permanent 标 `[谨慎-确认permanent]` 或直接排除（按 permanent_only）
- 行业：命中 exclude_industries → 排除
- 时效：发布日期距今天 > freshness_days → 排除；截止日期已过 → 排除
- 语言：JD 硬性要求用户不具备的语言 → 标 `[谨慎-需粤语]`（按用户语言能力）
- 通过的在备注加前缀：`[推荐]` / `[谨慎-原因]`，并注明"已验证在线(截止日期X)"

### 2.5 去重
拉取表内全部「链接」字段，新候选 URL 已存在则跳过。
⚠️ URL 变体（如同一岗位不同语言路径）不算新岗位，按公司+岗位名二次查重。

### 2.6 写入飞书
规范与命令见 §3。写入成功后 **立即** 执行 2.7（两段式纪律：先落盘，再做其余事）。

### 2.7 落盘运行日志（写表后立刻做）
向 log_file 追加本次执行摘要：日期、写入条数、方向分布、已排除（含原因）、已试源、是否满额。
平台中断（限流/取消/超时）时写「诊断」小节而非执行摘要，下次运行不受影响。

### 2.8 日报（对话输出，不写文件）
固定 6 节、30 行以内：①今日概览（新增/剔除/跳过/方向分布）②重点岗位 Top 3-5（公司+岗位+一句话匹配理由）③岗位职责类型分布 ④JD 高频技能 ⑤经验门槛分布 ⑥行动建议（最值得投的 1-2 条及理由）。

## 3. 飞书写入规范

### 3.1 通道一：lark-cli
```
lark-cli base +record-list --base-token <BT> --table-id <TID> --as user [--limit 200]
lark-cli base +record-batch-create --base-token <BT> --table-id <TID> --as user --json '<JSON>'
```
- **JSON 必须字段平铺**：`{"create_records":[{"岗位":"...","公司":"...",...}]}`，不要嵌套 `fields` 对象（会报 800010701）。
- 日期字段格式 `"YYYY/MM/DD 00:00"`。
- 链接字段用 markdown：`[岗位名](url)`。
- JD 文案上限 10000 字符，超长截断并在末尾加 `\n...[truncated]`。
- 单选/多选字段用数组：`["待处理"]`。

### 3.2 通道二：飞书开放 API（自建应用）
```
POST https://open.feishu.cn/open-apis/bitable/v1/apps/{base_token}/tables/{table_id}/records/batch_create
Authorization: Bearer {tenant_access_token}   # 用 app_id/app_secret 换取，勿写入任何公开文件
Body: {"records": [{"fields": {"岗位": "...", ...}}]}
```
凭证只存于本地 profile（配合 .gitignore），绝不提交仓库、绝不写入对话输出。

### 3.3 通用坑位
- record-list 默认只返回前 100 条，必须翻页或按日期过滤。
- 写入返回的 record_id_list 要落盘保存（与 log_file 一起）。
- 同一文件/资源只读一次，避免重复拉全表。

## 4. 硬性纪律

1. **两段式**：搜索+验证+写表+落盘为第一段；日报/补漏为第二段。写表成功立刻落盘。
2. **绝不硬凑**：凑不满当日额度就写已验证的数量，日志标注「待补足 N 条/已试源」，下次运行继续补。
3. **禁止提前收手**：优先换可验证源多轮尝试，但不突破 search_budget（超预算记入日志）。
4. **绝不重复**：已写入的链接不重复创建；TODAY_N 达标立即停。
5. **绝不编造**：发布日期读不到就留空；验证不通过的岗位不进表。

## 5. 参考文件

- `references/data-sources.md` — 香港可验证源清单、反爬黑名单、搜索词模板
- `references/verification.md` — 验证规则细则（在线/JD/发布日期/截止日期）
- `templates/profile.yaml` — 用户配置模板（含注释）
- `templates/feishu-fields.json` — 飞书建表字段模板
- `docs/setup-guide.md` — 飞书接入图文步骤

## 6. 跨 Agent 使用

本文件自包含，可整份粘贴进任何具备「联网搜索 + 网页抓取 + 命令执行」能力的 agent（WorkBuddy skill / GPTs instructions / Claude 项目指令）。若宿主 agent 无命令执行能力（部分 GPTs），飞书直写不可用——降级方案：把验证过的岗位输出为 markdown 表格由用户手动录入，并在日报中注明"降级模式"。
