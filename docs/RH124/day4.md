# 第4天

## <font color=red>Systemd</font>
* systemd 进程 ID 为 1，是所有进程的父进程，负责激活系统中的其他服务。
* sytemctl 命令用于管理各种 systemd 对象，称为单元，可通过 ```systemctl -t help``` 查看所有类型

### 查看单元
>查看所有已加载单元状态

```
systemctl
```

>查看指定类型单元状态,--all 显示所有单元

```
systemctl -t service
```

>查看指定单元状态

```
systemctl status sshd
```

>判断单元是否活动

```
systemctl is-active sshd
```

>判断单元是否开机启动

```
systemctl is-enabled sshd
```

### 其它命令
| 操作 | 命令 |
| --- | --- |
| 查看状态 | systemctl status UNIT |
| 停止单元 | systemctl stop UNIT |
| 启动单元 | systemctl start UNIT |
| 重启单元 | systemctl restart UNIT |
| 重载配置 | systemctl reload UNIT |
| 屏蔽单元 | systemctl mask UNIT |
| 解除屏蔽 | systemctl unmask UNIT |
| 开机自启 | systemctl enable UNIT |
| 取消自启 | systemctl disable UNIT |
| 查看依赖 | systemctl list-dependencies UNIT |
**活动单元 restart 进程 ID改变，reload 进程 ID 不变**

## <font color=red>OpenSSH</font>

>SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。
>SSH提供两种级别的安全验证：

* 基于口令的安全验证（用户密码）
* 基于密匙的安全验证（公钥+私钥）

>椭圆曲线数字签名算法（Elliptic Curve Digital Signature Algorithm），简称ECDSA。
ECDSA key fingerprint 为主机的 ID 标识，用户初次连接无法确定主机真实性会进行询问，之后会保存在用户家目录下的 ```.ssh/known_hosts```文件中，下次连接不再询问，若主机 ID 发生改变则禁止连接。

### 基于口令的安全验证

>在远程主机上创建对用户，并设置密码即可（ssh 服务默认开机启动，且允许口令验证）

```
[root@server ~]# useradd sshuser
[root@server ~]# echo redhat | passwd --stdin root
```

>客户端连接远程主机需要输入密码

```
[root@desktop ~]# ssh sshuser@172.25.0.11
```

### 基于密匙的安全验证

>生成秘钥对

```
[root@desktop ~]# ssh-keygen -t rsa
```

>发送公钥到被管理主机

```
[root@desktop ~]# ssh-copy-id root@172.25.0.11
```

>免密码登陆

```
[root@desktop ~]# ssh root@172.25.0.11
```

### 安全策略

>禁用密码验证(需要提前准备秘钥认证)

```
[root@server ~]# vim /etc/ssh/sshd_config
PasswordAuthentication no
```

>仅允许使用秘钥进行身份验证（不能使用口令验证）

```
PermitRootLogin without-password
```

>禁止root用户直接登录（需要创建其他用户并设置密码，若之前禁用密码验证则需要准备秘钥认证）

```
[root@server ~]# vim /etc/ssh/sshd_config
PermitRootLogin no
```

### 其它命令
| 命令 | 选项及作用 |
| --- | --- |
| ssh | 远程登录主机，-X 启用 X11 图形化转发，-i 指定身份验证文件（私钥） |
| ssh-keygen | 生成秘钥对，-f 指定密钥对文件名，-t 指定加密算法 |
| ssh-copy-id | 发送公钥到远程主机，-i 指定身份验证文件（公钥）|

### 练习
* 在 foundationX 上使用 ssh-keygen 生成密钥对，使用 ssh-copy-id 发送至 desktopX 和 serverX然后使用 ssh 登陆 desktopX 和 serverx，执行以下操作，所有操作不能直接到虚拟机操作
* 在 desktopX 上使用 ssh-keygen -f 生成密钥，保存位置为 ~/sshkey/key
* desktopX 使用 ssh-copy-id -i 发送公钥至 root@serverX
* desktopX 使用 ssh -i 连接 root@serverX
* 修改 desktopX 的 /etc/ssh/ssh_config，添加 IdentityFile ~/sshkey/key
* desktopX 使用 ssh root@serverX 直接连接
* 修改 serverX 的 /etc/ssh/sshd_config，禁止root用户使用密码的登陆
PermitRootLogin without-password
* desktopX 使用 ssh-copy-id -i 发送公钥至 student@serverX
* 修改 serverX 的 /etc/ssh/sshd_config，禁止密码验证
PasswordAuthentication no
* 修改 serverX 的 /etc/ssh/sshd_config，禁止 root 登陆
PermitRootLogin no
* 至此，desktopX 可以使用 student 用户的登陆 serverX，root 用户无法直接登陆，但可以通过 su - 切换 root 用户


## <font color=red>网络配置</font>

### 图形化配置工具
* 右上角->用户名->setting->Network
* nmtui
* nm-connection-edit

### 命令行工具 nmcli
* device 网络设备，一般是网卡
* connection 网络连接，可以理解为网卡的配置
* 一个设备可以对应多个连接，但同时只能有一个连接处于激活状态
* 配置文件 ```/etc/sysconfig/network-scripts/ifcfg-*```

>查看设备

```
nmcli device
```

>查看连接

```
nmcli connection
```

>添加连接

```
##DHCP获取地址
nmcli con add con-name [连接名] ifname [设备名] type ethernet
##手工配置地址
nmcli con add con-name [连接名] ifname [设备名] type ethernet ip4 [IP]/[掩码] gw4 [网关]
```

>删除连接

```
nmcli con del [连接名]
```

>修改连接

```
##修改IP
nmcli con mod [连接名] ipv4.method manual ipv4.addresses "[IP]/[掩码] [网关]"
##添加dns
nmcli con mod [连接名] ipv4.dns [DNS_IP]
```

>启用连接

```
nmcli con up [连接名]
```

>断开连接

```
nmcli con down [连接名]
```

>连接网卡

```
nmcli dev connect [设备名]
```

>断开网卡

```
nmcli dev disconnect [设备名]
```

### 其它命令
| 操作 | 命令 |
| --- | --- |
| 测试连通 | ping，-c 指定次数 |
| 查看路由 | ip route，route -n |
| 查看端口 | ss，netstat，-n 显示接口和端口编号，-t 显示 tcp 套接字，-u 显示 udp 套接字，-l 显示侦听中的套接字，-a 显示所有套接字，-p 显示套接字进程 |
| 路由追踪 | traceroute，tracepath |