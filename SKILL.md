---
name: lark-lite
description: "飞书轻量入口：认证指引 + 高频功能路由 + 常用快捷命令。不加载官方 Skill，直接通过 lark-cli 操作。"
metadata:
  requires:
    bins: ["lark-cli"]
  cliVersion: "1.0.39"
---

# Lark Lite

轻量飞书入口 Skill。只保留最核心、最稳定的内容，不依赖官方 Skill 加载。

**兼容 CLI 版本：1.0.39**

## 首次配置

```bash
# 1. 初始化应用配置
lark-cli config init

# 2. 登录（推荐权限范围）
lark-cli auth login --recommend

# 3. 验证状态
lark-cli auth status
```

如果之前登录过但权限不足，重新获取 Token：

```bash
lark-cli auth logout
lark-cli auth login --recommend
```

## 高频功能路由

| 需求 | lark-cli 子命令 | 详细指引 | 安装官方 Skill（可选） |
|:---|:---|:---|:---|
| 收发消息、群聊、搜索聊天记录 | `lark-cli im ...` | [`references/im.md`](references/im.md) | `npx skills add larksuite/cli -g -s lark-im` |
| 创建/读取/编辑文档 | `lark-cli docs ...` | [`references/doc.md`](references/doc.md) | `npx skills add larksuite/cli -g -s lark-doc` |
| 多维表格操作 | `lark-cli base ...` | [`references/base.md`](references/base.md) | `npx skills add larksuite/cli -g -s lark-base` |
| 电子表格操作 | `lark-cli sheets ...` | [`references/sheets.md`](references/sheets.md) | `npx skills add larksuite/cli -g -s lark-sheets` |
| 文件上传下载 | `lark-cli drive ...` | [`references/drive.md`](references/drive.md) | `npx skills add larksuite/cli -g -s lark-drive` |
| 查看日程/创建日程 | `lark-cli calendar ...` | [`references/calendar.md`](references/calendar.md) | `npx skills add larksuite/cli -g -s lark-calendar` |
| 会议记录/妙记 | `lark-cli vc ...` / `lark-cli minutes ...` | [`references/vc.md`](references/vc.md) | `npx skills add larksuite/cli -g -s lark-vc` |
| 知识库 | `lark-cli wiki ...` | [`references/wiki.md`](references/wiki.md) | `npx skills add larksuite/cli -g -s lark-wiki` |

> 快捷命令不够用？点击详细指引查看完整 Shortcuts 列表和用法。需要完整官方 Skill 时按需安装。

## 常用快捷命令

### 即时通讯

```bash
# 搜索群聊
lark-cli im +chat-search --query "群聊名称"

# 读取群聊最近消息
lark-cli im +chat-messages-list --chat-id "oc_xxx"

# 发消息到群聊
lark-cli im +messages-send --chat-id "oc_xxx" --text "内容"

# 发私聊
lark-cli im +messages-send --user-id "ou_xxx" --text "内容"

# 搜索消息
lark-cli im +messages-search --query "关键词"
```

### 文档

```bash
# 创建文档
lark-cli docs +create --title "标题" --markdown "# 内容"

# 读取文档
lark-cli docs +fetch --document-id "dox_xxx"

# 更新文档
lark-cli docs +update --document-id "dox_xxx" --markdown "# 新内容"

# 搜索文档
lark-cli docs +search --query "关键词"
```

### 电子表格

```bash
# 创建表格
lark-cli sheets +create --title "标题"

# 读取单元格
lark-cli sheets +values-get --spreadsheet-id "sht_xxx" --range "Sheet1!A1:D10"

# 写入单元格
lark-cli sheets +values-update --spreadsheet-id "sht_xxx" --range "Sheet1!A1" --values '[["a","b"]]'

# 追加行
lark-cli sheets +values-append --spreadsheet-id "sht_xxx" --range "Sheet1!A1" --values '[["a","b"]]'
```

### 云盘

```bash
# 上传文件
lark-cli drive +upload --file-path "./file.pdf"

# 下载文件
lark-cli drive +download --file-token "boxcn_xxx"
```

### 日历

```bash
# 查看今日日程
lark-cli calendar +agenda

# 创建日程
lark-cli calendar +create --title "会议" --start-time "2026-05-23T10:00:00+08:00" --end-time "2026-05-23T11:00:00+08:00"
```

## 通用技巧

- `--as bot`：使用 Bot 身份执行命令（默认是 user）
- `--dry-run`：预览请求不执行（适合有副作用的操作）
- `--format table`：表格输出；`--format pretty`：格式化 JSON
- `--page-all`：自动分页获取全部数据

## 官方仓库

- CLI 源码与 Skill：https://github.com/larksuite/cli
- 当官方 CLI 更新后，本 Skill 中的快捷命令可能需同步更新
