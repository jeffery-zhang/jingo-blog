---
title: 在 WSL 上部署 Clawdbot 完整指南
date: 2026-02-01 12:00:00
updated: 2026-02-01 12:00:00
tags:
  - WSL
  - Clawdbot
  - Telegram
  - Bot
  - Linux
categories:
  - 技术教程
  - AI 工具
description: 详细记录在 Windows WSL 环境中安装配置 Clawdbot（Moltbot）并与 Telegram 集成的完整流程，包括 WSL 安装、虚拟化配置、磁盘迁移和代理设置等关键步骤。
---

本文详细记录了在 Windows Subsystem for Linux (WSL) 环境中部署 Clawdbot（原 Moltbot）的完整过程，包括 WSL 的安装配置、虚拟化环境检查、磁盘迁移以及 Telegram 机器人的集成配置。

<!-- more -->

## 前言

Clawdbot 是一个强大的 AI 对话机器人工具，支持多平台集成。在 WSL 环境下部署可以充分利用 Linux 生态系统的优势，同时保持 Windows 桌面环境的便利性。本教程将带你完成从零开始的完整部署流程。

## 一、WSL 环境准备与安装

### 1.1 下载并安装 WSL

首先需要安装最新版本的 WSL。访问 [WSL GitHub Releases](https://github.com/microsoft/WSL/releases) 页面，下载最新的安装包：

- 下载文件：`Microsoft.WSL_2.6.3.0_x64_ARM64.msixbundle`
- 使用 PowerShell 管理员权限执行安装

```powershell
# 安装 WSL msixbundle 包
Add-AppxPackage Microsoft.WSL_2.6.3.0_x64_ARM64.msixbundle
```

### 1.2 安装 Ubuntu 发行版

从 [Microsoft Store](https://apps.microsoft.com/home?hl=en-US&gl=IN) 搜索并下载 Ubuntu 或直接下载 Ubuntu installer.exe 进行安装。推荐使用 Ubuntu 22.04 LTS 或更高版本以获得更好的兼容性。

### 1.3 虚拟化环境检查与配置

WSL 2 依赖于 CPU 虚拟化功能和 Windows 虚拟化平台，需要确保相关特性已正确启用。

#### 检查 CPU 虚拟化支持

打开命令提示符或 PowerShell，执行以下命令检查虚拟化状态：

```powershell
systeminfo | findstr /i "虚拟化"
```

正常输出应包含以下内容：

```
基于虚拟化的安全性: 状态： 正在运行
基本的虚拟化支持
APIC 虚拟化
```

如果虚拟化未启用，需要进入 BIOS/UEFI 设置界面，找到并启用以下选项（不同主板名称可能略有差异）：
- Intel CPU：Intel VT-x / Intel Virtualization Technology
- AMD CPU：AMD-V / SVM Mode

#### 启用 Windows 虚拟机平台

检查虚拟机平台（VMP）功能状态：

```powershell
dism /online /get-featureinfo /featurename:VirtualMachinePlatform
```

如果状态显示 "已禁用"，执行以下命令启用：

```powershell
dism /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

#### 检查 Hyper-V 状态

验证 Hyper-V 虚拟机监控程序是否正常运行：

```powershell
systeminfo | findstr /i "Hyper-V"
```

预期输出：

```
Hyper-V 要求: 已检测到虚拟机监控程序。将不显示 Hyper-V 所需的功能。
```

检查 Hyper-V 启动类型：

```powershell
bcdedit /enum | findstr -i hypervisorlaunchtype
```

应该显示：

```
hypervisorlaunchtype Auto
```

如果不是 Auto，执行以下命令设置：

```powershell
bcdedit /set hypervisorlaunchtype auto
```

**完成以上配置后，重启计算机使所有更改生效。**

## 二、WSL 磁盘迁移（可选）

默认情况下，WSL 会安装在 C 盘，如果 C 盘空间紧张或希望将其迁移到其他磁盘（如 D 盘），可以按照以下步骤操作。

### 2.1 导出现有 WSL 实例

```powershell
wsl --export Ubuntu D:\ProgramFiles\WSL\Ubuntu\Ubuntu.tar
```

这个过程会将整个 Ubuntu 文件系统打包成 tar 文件，可能需要几分钟时间，取决于系统大小。

### 2.2 卸载原有实例

```powershell
wsl --unregister Ubuntu
```

### 2.3 验证卸载结果

```powershell
wsl -l -v
```

确认 Ubuntu 已从列表中移除。

### 2.4 重新导入到目标位置

```powershell
wsl --import Ubuntu D:\ProgramFiles\WSL\Ubuntu D:\ProgramFiles\WSL\Ubuntu\Ubuntu.tar --version 2
```

### 2.5 验证迁移结果

```powershell
wsl -l -v
```

确认 Ubuntu 已成功重新注册，进入 WSL 后执行：

```bash
df -h /
```

检查文件系统挂载点，确认数据已迁移到 D 盘对应的虚拟磁盘。

## 三、WSL 网络代理配置

由于 Telegram API 在某些地区可能无法直接访问，需要配置代理以确保网络连接正常。

### 3.1 获取 Windows 主机 IP 地址

在 WSL 终端中执行：

```bash
ip route show | grep -i default | awk '{ print $3}'
```

这将返回 Windows 主机在 WSL 网络中的 IP 地址，通常是 `172.x.x.1` 格式。

### 3.2 配置系统代理环境变量

假设你的代理服务运行在 Windows 主机的 7890 端口（根据实际情况调整），执行：

```bash
export all_proxy="http://172.x.x.1:7890"
export ALL_PROXY="http://172.x.x.1:7890"
export http_proxy="http://172.x.x.1:7890"
export https_proxy="http://172.x.x.1:7890"
```

### 3.3 持久化代理配置

为了避免每次重启后重新配置，将代理设置写入 Bash 配置文件：

```bash
nano ~/.bashrc
```

在文件末尾添加以下内容（替换为实际的主机 IP 和端口）：

```bash
# Proxy configuration
HOST_IP=$(ip route show | grep -i default | awk '{ print $3}')
export all_proxy="http://${HOST_IP}:7890"
export ALL_PROXY="http://${HOST_IP}:7890"
export http_proxy="http://${HOST_IP}:7890"
export https_proxy="http://${HOST_IP}:7890"
```

保存后执行：

```bash
source ~/.bashrc
```

### 3.4 验证代理配置

执行以下命令检查当前网络出口 IP：

```bash
curl -s ipinfo.io
```

如果代理配置成功，返回的 IP 地址应该是代理服务器的地址，而非本地 IP。

## 四、安装 Moltbot 与配置 Telegram 机器人

### 4.1 安装 Moltbot

在 Windows PowerShell（非 WSL）中执行以下命令安装 Moltbot：

```powershell
iwr -useb https://molt.bot/install.ps1 | iex
```

安装完成后，按照提示配置所需的 AI 模型（如 OpenAI、Claude 等）。

### 4.2 创建 Telegram 机器人

1. 打开 Telegram，搜索 [@BotFather](https://t.me/botfather)
2. 在聊天框中输入 `/newbot` 创建新机器人
3. 按照提示设置机器人的显示名称（Display Name）
4. 设置机器人的用户名（Username），必须以 `bot` 结尾
5. 创建成功后，BotFather 会返回机器人的 API Token

示例交互：

```
You: /newbot
BotFather: Alright, a new bot. How are we going to call it?
You: My Clawdbot
BotFather: Good. Now let's choose a username for your bot.
You: my_clawdbot_bot
BotFather: Done! ... Use this token to access the HTTP API:
1234567890:ABCdefGHIjklMNOpqrsTUVwxyz
```

### 4.3 配置 Moltbot 连接 Telegram

选择配置 Telegram 机器人选项，将获取到的 API Token 粘贴到 Moltbot 配置中。Moltbot 会自动启动 Gateway 并连接到 Telegram API。

**注意**：如果你的网络环境需要代理访问 Telegram，确保 Windows 主机已启动代理工具（如 Clash、V2Ray 等）的 TUN 模式或系统代理模式，这样 Moltbot Gateway 才能正常连接。

### 4.4 配对激活机器人

1. 在 Telegram 中找到你刚创建的机器人，点击 "Start" 或发送 `/start`
2. 机器人会返回一个配对码（Pairing Code）
3. 在 WSL 终端中执行以下命令完成配对：

```bash
clawdbot pairing approve telegram <配对码>
```

成功后，你就可以通过 Telegram 与 Clawdbot 进行对话了。

## 五、常见问题与故障排查

### 5.1 WSL 无法启动

- 确认已启用虚拟化功能并重启
- 检查 Windows 更新，确保系统版本支持 WSL 2
- 尝试以管理员权限运行 `wsl --update`

### 5.2 Telegram 连接失败

- 检查代理配置是否正确
- 确认 Windows 防火墙未阻止代理端口
- 尝试在 WSL 中 `curl https://api.telegram.org` 测试连通性

### 5.3 Clawdbot 无响应

- 查看 Moltbot 日志排查错误信息
- 确认 API Key 配额未用尽
- 检查机器人 Token 是否正确

## 总结

通过以上步骤，你已经成功在 WSL 环境中部署了 Clawdbot 并与 Telegram 集成。这种部署方式结合了 Linux 的灵活性和 Windows 的易用性，为使用 AI 助手提供了便捷的本地化解决方案。

如果在部署过程中遇到问题，建议查阅官方文档或在相关社区寻求帮助。祝你使用愉快！

---

**参考资源**：
- [WSL 官方文档](https://learn.microsoft.com/zh-cn/windows/wsl/)
- [Moltbot 官网](https://molt.bot/)
- [Telegram Bot API 文档](https://core.telegram.org/bots/api)
