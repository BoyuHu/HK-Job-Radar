# 飞书接入指南（约 10 分钟）

两条通道任选其一：**lark-cli**（WorkBuddy 内最简单）或**飞书自建应用**（通用，任何 agent 可用）。

## 第 1 步：创建多维表格（两条通道都需要）

1. 打开飞书 → 新建「多维表格」，命名如「我的岗位雷达」。
2. 建一张数据表，字段按 `templates/feishu-fields.json` 手动创建：
   - 岗位 / 公司：文本
   - 方向：单选（选项填你 profile 里的 directions）
   - 状态：单选（待处理 / 已投递 / 面试中 / 不合适 / 过期）
   - 链接 / 备注 / JD文案：文本（JD文案建议调大字数上限）
   - 日期 / 发布时间：日期
3. 从表格 URL 中取出凭证：
   `https://xxx.feishu.cn/base/{base_token}?table={table_id}&...`
   → `base_token` 和 `table_id` 填入 `profile.yaml` 的 `feishu` 段。

## 第 2 步 A：lark-cli 通道（WorkBuddy 用户推荐）

1. 确认 agent 环境有 `lark-cli`（WorkBuddy 内置）。
2. 对 agent 说：「用 lark-cli 登录飞书」→ 完成浏览器授权（`--as user`）。
3. 验证：让 agent 跑一次 `+record-list`，能读出空表即通。
4. `profile.yaml` 中 `channel: "lark-cli"`。

## 第 2 步 B：飞书自建应用通道（其他 agent 用户）

1. 打开 [飞书开放平台](https://open.feishu.cn/) → 创建**企业自建应用**（个人版飞书也可）。
2. 在「权限管理」开通：`bitable:app`（多维表格读写）。
3. 在「安全设置」添加重定向 URL（如按提示需要）。
4. 发布应用版本并启用。
5. 复制 `App ID` / `App Secret` 填入 `profile.yaml` 的 `feishu.app_id / app_secret`。
6. **把表格授权给应用**：打开多维表格 → 右上角「…」→「更多」→「添加文档应用」→ 搜索你的应用名并添加（可编辑权限）。**漏掉这步 API 会报 403。**
7. `profile.yaml` 中 `channel: "openapi"`。

## 第 3 步：验证

对 agent 说：「跑一次岗位雷达（测试模式，1 条即可）」。
- 表里出现 1 条验证过的岗位 → 接入成功。
- 报 403/99991663 → 检查第 2 步 B 的文档授权或权限范围。

## 安全提醒

- `app_secret` 只存本地 `profile.yaml`（本仓库 `.gitignore` 已排除该文件名）。
- 不要把 profile 内容粘贴到公开对话、issue 或截图中。
