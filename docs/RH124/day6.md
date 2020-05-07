# 第6天

## <font color=red>软件包管理</font>
* 源码编译安装
* rpm
* yum

### rpm 命令选项
* -i 安装
* -U 升级
* -e 卸载
* -v 详细信息
* -h 安装时显示进度
* -q 查询

以下选项均需与 -q 连用
* -a 所有已安装软件包
* -l 列出 rpm 软件包所有内容
* -c 列出 rpm 软件包配置文件
* -i 列出软件包信息
* -p 未安装的软件包
* --scripts 安装或删除之后执行的脚本
* --changelog 列出更改信息

### yum 命令
| 命令 | 作用  |
| --- | --- |
| yum list ［NAME-PATTERN］ | 按名称列出已安装和可用的软件包 |
| yum grouplist | 列出已安装和可用的组 |
| yum search KEYWORD | 按关键字搜索软件包 |
| yum info PACKAGENAME | 显示软件包的详细信息 |
| yum install PACKAGENAME | 安装软件包 |
| yum groupinstall GROUPNAME |  安装软件包组 |
| yum update |  更新所有软件包 |
| yum remove PACKAGENAME |  删除软件包 |
| yum history |  显示事务历史记录 |

### yum 源配置文件 /etc/yum.repos.d/*.repo
```
[repo_id]
name = repo_name
enabled = 1
baseurl = http://...  ftp://...  file://...
gpgcheck = 0    #若 gpgcheck 为 1，则需要指定 gpgkey = ...
```
**CentOS 可用 yum 源：**https://wiki.centos.org/zh/AdditionalResources/Repositories

### 源码编译安装
* 安装 编译器，一般是 `gcc`，也可以安装包组 `Development tools`
* 从软件官网下载源码包，通常是 tar 包
* 到解压目录下执行 `./configure` 进行预编译，`./configure --help` 查看预编译选项
* 补充依赖组件，再次预编译（多次），直到生成 makefile
* 执行 `make` 进行编译
* 执行 `make install` 进行安装

------

## <font color=red>文件系统</font>
#### I/O 设备
I/O设备大致分为两类：块设备和字符设备。
* 块设备将信息存储在固定大小的块中，每个块都有自己的地址。块设备的基本特征是每个块都能独立于其它块而读写。磁盘是最常见的块设备。
* 字符设备是指在I/O传输过程中以字符为单位进行传输的设备，例如键盘，打印机等。

[硬盘简介](disk.md)

#### 文件系统
**文件的系统是操作系统用于明确磁盘或分区上的文件的方法和数据结构，即在磁盘上组织文件的方法。**

* Windows 常用文件系统 NTFS，FAT32，exFAT
* Linux 常用文件系统 XFS，Ext4

#### 常用命令
* fdisk                          分区工具
* lsblk                          查看块设备
* blkid                          查看分区，UUID，文件系统
* mkfs                          格式化，写入文件系统
* df                               查看文件系统挂载及使用情况
* mount，umount    文件系统的挂载和卸载

#### 软链接和硬链接
>命令 `ln`硬链接    `ln -s`软链接

* 硬链接直接引用文件系统中的文件
* 所有硬链接权限，链接数，属主、属组，时间戳，文件内容保持一致
* 删除某个硬链接其它硬链接仍能访问改文件
* 硬链接不能跨文件系统
* 软链接又叫符号链接，指向现有文件或目录
* 删除该文件或目录后，软链接不可用
* 软链接可以跨文件系统

#### 文件查找
**locate**
* 从预生成的数据库中查找，数据库每天更新
* 当前新建文件或目录可能无法查找
* 使用 updatedb 可以及时更新数据库
* -i 忽略大小写，-n 指定行数

**find**
* 对整个文件系统实时搜索

| 选项 | 作用 |
| --- | --- |
| -i | 忽略大小写 |
| -name | 指定文件名 |
| -user，-uid | 指定用户名或 uid |
| -group，-gid | 指定组名或 gid |
| -perm | 指定权限，/ U，G，O 中至少匹配一位（或），- U，G，O 至少各匹配一位（与） |
| -size | 指定大小，+ 大于，- 小于 |
| -mmin | 指定时间（分钟），+ 分钟前，- 分钟内 |
| -type | 指定类型，f 文件，d 目录，l 软链接，b 块设备 |
| -links | 指定硬连接数，+ 大于，-小于 |

------

## <font color=red>虚拟化简介</font>
### 常见虚拟化
* xen 开源免费的虚拟化软件，基于硬件的完全分割，物理上有多少的资源就只能分配多少资源。
* kvm 开源免费的虚拟化软件，Linux 下的全功能虚拟化架构，甚至拥有独立的BIOS控制，所以对母服务器性能影响较大。
* vmware 是付费的虚拟化软件，VPS（含独立面板）系列产品授权费用非常昂贵。
* hyper-v 比较特别，是微软 windows 附带的虚拟化组件，如果你买了足够的授权，hyper-v 可以免费使用，Hyper-V 专为 Windows 定制，管理起来较为方便，虽然目前也支持 Linux，但性能损失比较严重。。

### qemu 与 kvm
* Qemu 是纯软件实现的虚拟化模拟器，几乎可以模拟任何硬件设备，我们最熟悉的就是能够模拟一台能够独立运行操作系统的虚拟机，虚拟机认为自己和硬件打交道，但其实是和 Qemu 模拟出来的硬件打交道，Qemu 将这些指令转译给真正的硬件。
* 正因为 Qemu 是纯软件实现的，所有的指令都要经 Qemu 过一手，性能非常低，所以，在生产环境中，大多数的做法都是配合 KVM 来完成虚拟化工作，因为 KVM 是硬件辅助的虚拟化技术，主要负责 比较繁琐的 CPU 和内存虚拟化，而 Qemu 则负责 I/O 虚拟化，两者合作各自发挥自身的优势，相得益彰。

### kvm 简介

* KVM 是开源软件，全称是kernel-based virtual machine（基于内核的虚拟机）。
* KVM 是 x86 架构且硬件支持虚拟化技术（如 intel VT 或 AMD-V）的Linux全虚拟化解决方案。
* KVM 包含一个为处理器提供底层虚拟化可加载的核心模块kvm.ko（kvm-intel.ko或kvm-AMD.ko）。
* KVM 需要一个经过修改的QEMU软件（qemu-kvm），作为虚拟机上层控制和界面。
* KVM能在不改变 linux 或 windows 镜像的情况下同时运行多个虚拟机，（它的意思是多个虚拟机使用同一镜像）并为每一个虚拟机配置个性化硬件环境（网卡、磁盘、图形适配器……）。
* 在 2.6.20 以上的内核均已包含了KVM核心。
* 要求cpu 必须支持虚拟化。
* 作为服务器性能很好，cpu 使用率控制很好，控制上比较简洁，功能比较丰富，可是图形能力较差。