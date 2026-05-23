# IM Interactive Card

飞书消息卡片（interactive）JSON Schema 参考。通过 `lark-cli im +messages-send --msg-type interactive --content '<json>'` 发送。

## 根结构

```json
{
  "config": { "wide_screen_mode": true },
  "header": { "title": { "tag": "plain_text", "content": "标题" }, "template": "blue" },
  "elements": [ ... ]
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `config` | object | 否 | 卡片全局配置 |
| `header` | object | 否 | 卡片头部（标题 + 主题色） |
| `elements` | array | 否 | 内容元素列表，按顺序渲染 |

## Config

```json
{ "wide_screen_mode": true }
```

- `wide_screen_mode` (bool): `true` 为宽屏模式，`false` 为窄屏模式

## Header

```json
{
  "title": { "tag": "plain_text", "content": "卡片标题" },
  "template": "blue"
}
```

- `title` (object): 标题文本，支持 `plain_text` 或 `lark_md`
- `template` (string): 主题色。可选值：`blue`（蓝）、`wathet`（浅蓝）、`turquoise`（青）、`green`（绿）、`yellow`（黄）、`orange`（橙）、`red`（红）、`carmine`（洋红）、`violet`（紫）、`purple`（紫）、`indigo`（靛）、`grey`（灰）、`default`（默认灰）

## Elements

### div — 文本块

最常用的内容元素，支持 Markdown 格式。

```json
{
  "tag": "div",
  "text": { "tag": "lark_md", "content": "**加粗** 和 [链接](https://...)" }
}
```

- `text.tag`: `plain_text`（纯文本）或 `lark_md`（支持 Markdown）
- `text.content`: 文本内容。`lark_md` 支持 `**bold**`、`*italic*`、`[text](url)`、`` `code` `` 等语法

### hr — 分割线

```json
{ "tag": "hr" }
```

### img — 图片

```json
{
  "tag": "img",
  "img_key": "img_v2_xxx",
  "alt": { "tag": "plain_text", "content": "描述文字" },
  "preview": true
}
```

- `img_key`: 通过 `lark-cli drive +upload` 或 `lark-cli images create` 上传后获取
- `preview`: `true` 时支持点击图片预览

### action — 按钮组

```json
{
  "tag": "action",
  "actions": [
    {
      "tag": "button",
      "text": { "tag": "plain_text", "content": "确认" },
      "type": "primary",
      "value": { "key": "confirm" },
      "confirm": {
        "title": { "tag": "plain_text", "content": "确认操作？" },
        "text": { "tag": "plain_text", "content": "此操作不可撤销" }
      }
    }
  ]
}
```

- `type`: `default`（默认）、`primary`（主色）、`danger`（危险红）
- `value`: 点击后回调携带的数据（需配置回调 URL）
- `confirm`: 二次确认弹窗（可选）

### note — 备注/脚注

```json
{
  "tag": "note",
  "elements": [
    { "tag": "plain_text", "content": "备注：" },
    { "tag": "plain_text", "content": "这是一条备注信息" }
  ]
}
```

### column_set — 多列布局

```json
{
  "tag": "column_set",
  "flex_mode": "none",
  "background_style": "default",
  "columns": [
    {
      "tag": "column",
      "width": "weighted",
      "weight": 1,
      "elements": [
        { "tag": "div", "text": { "tag": "lark_md", "content": "左列" } }
      ]
    },
    {
      "tag": "column",
      "width": "weighted",
      "weight": 1,
      "elements": [
        { "tag": "div", "text": { "tag": "lark_md", "content": "右列" } }
      ]
    }
  ]
}
```

- `flex_mode`: `none`（不换行）、`stretch`（拉伸）、`flow`（流式）
- `width`: `auto`（自适应）、`weighted`（权重分配）、固定像素如 `100px`

## 完整示例

```bash
lark-cli im +messages-send --chat-id "oc_xxx" --msg-type interactive --content '{
  "config": { "wide_screen_mode": true },
  "header": {
    "title": { "tag": "plain_text", "content": "任务提醒" },
    "template": "blue"
  },
  "elements": [
    { "tag": "div", "text": { "tag": "lark_md", "content": "**任务：** 完成 Skill 文档更新\n**截止：** 2026-05-24" } },
    { "tag": "hr" },
    {
      "tag": "action",
      "actions": [
        { "tag": "button", "text": { "tag": "plain_text", "content": "标记完成" }, "type": "primary", "value": { "action": "done" } },
        { "tag": "button", "text": { "tag": "plain_text", "content": "查看详情" }, "type": "default", "value": { "action": "view" } }
      ]
    }
  ]
}'
```

## 注意事项

- `--content` 接收的是 JSON 字符串，需用单引号包裹整个 JSON，内部使用双引号
- 卡片内容总大小限制约 30KB，超出会发送失败
- `img_key` 需先上传图片到飞书获取，不能直接用外部 URL
- 按钮的 `value` 回调需要应用配置事件订阅和回调地址才能生效；仅发送卡片无需回调配置

## 官方参考

- [飞书开放平台 — 消息卡片概述](https://open.feishu.cn/document/uAjLw4CM/ukzMukzMukzM/feishu-cards/card-introduce)
