<p align="center">
  <img src="https://assets.hypersampling.com/hyper-sampling-2.jpg" alt="hyper-sampling" height="50"/>
  &nbsp;&nbsp;&nbsp;
  <img src="https://raw.githubusercontent.com/kjx-talesofai/claude-skill-hypersampling/master/neta_logo.png" alt="neta.art" height="50"/>
</p>

<p align="center">
  <strong><a href="https://hypersampling.com">Jiaxin Kou 寇佳新</a></strong>
  &nbsp;·&nbsp;
  <strong><a href="https://www.neta.art">Neta Art 捏Ta</a></strong>
  &nbsp;·&nbsp;
  <a href="https://github.com/kjx-talesofai">GitHub @kjx-talesofai</a>
</p>

---

# lark-lite

个人/组织专用的飞书轻量入口 Skill。替代官方 30 个 Skill 的庞大上下文占用，只保留最高频、最稳定的操作。

> **核心定位**：Agent 通过本 Skill 指引，以**用户个人身份**操作飞书，而非 Bot 身份。

## 安装

### 前置依赖：lark-cli

本 Skill 依赖飞书官方 CLI 工具。如果尚未安装：

```bash
npm install -g @larksuite/cli
```

> 官方仓库：https://github.com/larksuite/cli

### 安装 Skill

```bash
npx skills add kjx-talesofai/lark-lite -g
```

## 设计原则

- **纯路由**：SKILL.md 只包含身份策略、高频指令和踩坑速查
- **不引用官方 Skill 内容**：references/ 目录保存官方文档副本，按需读取
- **手动维护快捷命令**：只保留经过实机测试的命令，自己控制更新节奏
- **用户身份优先**：所有操作默认以用户身份执行，Bot 身份仅在必要时切换

## 为什么不用 `--recommend` 登录？

官方 `lark-cli auth login --recommend` 只包含约 148 个"常用"scope，**实际测试发现很多高频场景缺失**，例如：

| 缺失的 scope | 影响的命令 |
|:---|:---|
| `im:message.send_as_user` | 以用户身份发消息到群聊 |
| `search:message` | 跨群搜索消息 |
| `search:docs:read` | 搜索文档 |

更麻烦的是：OAuth 要求客户端**显式声明** scope。如果你先用 `--recommend` 登录，后来又需要新 scope，必须**重新走一遍 device flow**（再次打开链接、点击同意）。

**解决方案**：一次性导出当前已授权的全部 scope 到文件，以后直接引用：

```bash
# 首次配置时导出（只需做一次）
lark-cli auth status | python3 -c "import json,sys; d=json.load(sys.stdin); open('lark-lite-scopes.txt','w').write(d.get('scope',''))"

# 以后登录直接引用
lark-cli auth login --scope "$(cat lark-lite-scopes.txt)"
```

> `lark-lite-scopes.txt` 已包含在仓库中作为模板，不同租户可根据实际审批情况调整。

## 身份策略：User 为主，Bot 为辅

本 Skill 的核心设计是**以用户个人身份操作飞书**。这意味着：

- 读取到的群聊、日程、文档都是**你自己的**
- 发消息显示的是你**本人的头像和名字**
- Bot 身份仅在**明确需要**时切换

**必须用 Bot 身份的场景**（实测）：

| 操作 | 命令 | 原因 |
|:---|:---|:---|
| 下载消息中的图片 | `im +messages-resources-download --as bot` | User token 不支持图片下载 API |

**策略**：
1. 所有命令**默认不加 `--as`**（CLI 自动处理，通常解析为 user）
2. 如果 User 身份报错且提示"access token not support"，再尝试 `--as bot`
3. 不要无差别加 `--as bot`，否则读取到的资源可能是 Bot 视角（应用资源而非个人资源）

## 踩坑记录

### 1. 发消息到某些群被拒绝（230027）

`im +messages-send` 在某些群会报 `Permission denied [230027]`。这不是 scope 问题，而是**群聊级别的应用白名单限制**。需要管理员在飞书开放平台将该应用加入群的白名单。

### 2. 搜索 scope 需管理员审批

`search:message` 和 `search:docs:read` 属于敏感权限，**需要管理员在飞书开放平台手动审批**后才能获取。如果管理员未审批，相关搜索命令无法使用。

### 3. Docs v2 返回格式差异

`docs +fetch --api-version v2` 返回的是文档块格式（`data.document.content`，含 `<title>`、`<callout>`、`<img>`、`<grid>` 等标签），不是 markdown。`docs +create` 和 `docs +update` 也需用 `--content` 代替 `--markdown`，并指定 `--doc-format markdown` 或 `--doc-format xml`（默认 xml）。详细用法参考 references/doc.md。

### 4. Sheets 操作需先获取 sheet-id

`sheets +read` / `+write` / `+append` 需要 `--sheet-id` 参数，不是直接从 URL 猜的。先用 `sheets +info --url <URL>` 获取。

### 5. Drive 上传/下载需相对路径

`drive +upload --file` 和 `drive +download --output` 要求相对路径（如 `./file.pdf`），绝对路径会被拒绝。

### 6. 日历创建参数名不直观

`calendar +create` 用 `--summary` 而不是 `--title`，用 `--start/--end` 而不是 `--start-time/--end-time`。

## 与官方 Skill 的关系

| | lark-lite | 官方 Skill (larksuite/cli) |
|:---|:---|:---|
| 上下文占用 | ~5 KB | ~300+ KB（30 个 Skill） |
| 更新方式 | 手动维护 | `npx skills update -g` 自动同步 |
| 功能覆盖 | 高频快捷命令 + 路由 + references | 完整 API + 详细决策树 |
| 身份策略 | 用户身份优先，Bot 兜底 | 混合，依 Skill 而定 |
| 适用场景 | 日常快速操作 | 复杂/深度操作 |

## 更新 references

当官方 CLI 更新后，运行以下脚本同步 references 内容：

```bash
git clone --depth 1 https://github.com/larksuite/cli.git /tmp/lark-cli-official

cp /tmp/lark-cli-official/skills/lark-im/SKILL.md     references/im.md
cp /tmp/lark-cli-official/skills/lark-doc/SKILL.md    references/doc.md
cp /tmp/lark-cli-official/skills/lark-base/SKILL.md   references/base.md
cp /tmp/lark-cli-official/skills/lark-sheets/SKILL.md references/sheets.md
cp /tmp/lark-cli-official/skills/lark-drive/SKILL.md  references/drive.md
cp /tmp/lark-cli-official/skills/lark-calendar/SKILL.md references/calendar.md
cp /tmp/lark-cli-official/skills/lark-vc/SKILL.md     references/vc.md
cp /tmp/lark-cli-official/skills/lark-wiki/SKILL.md   references/wiki.md

# 更新 SKILL.md 中的 cliVersion，然后提交
```

## Credits

本项目基于 [飞书官方 CLI](https://github.com/larksuite/cli) 和官方 Skill 集合构建。lark-lite 是这些能力的轻量封装与路由层，所有实际 API 调用均通过 `lark-cli` 完成。

- **CLI 源码与完整 Skill 集合**：https://github.com/larksuite/cli
- **当前兼容 CLI 版本**：`1.0.39`

## 更新记录

| 日期 | CLI 版本 | 变更 |
|:---|:---|:---|
| 2026-05-23 | 1.0.39 | 初始版本，覆盖 IM / Doc / Sheets / Drive / Calendar / VC / Wiki / Base 高频命令 |
| 2026-05-23 | 1.0.39 | 修正快捷命令参数（实机测试后），添加身份策略和踩坑记录 |
