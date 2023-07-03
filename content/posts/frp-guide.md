---
title: "Frp 内网穿透安装说明文档"
date: 2022-12-22T00:14:41+08:00
draft: false
---

## 序
最近买了一台 Intell Nuc 电脑打算放在家里面当作一个工作站使用，对于我自己而言有一下几个需求：

1. Nuc 会放置在家里面，需要能支持创建虚拟机
2. 某些服务或者虚拟机要支持能够暴露给外网环境使用

针对以上两点，也就是需要对 Nuc 安装虚拟机软件，同时需要一个内网穿透的软件。对于前者好说，虚拟机软件有很多；对于后者，今天分享下使用 frp 软件的使用方法。 
![](https://raw.githubusercontent.com/Idiot-Alex/picgo-repo/main/storage/blog/202307040021373.png)

## 1. 准备工作
frp 是一个提供内网穿透方案的软件，简单来讲就是一个网络中转的工具，类似中介的角色。

想要正常使用 frp，需要提前准备好这几点：

1. 一台能公网访问的服务器，比如阿里云、腾讯云的云服务器
2. 至少一台局域网内的电脑，并且该电脑需要能访问公网

我使用的是阿里云的云服务器（CentOS），配置不高，也就 1 核 2 G；局域网内的电脑就是上面提到的放置在家里面的 Nuc，我在 Nuc 里面安装好了 CentOS 系统，然后在 CentOS 里面安装了 Virtual Box Manage 虚拟机托管软件，并且创建了两台虚拟机，一台 Windows，一台 CentOS。

假设性条件：
云服务器 IP：101.101.101.101

下面是我的要求：

1. 能通过外网访问 Nuc 宿主机 CentOS 的 CockPit 管理页面
2. 能通过外网远程连接 Nuc 宿主机 CentOS 的 ssh
3. 能通过外网远程连接 Nuc 虚拟机 Windows
4. 能通过外网远程连接 Nuc 虚拟机 CentOS 的 ssh

简单解释下，CentOS 的 CockPit 是一款监控 CentOS 系统的软件，默认会使用 9090 端口作为 web 管理页面的入口；ssh 这里就不赘述了；Windows 需要远程访问，依赖 RDP 服务，默认端口是 3389。

说回 frp，首先需要下载 frp 的服务和 frp 客户端，可以通过 github 下载。
本文以 [https://github.com/fatedier/frp/releases/tag/v0.44.0](https://github.com/fatedier/frp/releases/tag/v0.44.0) 版本为例。

frp 软件支持的平台挺多，根据实际情况选择版本。
![](https://raw.githubusercontent.com/Idiot-Alex/picgo-repo/main/storage/blog/202307040021965.png)
> 需要注意的是，每个软件包里面都包含了 frp 服务端和 frp 客户端软件，如果是不通系统之间做内网穿透，需要下载多个软件包。
> 
> 比如使用 CentOS 安装 frp 服务端，使用 Windows 安装 frp 客户端，就需要下载  _linux_amd 版本和 _windows_amd 版本，分别使用里面的 frps 文件和 frpc 文件。

## 2. 安装 frp 服务端
frp 服务端需要安装到公网可访问的云服务器上。

frp 服务端只需要两个文件：

- frps 
- frps.ini

安装步骤：

1. 下载 frp 软件，解压到云服务器 CentOS 系统上，我的安装位置是 /usr/local/frp_0.44.0_linux_amd64
2. 使用 vim 编辑 frps.ini 配置文件，写入下面内容：
```shell
[common]
# 绑定端口，默认 7000
bind_port = 17000
# frp 服务 web 控制台端口，默认 7500
dashboard_port = 17500
# frp 服务 web 控制台用户名
dashboard_user = admin
# frp 服务 web 控制台密码
dashboard_pwd = 123455
# 用于 frp 服务端和客户端验证的 token
token = qwertyuiodfghjkertyughj

```

3. 启动 frp 服务端软件，使用下面命令启动：
```shell
./frps -c ./frps.ini
```

4. 访问该服务器 17500 端口，输入用户名密码即可进入下面类似页面：

![](https://raw.githubusercontent.com/Idiot-Alex/picgo-repo/main/storage/blog/202307040024137.png)

到此为止，frp 服务端就已经配置好了，接下来我们配置 frp 客户端。
## 3. 安装 frp 客户端
在云服务器上安装好 frp 服务端之后，我们还需要安装 frp 客户端，然后使用 frp 来一一完成我上面的需求。

frp 客户端的安装方式和 frp 服务端一样，同样只需要两个文件：

- frpc
- frpc.ini

服务启动方式也一样：
```shell
./frpc -c ./frpc.ini
```

主要是 frpc.ini 配置文件不一样，下面是针对不同的需求配置不同的内容。
### 1. 实现外网访问 Nuc 宿主机 CentOS 的 CockPit 管理页面
针对这个需求，需要在我的 Nuc 宿主机上安装 frp 客户端，然后防火墙开放 9090 端口，最后在 frp 客户端配置文件上配置好 9090 端口的转发规则。

安装步骤：

1. 下载 frp 软件，解压到 Nuc 宿主机 CentOS 系统上，我的安装位置是 /usr/local/frp_0.44.0_linux_amd64
2. 使用 vim 编辑 frpc.ini 配置文件，写入下面内容：
```shell
[common]
# frp 服务端 IP
server_addr = 101.101.101.101
# frp 服务端口
server_port = 17000
# tls 配置
tls_enable = true
# 用于 frp 服务端和客户端验证的 token
token = qwertyuiodfghjkertyughj

# 访问 cockpit web 配置，名称随意，不冲突就好
[nuc web]
type = tcp
local_ip = 127.0.0.1
local_port = 9090
remote_port = 19090

```

3. 启动 frp 服务端软件，使用下面命令启动：
```shell
./frpc -c ./frpc.ini
```

启动完成之后，可以访问下云服务器部署的 frp 服务端 web 控制台看到新增的一个 TCP 代理，然后我们只需要开放云服务器上的 19090 端口之后就可以通过 http://101.101.101.101:19090/ 访问原本在 Nuc 宿主机 CentOS 系统的 Cockpit web 页面了。
![](https://raw.githubusercontent.com/Idiot-Alex/picgo-repo/main/storage/blog/202307040032675.png)
![](https://raw.githubusercontent.com/Idiot-Alex/picgo-repo/main/storage/blog/202307040032357.png)
### 2. 实现外网远程连接 Nuc 宿主机 CentOS 的 ssh
针对这个需求我是想通过访问云服务器（比如 IP 地址为 101.101.101.101）的 10022 端口，使用 ssh 协议直接连接到 Nuc 宿主机 CentOS 的服务器。

由于上面已经安装好 frp 客户端软件了，现在只需要修改 frp 客户端配置文件，然后重启 frp 客户端服务就可以。

配置步骤如下：

1. 进入 Nuc 宿主机 CentOS 系统 /usr/local/frp_0.44.0_linux_amd64 目录
2. 使用 vim 编辑 frpc.ini 配置文件，写入下面内容（ssh 部分为新增部分）：
```shell
[common]
# frp 服务端 IP
server_addr = 101.101.101.101
# frp 服务端口
server_port = 17000
# tls 配置
tls_enable = true
# 用于 frp 服务端和客户端验证的 token
token = qwertyuiodfghjkertyughj

# 访问 cockpit web 配置，名称随意，不冲突就好
[nuc web]
type = tcp
local_ip = 127.0.0.1
local_port = 9090
remote_port = 19090

# ssh 访问配置，新增内容
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 10022

```

3. 启动 frp 服务端软件，使用下面命令启动：
```shell
./frpc -c ./frpc.ini
```

同样的，需要保证 Nuc 宿主机 CentOS 系统开放了 22 端口，以及云服务器开放了 10022 端口。然后就可以使用 `ssh -p 10022 user@101.101.101.101`访问 Nuc 宿主机的 CentOS 系统了。

> frp 服务端的 web 控制页面也可以看到新的 TCP 代理配置。

### 3. 实现外网远程连接 Nuc 虚拟机 Windows
针对这个需求我是想通过访问云服务器（比如 IP 地址为 101.101.101.101）的 13389 端口，使用类似 RDP 的方式直接远程连接到 Nuc 宿主机 Windows 电脑。

安装步骤：

1. 下载 frp 软件，解压到 Nuc 虚拟机 Windows 系统上，我的安装位置是 C:\sorftware\frp_0.44.0_windows_amd64
2. 使用文本编辑软件编辑 frpc.ini 配置文件，写入下面内容：
```shell
[common]
# frp 服务端 IP
server_addr = 101.101.101.101
# frp 服务端口
server_port = 17000
# tls 配置
tls_enable = true
# 用于 frp 服务端和客户端验证的 token
token = qwertyuiodfghjkertyughj

# 访问 windows rdp 配置，名称随意，不冲突就好
[rdp]
type = tcp
local_ip = 127.0.0.1
local_port = 3389
remote_port = 13389

```

3. 启动 frp 服务端软件，使用下面命令启动：
```shell
./frpc.exe -c ./frpc.ini
```

4. 开放 Windows 系统远程桌面连接，默认端口号 3389

启动完成之后，可以访问下云服务器部署的 frp 服务端 web 控制台看到新增的一个 TCP 代理，然后我们只需要开放云服务器上的 13389 端口之后就可以通过类似 RDP 的软件访问 101.101.101.101:13389 远程连接原本在 Nuc 虚拟机 Windows 系统。

> 针对以上第 4 个需求与第 2 个需求一致，照猫画虎即可。

## 4. 服务自启
虽然到目前为止成功解决了我的需求，但是有一个问题——那就是 frp 服务不会随着系统的系统而自启动。这一点对虚拟机是不够友好的，毕竟虚拟机可能不会一直启动，需要启动的时候可能暂时没在家里而无法操作，这样的内网穿透就不算完整了。

所以针对这个问题，下面分别提供两种系统的服务自启方案。
### 1. Linux 系统服务自启动
Linux 系统 frp 服务自启比较好做，提供一个 xx.service 文件即可。

废话不多说，上步骤：

1. 在 `/etc/systemd/system/`目录新建文件 frps.service 或者 frpc.service，分别针对 frp 服务端和 frp 客户端
2.  在 frps.service 或者 frpc.service 文件中输入以下内容（注意 frp 安装目录）：
```shell
[Unit]
Description=frps daemon
After=syslog.target network.target
Wants=network.target

[Service]
Type=simple
ExecStart=/usr/local/frp_0.44.0_linux_amd64/frps -c /usr/local/frp_0.44.0_linux_amd64/frps.ini
Restart=always
RestartSec=1min

[Install]
WantedBy=multi-user.target
```
```shell
[Unit]
Description=frpc daemon
After=syslog.target network.target
Wants=network.target

[Service]
Type=simple
ExecStart=/usr/local/frp_0.44.0_linux_amd64/frpc -c /usr/local/frp_0.44.0_linux_amd64/frpc.ini
Restart=always
RestartSec=1min

[Install]
WantedBy=multi-user.target
```

3. 使用下面命令打开开启自启服务：
```shell
# frp 服务端自启
systemctl enable frps
# frp 客户端自启
systemctl enable frpc
# 其他命令
systemctl start frps
systemctl stop frps
systemctl restart frps
```
### 2. Windows 系统服务自启动
Windows 系统服务自启动会稍微麻烦一点，需要借助 NSSM 软件来完成服务注册。

NSSM 官网地址：[https://nssm.cc/download](https://nssm.cc/download)

操作步骤：

1. 先创建一个 bat 脚本，用来启动 frp 客户端服务，比如 frpc_start.bat
```shell
C:\software\frp_0.44.0_windows_amd64/frpc.exe -c C:\software\frp_0.44.0_windows_amd64/frpc.ini
```

2. 下载 NSSM 软件，解压下载的 zip 文件至本地目录，我的目录是 C:\software\nssm2.24
3. 打开 CMD 命令行窗口进入 NSSM 目录，输入下面命令（Windows 一般都作为 frp 客户端）：
```shell
# frpc 为服务名称
./nssm.exe install frpc
```

3. 输入完成之后，会弹出类似下面的窗口，选择第一步编写好的启动脚本，点击「Install service」

![](https://raw.githubusercontent.com/Idiot-Alex/picgo-repo/main/storage/blog/202307040034120.png)

4. 服务启动：通过以上操作后，即可以在本地计算机服务列表中看到 frpc 服务，选择自启动的方式即可。打开本地服务程序的方法为：按键 Win+R，输入 services.msc。

![](https://raw.githubusercontent.com/Idiot-Alex/picgo-repo/main/storage/blog/202307040035249.png)

> nssm 其他命令参考：
> 1. 创建一个服务名称为 servicename 的服务
nssm install servicename
> 2. 启动创建的servername服务
nssm start servicename
> 3. 停止创建的servername服务
nssm stop servicename
> 4. 重新启动创建的servername服务
nssm restart servicename
> 5. 删除创建的servername服务
nssm remove servicename


到此为止，关于 frp 的使用就结束了。
