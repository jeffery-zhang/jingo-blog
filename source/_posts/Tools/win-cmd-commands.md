---
title: Windows cmd 常用命令备忘
description: Windows cmd 常用命令备忘
date: 2022-04-13 14:33:44
categories:
  - Tools
tags:
  - Windows
  - cmd
  - 命令行
  - 常用命令
---

# Windows cmd 常用命令备忘

## cd 命令

```sh
D: # 进入D盘

cd /? # 使用帮助
cd \  # 进入当前盘根目录
cd C:\WINDOWS  # 进入当前盘其他目录
cd /d E:\dir  # 进入其他盘某个目录, 必须加 /d 参数
cd ..  # 跳转上一层目录
```

## 查看目录文件

```sh
dir  # 查看当前目录下的文件
dir /a  # 查看当前目录下的包含隐藏文件的所有文件
```

## 创建和删除目录

```sh
md test_dir  # 创建空目录
mkdir test_dir  # 创建空目录
rd /q/s test_dir  # 删除目录, /q 表示安静模式, /s 将会删除目录树
rmdir /q/s test_dir  # 删除目录, /q 表示安静模式, /s 将会删除目录树
```

## 文件处理

```sh
copy path\to\file path\to\file # 复制一个文件到另一个位置
move path\to\file path\to\file # 移动一个文件到另一个位置
del path\to\file # 删除一个文件
```

## 结束进程

```sh
TASKKILL

/P [password] 为提供的用户上下文指定密码。如果忽略，提示输入。
/F 指定要强行终止的进程。
/FI filter 指定筛选进或筛选出查询的的任务。
/PID process id 指定要终止的进程的PID。
/IM image name 指定要终止的进程的映像名称。通配符 '*'可用来指定所有映像名。
/T Tree kill: 终止指定的进程和任何由此启动的子进程。
/? 显示帮助/用法。
```

## 查看网络连接状态

```sh
netstat

-a或--all：显示所有连线中的Socket；
-A<网络类型>或--<网络类型>：列出该网络类型连线中的相关地址；
-c或--continuous：持续列出网络状态；
-C或--cache：显示路由器配置的快取信息；
-e或--extend：显示网络其他相关信息；
-F或--fib：显示FIB；
-g或--groups：显示多重广播功能群组组员名单；
-h或--help：在线帮助；
-i或--interfaces：显示网络界面信息表单；
-l或--listening：显示监控中的服务器的Socket；
-M或--masquerade：显示伪装的网络连线；
-n或--numeric：直接使用ip地址，而不通过域名服务器；
-N或--netlink或--symbolic：显示网络硬件外围设备的符号连接名称；
-o或--timers：显示计时器；
-p或--programs：显示正在使用Socket的程序识别码和程序名称；
-r或--route：显示Routing Table；
-s或--statistice：显示网络工作信息统计表；
-t或--tcp：显示TCP传输协议的连线状况；
-u或--udp：显示UDP传输协议的连线状况；
-v或--verbose：显示指令执行过程；
-V或--version：显示版本信息；
-w或--raw：显示RAW传输协议的连线状况；
-x或--unix：此参数的效果和指定"-A unix"参数相同；
--ip或--inet：此参数的效果和指定"-A inet"参数相同。
```

## find

```sh
find

/V         显示所有未包含指定字符串的行。
/C         仅显示包含字符串的行数。
/N         显示行号。
/I         搜索字符串时忽略大小写。
/OFF[LINE] 不要跳过具有脱机属性集的文件。
"string" 指定要搜索的文本字符串。
[drive:][path]filename 指定要搜索的文件。

如果没有指定路径，FIND 将搜索在提示符处键入
的文本或者由另一命令产生的文本。

netstat --ano | find ":8" # 使用管道符模糊查询
```
