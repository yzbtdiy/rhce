# 第5天

## <font color=red>日志</font>

### systemd-journald 和 rsyslog
- 进程和操作系统内核需能够为发生的事件记录日志
- 日志内容可用于系统审计和故障排除
- RHEL7 的日志系统由`systemd-journald`和 `rsyslog` 两个服务提供
- RHEL7 中，`systemd` 的日志保存在 `/run/log/journal` 中，系统重启就会清除。通过新建 `/var/log/journal` 目录，日志会自动记录到这个目录中，并永久存储
- `rsyslog` 服务随后根据优先级排列日志信息，将它们写入到 `/var/log`目录中永久保存
- `systemd-journald` 是一种改进的日志管理服务，是 `syslog` 的补充，收集来自内核、启动过程早期阶段、标准输出、系统日志，守护进程启动和运行期间错误的信息

```
systemd ---> systemd-journald ---> ram DB ---> rsyslog ---> /var/log
service daemon ---> rsyslog ---> /var/log
```

### logger -p

> -p， --priority priority_level

```
指定输入消息日志级别，优先级可以是数字或者指定为 " facility.level" 的格式。默认级别是 "user.notice"。
       facility：是用来定义由谁产生的日志信息：那个软件、子系统运行过程中产生的日志信息。
             auth              用户授权
             authpriv          授权和安全
             cron              计划任务
             daemon            系统守护进程
             kern              与内核有关的信息
             lpr               与打印服务有关的信息
             mail              与电子邮件有关的信息
             news              来自新闻服务器的信息
             syslog            由 syslog 生成的信息
             user              用户的程序生成的信息，默认
             ftp               与 ftp 有关 的信息
             uucp              由 uucp 生成的信息
             local0~7          用来定义本地策略

     level：是用来定义记录什么类型的日志信息。是应用程序产生的所有信息都把它记录到日志 文件中呢，还是只记录该应用程序的错误日志信息等等。
             emerg(0)          系统不可用
             alert(1)          需要立即采取动作
             crit(2)           严重状况
             error(3)          非常严重错误状况
             warning(4)        警告状况
             notice(5)         正常但要注意
             info(6)           信息性事件
             debug(7)          调试级别消息
```

![log](images\log.jpg)

### systemd 日志持久化
```
mkdir /var/log/journal
chown root:systemd-journal /var/log/journal
chmod 2755 /var/log/journal
killall -USER1 systemd-journald
```

## <font color=red>压缩归档 tar</font>

| 选项 | 作用 |
| --- | --- |
| -c   | 创建存档             |
| -z   | 使用 gzip 压缩方式（.tgz 或 .tar.gz）   |
| -j   | 使用 bzip2 压缩方式（.tar.bz2）    |
| -J   | 使用 xz 压缩方式（.tar.xz）       |
| -v   | 详细信息，显示存档或提取的的文件    |
| -f   | 要操作的存档文件名      |
| -x   | 提取存档内容           |
| -t   | 列出存档内容           |
| -p   | 保留文件和目录的权限，不去除umask    |
| -C   | 指定解压到的目录        |
| -P   | 压缩时保留绝度路径 |

压缩率：gzip < bzip2 < xz
压缩速度：gzip > bzip2 > xz

## <font color=red>远程复制文件</font>

>scp

```
scp 本地文件 远程用户@远程主机:/保存位置
scp 远程用户@远程主机:/文件路径 本地保存位置
```
选项：
* -r 递归，复制目录使用
* -p 保留时间和权限

>sftp

```
sftp root@server0
>cd /tmp       #cd 服务端 /tmp 目录
>ls            #查看服务端 /tmp 内容
>lcd /etc      #cd 客户端 /etc 目录
>put passwd    #上传 passwd 文件
>lcd /boot     #cd 客户端 /boot 目录
>mkdir grub2   #服务端创建上传目录同名目录
>put -r grub2  #上传 grub2 目录
>ls            #查看服务端 /tmp 内容
>lcd /tmp      #cd 客户端 /tmp 目录
>lls           #查看客户端 /tpm 目录内容
>get passwd    #从服务端下载 passwd 文件
>get -r grub2  #从服务端下载 grub2 目录
>lls           #查看客户端 /tmp 目录内容
```
选项：
* -r 递归，上传或下载目录使用，上传目录必须在服务端口同名目录
* -p 保留时间和权限

>rsync

```
rsync 本地目录 远程用户@远程主机:/保存位置
rsync 远程用户@远程主机:/目录路径 本地保存位置
```
选项：
* -av    -a 存档模式，-v 详细信息

-a 代表：
-r 递归，-l 同步符号链接，-p 保留权限，-t 保留时间戳，-g 保留组所有权，-o 保留所有者，-D 同步设备文件