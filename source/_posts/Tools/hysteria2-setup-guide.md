---
title: "Hysteria2 代理服务器部署指南"
date: 2026-01-28
description: "在 VPS 上部署 Hysteria2 代理工具的详细步骤"
categories:
  - Tools
tags:
  - hysteria2
  - proxy
  - vps
  - setup
  - sni
---

# Hysteria2 代理服务器部署指南

本文档详细记录了在 VPS 上部署 Hysteria2 代理服务器的完整步骤。

## 环境要求

- Linux VPS（已安装 Ubuntu/CentOS 等主流发行版）
- Root 或 sudo 权限
- 防火墙配置能力

## 部署步骤

### 1. 准备工作

确认系统时间和日期正确：

```bash
timedatectl status
```

### 2. 下载安装脚本

使用 GitHub 上流行的安装脚本：

```bash
curl -fsSL https://raw.githubusercontent.com/TheyCallMeSecond/config-examples/main/Hysteria2/install.sh -o install.sh
chmod +x install.sh
```

### 3. 执行安装

运行安装脚本：

```bash
sudo ./install.sh
```

在安装过程中选择 Hysteria2 协议。

### 4. 配置服务

安装完成后，脚本会自动生成配置文件。默认配置信息如下：

- **端口号**: 20000 (UDP)
- **协议**: Hysteria2
- **密码**: pwd
- **SNI**: www.google.com

### 5. 防火墙设置

安装完成后，需要开放相应端口：

```bash
# Ubuntu/Debian
sudo ufw allow 20000/udp
sudo ufw reload

# CentOS/RHEL
sudo firewall-cmd --permanent --add-port=20000/udp
sudo firewall-cmd --reload
```

### 6. 服务管理

检查服务状态：

```bash
sudo systemctl status sh
```

重启服务：

```bash
sudo systemctl restart sh
```

### 7. 连接信息

#### IPv4 连接地址:

```
hy2://pwd@xx.xx.xx.xx:20000?insecure=1&sni=www.google.com#HY2
```

#### IPv6 连接地址:

```
hy2://pwd@[xx:xx:xx:xx::1]:20000?insecure=1&sni=www.google.com#HY2-V6
```

## 故障排除

### 检查服务状态

```bash
sudo systemctl status sh
```

### 查看服务日志

```bash
sudo journalctl -u sh -f
```

### 检查端口监听

```bash
sudo ss -tulnp | grep 20000
```

## 安全注意事项

- 定期更换密码
- 监控流量使用情况
- 保持软件版本更新
- 配置适当的防火墙规则

## 总结

Hysteria2 代理服务器已成功部署，当前配置可提供高效的网络代理服务。服务器已开放端口 20000/UDP 并启用了防火墙保护。
