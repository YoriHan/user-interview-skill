# 用户访谈纲要整理 · Claude Code Skill

从腾讯会议转写链接自动整理用户访谈记录，按结构化模版提炼提纲，自动编号上传到 GitHub。

## 功能

- 用 headless browser 抓取腾讯会议转写页面内容
- 按三段式模版整理：用户背景 / 产品使用体验表格 / 问题反馈与需求表格
- 自动读取 GitHub 已有文件，确定下一个编号（01、02、03…）
- 上传到 `sheet0/gtm` 的 `Launch/user interview/` 目录

## 使用方式

1. 将 `SKILL.md` 放入 `~/.claude/skills/user-interview/SKILL.md`
2. 在 Claude Code 中触发：

```
/user-interview
```

然后提供腾讯会议转写链接和受访者标识即可。

## 输出格式

每份访谈生成一个编号 Markdown 文件，例如 `01-Helio用户访谈-x.md`，包含：

- **用户背景**：身份、AI工具迁移路径、来 Helio 的动机
- **产品使用体验**：两列表格，每格编号列出细节（Helio配置 / 工作流 / Channel / HR / Calendar / Memory）
- **问题反馈与需求**：两列表格（Bug / 认知卡点 / 功能需求 / 留存与付费意愿）

## 目标仓库

- Repo: `sheet0/gtm`
- 路径: `Launch/user interview/`
- 命名: `NN-Helio用户访谈-[受访者标识].md`
