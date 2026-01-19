---
title: Claude Code + PackyAPI 使用教程
description: 详细介绍如何安装配置 Claude Code，并通过 PackyAPI 服务使用 Claude 的 AI 能力
date: 2025-01-19 10:52:28
categories:
  - Ai
tags:
  - Claude
  - PackyAPI
  - AI 工具
---

# Claude Code + PackyAPI 使用教程

Claude Code 是 Anthropic 推出的命令行工具，支持直接在终端中使用 Claude 的 AI 能力。本文将介绍如何安装配置 Claude Code，并通过 PackyAPI 服务进行使用。

## 安装 Claude Code

在 Windows 系统上，可以通过 winget 快速安装：

```sh
winget install Anthropic.ClaudeCode
```

安装完成后，打开终端输入 `claude` 即可验证安装是否成功。

## 创建 PackyAPI Token

[PackyAPI 官网](https://www.packyapi.com/)

PackyAPI 是一个提供 Anthropic API 服务的平台，相比直接使用 Anthropic 官方 API，提供了更便捷的访问方式。

### 创建步骤

1. 进入 PackyAPI 控制台
2. 点击进入「令牌管理」页面

![令牌管理](/images/claude-code-and-packyapi-usage-01.png)

3. 点击「添加令牌」按钮
4. 输入自定义的令牌名称（便于识别）
5. **重要：选择 `cc` 分组**（这个分组专门用于 Claude Code）
6. 点击提交

创建成功后，系统会自动生成一个 API Key，请务必妥善保存。

![生成 API Key](/images/claude-code-and-packyapi-usage-02.png)

> **注意**：API Key 只在创建时显示一次，关闭窗口后将无法再次查看，请务必复制保存。

## 配置 Claude Code

Claude Code 的配置文件位于用户目录下，需要创建和修改几个 JSON 文件。

### 步骤一：跳过向导

1. 按 `Win + R`，输入 `%userprofile%` 进入当前用户目录
2. 打开 `.claude.json` 文件（如果没有则创建）
3. 搜索 `hasCompletedOnboarding` 键，将其值改为 `true`

```json
{
  "hasCompletedOnboarding": true
}
```

这一步的作用是跳过 Claude Code 的首次配置向导，直接使用我们的自定义配置。

### 步骤二：配置 API 提供者

1. 进入用户目录下的 `.claude` 文件夹（如果没有则创建）
2. 打开或创建 `config.json` 文件
3. 写入以下内容：

```json
{
  "primaryApiKey": "PackyApi"
}
```

这里指定了使用 PackyAPI 作为主要的 API 提供者。

### 步骤三：设置环境变量

1. 在 `.claude` 目录下打开或创建 `settings.json` 文件
2. 写入以下内容：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://www.packyapi.com",
    "ANTHROPIC_AUTH_TOKEN": "sk-xxxxxxx"
  }
}
```

**重要**：将 `sk-xxxxxxx` 替换为你在上一步创建令牌时生成的真实 API Key。

## 验证配置

配置完成后，可以通过以下步骤验证是否设置成功：

1. 按 `Win + R`，输入 `cmd` 打开命令行终端
2. 输入 `claude` 命令
3. 如果配置正确，你将看到 Claude Code 的交互界面

配置成功后，不仅可以直接在终端使用 `claude` 命令，还可以在 VS Code 中安装 Claude Code 插件，享受更丰富的 IDE 集成体验。

## 常见问题

### Q: 为什么需要选择 `cc` 分组？

A: PackyAPI 根据不同的分组提供不同的 API 端点，`cc` 分组专门为 Claude Code 配置，使用其他分组可能导致连接失败。

### Q: API Key 丢失了怎么办？

A: 目前 PackyAPI 不支持查看已生成的 API Key，只能删除旧令牌并重新创建新的。请务必妥善保存您的 API Key。

### Q: 在 VS Code 中如何使用？

A: 在 VS Code 扩展商店搜索并安装「Claude Code」插件，配置完成后即可在编辑器中直接使用 Claude 的 AI 能力。

### Q: 配置文件位置错误怎么办？

A: 确保 `.claude` 文件夹位于用户根目录（如 `C:\Users\你的用户名\.claude`），而不是其他位置。

## 注意事项

1. **安全性**：不要将 API Key 提交到公共代码仓库，建议将其添加到 `.gitignore` 文件中
2. **稳定性**：第三方 API 服务可能会出现不稳定的情况，建议有备选方案
3. **成本**：使用 Claude API 会产生费用，请关注 PackyAPI 的计费规则，避免意外产生高额费用

## 总结

通过 Claude Code 和 PackyAPI 的组合，我们可以方便地在命令行环境中使用 Claude 的强大 AI 能力。无论是代码编写、问题解答还是文档生成，都能显著提升工作效率。

希望这篇教程对你有所帮助！
