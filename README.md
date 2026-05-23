# lark-lite

飞书轻量入口 Skill。只保留最核心、最稳定的内容，避免加载 30 个官方 Skill 占用大量上下文。

## 安装

```bash
npx skills add kjx-talesofai/lark-lite -g
```

## 设计原则

- **纯路由**：SKILL.md 只包含认证指引、功能路由表和常用快捷命令
- **不引用官方 Skill 内容**：避免随官方更新而过时
- **手动维护快捷命令**：只保留最稳定、最常用的命令，自己控制更新节奏

## 与官方 Skill 的关系

| | lark-lite | 官方 Skill |
|:---|:---|:---|
| 上下文占用 | ~5 KB | ~300+ KB（30 个 Skill） |
| 更新方式 | 手动维护 | `npx skills update -g` 自动同步 |
| 功能覆盖 | 高频快捷命令 + 路由 | 完整 API 覆盖 |
| 适用场景 | 日常快速操作 | 复杂/深度操作 |

## 官方仓库

- **CLI 源码与完整 Skill 集合**：https://github.com/larksuite/cli
- **当前兼容 CLI 版本**：`1.0.39`

## 更新记录

| 日期 | CLI 版本 | 变更 |
|:---|:---|:---|
| 2026-05-23 | 1.0.39 | 初始版本，覆盖 IM / Doc / Sheets / Drive / Calendar 高频命令 |

## 按需安装官方 Skill

当 lark-lite 无法满足需求时，可以单独安装对应的官方 Skill：

```bash
npx skills add larksuite/cli -g -s lark-im
npx skills add larksuite/cli -g -s lark-doc
npx skills add larksuite/cli -g -s lark-base
# ... 按需添加
```
