---
name: lark-lite
description: "飞书轻量入口：身份策略 + 高频命令 + 功能路由。不加载官方 Skill，直接通过 lark-cli 操作。默认 user 身份，Bot 仅在必要时切换。"
metadata:
  requires:
    bins: ["lark-cli"]
  cliVersion: "1.0.39"
---

# Lark Lite

轻量飞书入口 Skill。只保留最核心、最稳定的内容，不依赖官方 Skill 加载。

**兼容 CLI 版本：1.0.39**

## 身份策略

**默认以用户身份操作**。优先不加 `--as`，让 CLI 自动解析。只有下载消息中的图片需要 `--as bot`（User token 不支持该 API）。不要默认加 `--as bot`，否则看到的是应用资源而非个人资源。

```bash
# 唯一需要 Bot 身份的操作
lark-cli im +messages-resources-download --as bot \
  --message-id "om_xxx" --file-key "img_v3_xxx" --type image --output ./img.png
```

## 首次配置

**不要用 `--recommend`**，它缺少多个高频 scope，会导致反复登录。

### ⚠️ Cohub 沙盒特别注意

`config init --new` 和 `auth login` 都是**阻塞式轮询命令**：它们启动后在本地等待用户在浏览器完成 OAuth 授权，进程必须一直活着才能轮询到结果并写入配置。

Cohub 的 shell 不是桌面终端——bash 命令作为 tool call 运行，受 turn/task 生命周期管控：
- 前台命令设了短 `timeout` → 到点杀进程
- 当前 turn 被 abort/interrupt → 关联进程被杀
- background task 超时/完成后的 **steer 消息会打断当前 turn**，连带杀掉等待中的进程（2025-07-06 后新增）
- 杀掉的进程对应的 `user_code` 立即作废，即使用户在浏览器已授权也无法写入配置

**稳妥方式：用 `nohup` 让登录流程脱离 Cohub 的 tool/turn 生命周期。**

#### 步骤一：config init（仅首次，或配置丢失后）

```bash
# 用 nohup 逃逸 Cohub 管控，避免进程被 turn/steer 杀掉
nohup lark-cli config init --new > /tmp/lark-cli-init.log 2>&1 &
sleep 8
cat /tmp/lark-cli-init.log   # 获取 verification URL，发给用户
```

等用户在浏览器完成授权后，验证配置写入：
```bash
lark-cli config show
```

> ⚠️ `config init --new` 每次运行都会创建全新的飞书应用 + 新 `user_code`。
> 如果一次不成功，**不要在同一 turn 重试**——旧进程可能还没释放，会叠出多个等待器互相干扰。
> 先 `pkill -f "lark-cli config init"` 清理干净再重来。

#### 步骤二：auth login（每次 token 过期后也需要）

用 **两步式**（`--no-wait` → 展示二维码 → `--device-code`），不要用前台阻塞式：

```bash
# Step 1: 获取 device_code 和 verification_url
lark-cli auth login --no-wait --json --scope "$(cat lark-lite-scopes.txt)"

# Step 2: 生成二维码 PNG（必须展示给用户）
cd /workspace && lark-cli auth qrcode --output ./feishu-login-qr.png "<verification_url>"

# Step 3: 用户扫码授权后，用 device_code 完成登录
lark-cli auth login --device-code "<device_code>"
```

> ⚠️ `--device-code` 也是阻塞轮询（最长 10 分钟），但通常几秒内就能返回，风险远小于 `config init`。

#### 验证

```bash
lark-cli auth status
```

### 桌面终端（非 Cohub）

在普通桌面终端中直接运行即可，无需 `nohup`：

```bash
lark-cli config init
lark-cli auth login --scope "$(cat lark-lite-scopes.txt)"
lark-cli auth status
```

---

`lark-lite-scopes.txt` 是本仓库自带的模板。token 过期后直接重跑 `auth login` 即可（不需要重新 `config init`）。管理员后续审批了新 scope，用 `lark-cli auth status | python3 -c "import json,sys; d=json.load(sys.stdin); open('lark-lite-scopes.txt','w').write(d.get('scope',''))"` 重新导出。

> **沙盒重启会清掉 `~/.lark-cli/`，届时需要从步骤一开始。** 可以考虑把 `~/.lark-cli/config.json` 备份到 workspace 中。

## 高频命令速查

> 参数疑问？运行 `lark-cli <command> --help` 查看完整帮助。复杂场景？查看详细指引。

### 即时通讯

```bash
lark-cli im +chat-search --query "群聊名称"
lark-cli im +chat-messages-list --chat-id "oc_xxx"
lark-cli im +messages-send --chat-id "oc_xxx" --text "内容"
lark-cli im +messages-send --chat-id "oc_xxx" --markdown "# 标题\n\n**加粗**"
lark-cli im +messages-send --chat-id "oc_xxx" --msg-type interactive --content '{"config":{"wide_screen_mode":true},"header":{"title":{"tag":"plain_text","content":"卡片标题"}},"elements":[{"tag":"div","text":{"tag":"lark_md","content":"**内容**"}}]}'  # 卡片格式详见 references/im-interactive-card.md
lark-cli im +messages-send --user-id "ou_xxx" --text "内容"
lark-cli im +messages-resources-download --as bot --message-id "om_xxx" --file-key "img_v3_xxx" --type image --output ./img.png
```

### 文档（v2 API）

```bash
lark-cli docs +create --api-version v2 --title "标题" --content "# 内容" --doc-format markdown
lark-cli docs +fetch --api-version v2 --doc "dox_xxx"
lark-cli docs +update --api-version v2 --doc "dox_xxx" --command overwrite --content "# 新内容" --doc-format markdown
```

> `--fetch` 返回 `data.document.content`（文档块格式），不是 markdown。

### 电子表格

```bash
lark-cli sheets +create --title "标题"
lark-cli sheets +info --url "<URL>"                  # 获取 sheet-id
lark-cli sheets +read --url "<URL>" --sheet-id "xxx" --range "A1:D10"
lark-cli sheets +write --url "<URL>" --sheet-id "xxx" --range "A1:B1" --values '[["a","b"]]'
lark-cli sheets +append --url "<URL>" --sheet-id "xxx" --range "A1:B1" --values '[["a","b"]]'
```

### 云盘

```bash
lark-cli drive +upload --file "./file.pdf"           # 需相对路径
lark-cli drive +download --file-token "boxcn_xxx" --output ./file.pdf
```

### 日历

```bash
lark-cli calendar +agenda
lark-cli calendar +create --summary "会议" --start "2026-05-23T10:00:00+08:00" --end "2026-05-23T11:00:00+08:00"
```

> 无 `+search` shortcut，需用原生 API `calendar events search_event`。

### 视频会议

```bash
lark-cli vc +search --start "2026-05-01T00:00:00+08:00" --end "2026-05-23T23:59:59+08:00"
lark-cli vc +notes --meeting-ids "764xxx"              # 返回 note_doc_token / verbatim_doc_token
lark-cli vc +recording --meeting-ids "764xxx"          # 返回 minute_token
lark-cli vc meeting get --params '{"meeting_id":"764xxx","with_participants":true}'
```

> `vc +search` 必须带 `--start/--end` 或 `--query` 等至少一个过滤条件。

### 通讯录

```bash
lark-cli contact +search-user --query "姓名"           # 获取 open_id
```

### Wiki / 多维表格

```bash
# Wiki 链接解析（URL 里的 token 不是真实文档 token）
lark-cli wiki spaces get_node --params '{"token":"wikcn_xxx"}'

# 根据 obj_type 决定后续操作：
#   obj_type=docx → docs +fetch --api-version v2 --doc <obj_token>
#   obj_type=bitable → base +record-list --base-token <obj_token> --table-id <tblxxx>

lark-cli base +table-list --base-token "xxx"
lark-cli base +record-list --base-token "xxx" --table-id "tblxxx"
```

## 关键约束（CLI 不会提示的）

1. **某些群聊禁止应用发消息** — `im +messages-send` 报 `Permission denied [230027]` 是群白名单限制，不是 scope 问题。
2. **搜索功能缺 scope** — `im +messages-search` 和 `docs +search` 需要管理员在飞书开放平台审批 `search:message` / `search:docs:read`，目前未获批。
3. **某些命令 JSON 输出带前缀** — `wiki +node-list`、`+space-list`、`+node-get` 即使 `--format json` 也会在 JSON 前加 `Found X node(s)` 等前缀，需 `tail -n +2` 跳过第一行后再解析。

## 功能路由

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

## 通用技巧

### 探索优先：用 `--help` 探查命令结构

不确定命令用法或遇到参数错误时，**优先运行 `--help`**，不要猜测参数名：

```bash
lark-cli --help                          # 查看所有顶级命令
lark-cli im --help                       # 查看 im 子命令列表
lark-cli im +messages-send --help        # 查看具体命令的参数和示例
lark-cli schema im.message.create        # 查看原生 API 的参数结构
```

> CLI 的 `--help` 通常比文档更新更快，遇到报错先看 help 确认参数是否已变更。

### 其他技巧

- `--dry-run`：预览请求不执行
- `--page-all`：自动分页获取全部数据
- `--jq '.data.chats[0].chat_id'`：用 jq 提取特定字段
