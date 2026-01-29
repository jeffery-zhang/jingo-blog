# 墙内使用 v2rayN 配置虚拟网卡并安装 moltbot/clawdbot 到 windows

## 使用 tun 模式启动 gateway

由于 moltbot gateway 服务不会走系统代理, 通常 v2ray 默认只对 http/socks 进行代理, 所以需要开启 tun(虚拟网卡) 模式

### v2rayN 开启 tun 模式配置

tun 模式的原理: 开启 tun, 由 singbox 前置 dns 解析和路由, 代理流量通过 xray 发出, 设置方法基于这种模式进行

首先更新 sing-box 内核, 然后开启 tun 模式

#### 参数设置

- 开启流量探测, 选择 http, tls, quic
- 建议打开 Route Only
- 开启 Mux 多路复用 关闭, 与 sing-box 不兼容
- sing-box Mux 多路复用协议不做配置(留空)

![参数设置1](/images/moltbot-windows-v2rayN-installation-01.png)
![参数设置2](/images/moltbot-windows-v2rayN-installation-02.png)

#### tun 模式设置

- 开启 Strict Route
- 启用额外监听端口(可选)

![tun 模式设置](/images/moltbot-windows-v2rayN-installation-03.png)

#### core 类型设置

一定要设置代理服务对应的真实的 core 类型

#### 路由设置

- xray 域名解析策略设置为 Asls
- sing-box 域名解析策略建议不做配置(留空)

![路由设置](/images/moltbot-windows-v2rayN-installation-04.png)

### 验证 tun 是否启用

设置好了之后可以测试重启服务, xray 和 sing-box 是同时运行的双核心, 启动时也是各自启动, log 文件也是分别存储的

- 查看与服务器的连接是否正常
- curl -s ipinfo.io 查看解析的 ip 地址是否已定位到代理服务器所在位置

## 安装 moltbot 和配置 telegram

- 打开 powershell 执行 iwr -useb https://molt.bot/install.ps1 | iex, 安装好后配置模型
- 选择配置 telegram 机器人
- 打开 telegram 搜索 botfather, 聊天框输入 /new 创建机器人, 并配置名称和 username
- 将生成的 token 复制到 moltbot, 然后就会启动 gateway 连接 telegram api
- 如果启动了 tun 模式, gateway 能够正常连接 telegram
- 与创建的机器人对话, 输入 /start, 弹出配对码
- 终端执行 clawdbot pairing approve telegram <配对码>, 即可连接成功
