- https://github.com/microsoft/WSL/releases 下载 Microsoft.WSL_2.6.3.0_x64_ARM64.msixbundle
- powershell 安装
- https://apps.microsoft.com/home?hl=en-US&gl=IN 搜索下载 ubuntu installer.exe
- 检查 cpu 虚拟化 systeminfo | findstr /i "虚拟化"
- - 基于虚拟化的安全性: 状态： 正在运行
    基本的虚拟化支持
    基本的虚拟化支持
    APIC 虚拟化
- - 需要去 bios 启用
- 检查 VMP 虚拟机平台 dism /online /get-featureinfo /featurename:VirtualMachinePlatform
- - 状态 : 已启用
- - dism /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
- 检查 Hyper-V systeminfo | findstr /i "Hyper-V":
- - Hyper-V 要求: 已检测到虚拟机监控程序。将不显示 Hyper-V 所需的功能。
- 检查 Hyper-V 自启动 bcdedit /enum | findstr -i hypervisorlaunchtype:
- - hypervisorlaunchtype Auto
- - bcdedit /set hypervisorlaunchtype auto

- 重启

如果有需要 wsl 可以重装到非 C 盘(如 D 盘)

- 打包到 D 盘: wsl --export Ubuntu D:\ProgramFiles\WSL\Ubuntu\Ubuntu.tar
- 卸载 Ubuntu: wsl --unregister Ubuntu
- 执行: wsl -l -v, 查看是否卸载成功
- 用 tar 包重新安装: wsl --import Ubuntu D:\ProgramFiles\WSL\Ubuntu D:\ProgramFiles\WSL\Ubuntu\Ubuntu.tar --version 2
- 执行: wsl -l -v, 查看是否重装成功
- 重新进入 wsl, 输入: df -h /, 查看是否已移动到 D 盘

启动 wsl

- 配置本地代理
- - ip route show | grep -i default | awk '{ print $3}' 获取本机 ip
- - export all_proxy = "protocal + ip + port"
- - 将上一段命令写入系统配置文件: nano ~/.bashrc, 添加到文件尾部
- - 执行: curl -s ipinfo.io, 查看 ip 地址是否变化
