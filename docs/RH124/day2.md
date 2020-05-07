# 第2天

## <font color=red>重定向</font>

### linux 的标准输入与输出

linux标准输入设备是键盘，标准输出设备是显示器，标准错误输出是显示器。

| 设备   | 设备名      | 文件描述符 | 类型         |
| ------ | ----------- | ---------- | ------------ |
| 键盘   | /dev/stdin  | 0          | 标准输入     |
| 显示器 | /dev/stdout | 1          | 标准输出     |
| 显示器 | /dev/stderr | 2          | 标准错误输出 |

### Linux 文件描述符（通道）
**标准输入文件(stdin)**：stdin的文件描述符为0，程序默认从 stdin 读取数据。

**标准输出文件(stdout)**：stdout 的文件描述符为1，程序默认向 stdout 输出数据。

**标准错误文件(stderr)**：stderr的文件描述符为2，程序会向 stderr 流中写入错误信息。

### 输入重定向与输出重定向
**输入重定向**：是指不使用系统提供的标准输入端口，而进行重新的指定。换言之，输入重定向就是不使用标准输入端口输入文件，而是使用指定的文件作为标准输入设备。

**输出重定向**：输出重定向就是指不使用linux默认的标准输出设备显示信息，而是指定某个文件做为标准输出设备来存储文件信息。

### tty 与 pts
**tty**一词源于 Teletypes，或者 teletypewriters，原来指的是电传打字机，是通过串行线用打印机键盘通过阅读和发送信息的东西，后来这东西被键盘与显示器取代，所以现在叫终端比较合适。 

但是如果我们远程telnet到主机或使用xterm时不也需要一个终端交互么？是的，这就是虚拟终端**pty**(pseudo-tty)

**pts**(pseudo-terminal slave)是pty的实现方法，与ptmx(pseudo-terminal master)配合使用实现pty。 终端是一种字符型设备，它有多种类型，通常使用tty来简称各种类型的终端设备。tty是Teletype的缩写。Teletype是最早出现的一种终端设备，很像电传打字机，是由Teletype公司生产的。

**pts/ptmx**(pts/ptmx结合使用，进而实现pty): pts(pseudo-terminal slave)是pty的实现方法，与ptmx(pseudo-terminal master)配合使用实现pty。

## <font color=red>vim 使用</font>

### 三种模式

![](images\vim-workmodel.png)

| 功能 | 模式 |
| --- | --- |
| 命令模式 | 此模式用于文件导航、剪切和粘贴以及简单命令。撤销、恢复和其他操作也从此模式中执行。 |
| 输入模式 | 此模式用于常规文本编辑。替换模式是插入模式的一种变体，可以替换而不是插入文本。 |
| EX模式 | 此模式用于保存、退出和打开文件，以及搜索、替换和其它更为复杂的操作。 |

### 基础操作
| 按键      | 作用               |
| -------- | ----------------- |
| h 或 (←) | 光标向左移动一个字符 |
| j 或 (↓) | 光标向下移动一个字符 |
| k 或 (↑) | 光标向上移动一个字符 |
| l 或 (→) | 光标向右移动一个字符 |
| w | 光标移至下一单词开头 |
| b | 光标移至上一单词开头 |
| ( | 光标移至单签句子或上一句子的开头 |
| ) | 光标移至下一句子的开头 |
| { | 光标移至当前或上一段落开头 |
| } | 光标移至下一段落开头 |
| ^ | 光标移至所在行行首 |
| $ | 光标移至所在行行尾 |
| gg | 光标移至文档第一行 |
| G | 光标移至文档最后一行 |
| yy   | 复制光标所在的那一行                     |
| nyy  | n 为数字，复制光标所在的向下 n 行          |
| dd   | 剪切光标所在的那一整行                    |
| ndd  | n 为数字，剪切光标所在的向下 n 行          |
| d^, d0 | 数字 0 ，剪切光标所在处，到该行首的字符      |
| d$, D  | 剪切光标所在处，到该行尾的字符              |
| cw | 替换光标位置到单词结尾的内容 |
| ciw | 替换光标所在位置单词 |
| caw | 替换光标所在位置单词，包括空白区 |
| cc | 光标所在行整行替换 |
| c^, c0 | 光标所在位置替换到行首 |
| c$, C | 光标所在位置替换到行尾 |
| ncc | n 为数字，替换光标所在的向下 n 行 |
| x, X | x 光标位置向后删除，X 光标位置向前删除 |
| p, P | p 为将已复制的数据在光标下一行贴上，P 则为贴在光标上一行 |
| i, I | i 为从光标所在处输入；I 为在所在行的第一个非空格符处输入 |
| a, A | a 为从光标所在的下一个字符处输入； A 为从光标所在行的最后一个字符处输入 |
| o, O | o 为在光标所在的下一行输入； O 为在光标所在上一行输入 |
| r, R | r 只取代光标所在的字符一次；R 一直取代光标所在的字符，直到按下 ESC 为止 |
| v, V, ctrl+v | v 字符可视化编辑；V 行可视化编辑；ctrl+v 块可视化编辑 |
| :w | 保存更改并留在编辑器中 |
| :w filename | 另存为一个文件 |
| :wq | 保存并退出 |
| :x | 保存当前文件，然后退出 |
| :q | 退出当前文件，没有未保存更改 |
| :q! | 退出当前文件，忽略未保存更改 |

## <font color=red>用户管理</font>

### 用户和组
**用户**：系统中每个进程（运行程序）都由一个特定用户运行。每个文件归一个特定用户所有。对文件和目录的访问收到用户身份的限制。与运行进程相关连得用户可以确定该进程可访问的文件和目录。每个用户都有自己的用户名和编号（UID），操作系统内部是通过 UID 来区分用户的。

* UID 为 0 的是 root 用户。
* UID 为 1-200 为系统用户，静态分配给红帽的系统进程的。
* UID 为 201-999 为系统用户，供文件系统中没有自己的文件的系统进程使用，通常在安装软件时从池中动态分配，限制对应程序仅访问它们正常运行所需的资源。
* UID 1000+ 供普通用户使用。

**组**：组是用户的集合，组也有组名和编号（GID）。

**主要组**：每个用户有一个主要组，用户创建的新文件归其主要组所有。默认情况新建用户的主要组是与用户同名的新建组。

**补充组**：用户可以是 0 个或多个补充组的成员。补充组成员身份用于确保用户具有对系统中某些文件及其它资源的访问权限。

### 用户管理命令
| 命令 | 选项及作用 |
| --- |--- |
| id | 查看用户基本信息，不加参数显示当前用户 |
| su - | 切换用户，使用 - 在切换用户的同时更新环境变量，不加参数切换 root 用户  |
| useradd | 创建用户，-u 指定 uid，-g 指定基本组，-G 指定扩展组，-s 指定默认 shell，-N 不创建同名基本组，-d 指定家目录 |
| groupadd | 创建组，-g 指定 gid |
| usermod | 修改用户，-s 修改shell，-u 修改 uid，-g 修改基本组，-G 修改扩展组，-aG 添加扩展组，-L 锁定，-U 解锁，-m -d 指定新家目录并转移用户数据 |
| userdel | 删除用户，-r 删除用户家目录 |
| groupmod | 修改组，-g 修改 gid，-n 修改组名|
| chown | 修改文件或目录所有者及所属组，-R 递归 |
| chgrp | 修改文件或目录所属组，-R 递归 |

### sudo
sudo 是 linux 系统管理指令，是允许系统管理员让普通用户执行一些或者全部的 root 命令的一个工具减少了 root 用户的登录 和管理时间，同样也提高了安全性。

配置文件：/etc/sudoers
格式：**user**  **host**  **run_as**  **command**

* user：用户或组，组要以%开头
* host：主机名
* run_as 作为哪个用户执行，常用 root 或 ALL
* command 让用户或组运行的一个或多个根级别命令

------

## 练习
>1.创建用户useradmin，logviewer，diskmanager

>2.通过visudo赋予useradmin创建删除用户的权限，赋予logviewer查看日志的权限，赋予diskmanager分区权限

>3.通过useradmin用户创建用户user1，user2，user3，user4
user1是logviewer组成员，可以查看日志
user2用户的shell为/sbin/nologin
user3的家目录位于/user3
user4的uid为10086

```
[root@server0 ~]# useradd useradmin
[root@server0 ~]# useradd logviewer
[root@server0 ~]# useradd diskmanager
[root@server0 ~]# echo redhat | passwd --stdin useradmin 
[root@server0 ~]# echo redhat | passwd --stdin logviewer 
[root@server0 ~]# echo redhat | passwd --stdin diskmanager
[root@server0 ~]# visudo
useradmin    ALL=(root)    NOPASSWD: /usr/sbin/useradd,/usr/sbin/userdel
%logviewer   ALL=(root)    NOPASSWD: /usr/bin/cat
diskmanager  ALL=(root)    NOPASSWD: /usr/sbin/fdisk
[root@server0 ~]# su - useradmin
[useradmin@server0 ~]$ sudo useradd -G logviewer user1
[useradmin@server0 ~]$ sudo useradd -s /sbin/nologin user2
[useradmin@server0 ~]$ sudo useradd -d /user3 user3
[useradmin@server0 ~]$ sudo useradd -u 10086 user4
[useradmin@server0 ~]$ exit
[root@server0 ~]# echo redhat | passwd --stdin user1
[root@server0 ~]# echo redhat | passwd --stdin user2
[root@server0 ~]# echo redhat | passwd --stdin user3
[root@server0 ~]# echo redhat | passwd --stdin user4
[root@server0 ~]# su - logviewer
[logviewer@server0 ~]$ cat /var/log/messages
[logviewer@server0 ~]$ sudo cat /var/log/messages
[logviewer@server0 ~]$ exit
[root@server0 ~]# su - user1
[user1@server0 ~]$ cat /var/log/messages
[user1@server0 ~]$ sudo cat /var/log/messages
[user1@server0 ~]$ exit
[root@server0 ~]# su - diskmanager
[diskmanager@server0 ~]$ fdisk /dev/vdb 
[diskmanager@server0 ~]$ sudo fdisk /dev/vdb
```