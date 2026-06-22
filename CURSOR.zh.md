# 在 Cursor 中使用本仓库

本项目包含一个 **Cursor 项目规则**，因此受 Karpathy 启发的行为指南在你在此处工作时会自动生效。

## 在本仓库中

1. 在 Cursor 中打开该文件夹。
2. 规则 [`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc) 已通过 `alwaysApply: true` 提交，因此你无需额外安装步骤。
3. 在 Cursor 中，你可以在 **设置 → 规则**（或项目规则界面）中确认，`karpathy-guidelines` 应该会出现。

## 在其他项目中使用相同的指南

**Cursor（推荐）：** 将 `.cursor/rules/karpathy-guidelines.mdc` 复制到该项目的 `.cursor/rules/` 目录中（如果需要，请创建文件夹）。根据需要调整或与现有规则合并。

**其他工具：** 如果某个工具只支持根指令文件，请将 [`CLAUDE.zh.md`](CLAUDE.zh.md) 复制到该项目中（或将其内容合并到你现有的指令中）。

## 可选：个人 Agent 技能

如果你希望将相同的内容作为可复用的技能存放在 `~/.cursor/skills` 下，请使用 [`skills/karpathy-guidelines/SKILL.zh.md`](skills/karpathy-guidelines/SKILL.zh.md)。你可以将其复制或符号链接到你的个人技能目录中；使用你用于其他技能的任何布局。

## Claude Code vs Cursor

- **Claude Code：** 通过插件市场安装，详见 [`README.md`](README.md) 中的说明；该插件通过本仓库暴露技能。按项目使用也可以依赖 `CLAUDE.zh.md`。
- **Cursor：** 按照上述方式使用已提交的 `.cursor/rules/` 文件。Cursor 默认不会读取 `.claude-plugin/` 或 `CLAUDE.md`。

## 给贡献者的说明

当你修改四个原则时，请保持 **[`CLAUDE.zh.md`](CLAUDE.zh.md)** 和 **[`.cursor/rules/karpathy-guidelines.zh.mdc`](.cursor/rules/karpathy-guidelines.zh.mdc)** 同步。如果已发布的技能/插件文本应该匹配，请同时更新 **[`skills/karpathy-guidelines/SKILL.zh.md`](skills/karpathy-guidelines/SKILL.zh.md)**。
