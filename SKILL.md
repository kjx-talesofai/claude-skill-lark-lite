---
name: lark-lite
description: "飞书轻量入口：认证指引 + 高频功能路由 + 常用快捷命令。不加载官方 Skill，直接通过 lark-cli 操作。默认以用户身份执行，Bot 身份仅在必要时切换。"
metadata:
  requires:
    bins: ["lark-cli"]
  cliVersion: "1.0.39"
---

# Lark Lite

轻量飞书入口 Skill。只保留最核心、最稳定的内容，不依赖官方 Skill 加载。

**兼容 CLI 版本：1.0.39**

## 身份策略（CRITICAL — 必须先读）

**默认以用户身份操作**。Agent 执行任何命令时：

1. **优先不加 `--as`**：让 CLI 自动解析（通常为 user）
2. **仅在明确需要时切 Bot**：目前只有下载消息图片必须用 `--as bot`
3. **不要默认加 `--as bot`**：Bot 视角看到的是应用资源，不是用户个人资源

**必须用 Bot 身份的操作：**

```bash
# 下载消息中的图片（User token 不支持此 API）
lark-cli im +messages-resources-download --as bot \
  --message-id "om_xxx" --file-key "img_v3_xxx" --type image --output ./img.png
```

**其他所有操作**（发消息、读消息、读日程、读/写文档、表格、云盘等）**都不需要 `--as bot`**。

## 首次配置

**不要用 `--recommend`**，它缺少多个高频 scope，会导致反复登录。

```bash
# 1. 初始化应用配置
lark-cli config init

# 2. 登录（引用 scope 文件，一次性获取全部已审批权限）
lark-cli auth login --scope "$(cat lark-lite-scopes.txt)"

# 3. 验证状态
lark-cli auth status
```

如果 token 过期，直接重登（scope 文件已保存，无需重新导出）：

```bash
lark-cli auth login --scope "$(cat lark-lite-scopes.txt)"
```

> `lark-lite-scopes.txt` 是本仓库自带的模板。如管理员后续审批了新 scope，先单独登录获取，然后重新导出文件：`lark-cli auth status | python3 -c "import json,sys; d=json.load(sys.stdin); open('lark-lite-scopes.txt','w').write(d.get('scope',''))"`

## 高频功能路由

| 需求 | lark-cli 子命令 | 详细指引 |
|:---|:---|:---|
| 收发消息、群聊 | `lark-cli im ...` | [`references/im.md`](references/im.md) |
| 创建/读取/编辑文档 | `lark-cli docs ...` | [`references/doc.md`](references/doc.md) |
| 多维表格操作 | `lark-cli base ...` | [`references/base.md`](references/base.md) |
| 电子表格操作 | `lark-cli sheets ...` | [`references/sheets.md`](references/sheets.md) |
| 文件上传下载 | `lark-cli drive ...` | [`references/drive.md`](references/drive.md) |
| 查看日程/创建日程 | `lark-cli calendar ...` | [`references/calendar.md`](references/calendar.md) |
| 会议记录/妙记 | `lark-cli vc ...` / `lark-cli minutes ...` | [`references/vc.md`](references/vc.md) |
| 知识库 | `lark-cli wiki ...` | [`references/wiki.md`](references/wiki.md) |

> 快捷命令不够用？点击详细指引查看完整 Shortcuts 列表和用法。

## 常用快捷命令

### 即时通讯

```bash
# 搜索群聊
lark-cli im +chat-search --query "群聊名称"

# 读取群聊最近消息
lark-cli im +chat-messages-list --chat-id "oc_xxx"

# 发消息到群聊（某些群可能报 230027 权限拒绝，是群白名单限制）
lark-cli im +messages-send --chat-id "oc_xxx" --text "内容"

# 发私聊
lark-cli im +messages-send --user-id "ou_xxx" --text "内容"

# 搜索消息（需管理员审批 search:message scope）
lark-cli im +messages-search --query "关键词"

# 下载消息中的图片（必须用 --as bot）
lark-cli im +messages-resources-download --as bot \
  --message-id "om_xxx" --file-key "img_v3_xxx" --type image --output ./img.png
```

### 文档（全部使用 v2 API）

> v2 `--fetch` 返回 `data.document.content`（文档块格式，含 `<title>`、`<callout>`、`<img>` 等标签），不是 markdown。

```bash
# 创建文档
lark-cli docs +create --api-version v2 --title "标题" \
  --content "# 内容" --doc-format markdown

# 读取文档
lark-cli docs +fetch --api-version v2 --doc "dox_xxx"

# 更新文档（--command 支持 overwrite / str_replace / append / block_* 等）
lark-cli docs +update --api-version v2 --doc "dox_xxx" \
  --command overwrite --content "# 新内容" --doc-format markdown

# 搜索文档（需管理员审批 search:docs:read scope）
lark-cli docs +search --query "关键词"
```

### 电子表格

```bash
# 创建表格
lark-cli sheets +create --title "标题"

# 读取单元格（先 sheets +info --url <URL> 获取 sheet-id）
lark-cli sheets +read --url "<URL>" --sheet-id "xxx" --range "A1:D10"

# 写入单元格
lark-cli sheets +write --url "<URL>" --sheet-id "xxx" --range "A1:B1" --values '[["a","b"]]'

# 追加行
lark-cli sheets +append --url "<URL>" --sheet-id "xxx" --range "A1:B1" --values '[["a","b"]]'
```

### 云盘

```bash
# 上传文件（需相对路径）
lark-cli drive +upload --file "./file.pdf"

# 下载文件
lark-cli drive +download --file-token "boxcn_xxx" --output ./file.pdf
```

### 日历

```bash
# 查看今日日程
lark-cli calendar +agenda

# 创建日程（注意：--summary 不是 --title，--start/--end 不是 --start-time/--end-time）
lark-cli calendar +create --summary "会议" --start "2026-05-23T10:00:00+08:00" --end "2026-05-23T11:00:00+08:00"
```

### Wiki / 知识库

```bash
# Wiki 链接解析（获取真实 obj_token 和 obj_type）
lark-cli wiki spaces get_node --params '{"token":"wikcn_xxx"}'

# 当 obj_type=bitable 时，obj_token 就是 --base-token
# 当 obj_type=docx 时，obj_token 就是 --doc
```

### 多维表格（Base）

```bash
# 获取 base 信息
lark-cli base +base-get --base-token "ThnLbclVFa8Jy4sq6nTc63ITnkb"

# 列出表格
lark-cli base +table-list --base-token "xxx"

# 读取记录
lark-cli base +record-list --base-token "xxx" --table-id "tblxxx"

# 列出字段
lark-cli base +field-list --base-token "xxx" --table-id "tblxxx"
```

## 踩坑速查

Agent 执行命令遇到问题时，按以下顺序排查：

| 报错 | 原因 | 解决 |
|:---|:---|:---|
| `missing_scope` | scope 未授权 | 让管理员在飞书开放平台审批对应 scope，然后重新登录 |
| `Permission denied [230027]` | 群聊禁止该应用发消息 | 群白名单限制，非 scope 问题 |
| `user access token not support` (99991668) | 该 API 不支持 User token | 加 `--as bot` 重试 |
| `Invalid request param [234001]` | 参数格式错误 | 检查命令帮助，对比本 Skill 中的示例 |
| `File not in msg [234003]` | message-id 和 file-key 不匹配 | 确保从同一条消息中获取两者 |
| `not found sheetId [90215]` | 缺少 --sheet-id | 先用 `sheets +info --url <URL>` 获取 |
| `range in request is wrong [90202]` | range 格式错误 | 用 `A1:B1` 而不是 `Sheet1!A1`，且要和 values 列数匹配 |
| `unsafe file path` | 用了绝对路径 | 改用相对路径 `./filename` |
| `unknown flag: --format` | 该命令不支持 --format | 去掉 --format 或改用 `--format json` 看是否支持 |
| `unknown flag: --app-token` | base 命令参数名错误 | 用 `--base-token` 不是 `--app-token` |
| Wiki 链接无法直接操作 | `/wiki/{token}` 不是真实文档 token | 先用 `wiki spaces get_node` 获取 `obj_token` 和 `obj_type` |
| base 读取报错缺少 table-id | 未指定数据表 | 先用 `base +table-list --base-token xxx` 获取 `table-id` |

## 通用技巧

- `--dry-run`：预览请求不执行（适合有副作用的操作）
- `--format table`：表格输出；`--format pretty`：格式化 JSON
- `--page-all`：自动分页获取全部数据
- `--jq '.data.chats[0].chat_id'`：用 jq 提取特定字段

## 官方仓库

- CLI 源码与 Skill：https://github.com/larksuite/cli
- 当官方 CLI 更新后，本 Skill 中的快捷命令可能需同步更新
