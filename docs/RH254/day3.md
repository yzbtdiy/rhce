# 第3天

## <font color=red>FTP 文件传输</font>
### server
#### 安装 vsftpd
```
[root@server ~]# yum -y install vsftpd
```
> 配置文件 `/etc/vsftpd/vsftpd.conf`

#### 1.  匿名访问
* anonymous_enable=YES        开启匿名访问       
* anon_umask=022        匿名用户上传的 umask 值
* anon_upload_enable=YES        允许匿名用户上传文件
* anon_mkdir_write_enable=YES        允许匿名用户创建目录
* anon_other_write_enable=YES        允许匿名用户修改或删除目录

#### 2. 本地用户
* anonymous_enable=NO        禁止匿名访问
* local_enable=YES        开启本地本地用户访问
* write_enable=YES        是否有写权限
* local_umask=022	本地用户上传文件的 umask 值
* userlist_enable=YES	使用用户列表，名单文件为 user_list（可能是白名单，也可能是黑名单）和 ftpuser（始终是黑名单）
* userlist_deny=YES 	开启用户作用名单文件功能
* **userlist_enable=YES, userlist_deny=YES**<br/>
user_list 为黑名单，禁止 user_list 中用户登录  
* **userlist_enable=YES, userlist_deny=NO**<br/>
user_list 为白名单，仅允许 user_list 中的用户登录
* chroot_local_user=YES        用户禁锢在主目录，不允许跳转上级目录
* chroot_list_enable=YES        是否使用禁锢用户列表
* **chroot_local_user=YES, chroot_list_enable=YES**<br/>
chroot_list_file 为白名单，不受限制
* **chroot_local_user=NO, chroot_list_enable=YES**<br/>
chroot_list_file 为黑名单，受到限制
* **chroot_local_user=YES, chroot_list_enable=NO**<br/>
所有用户都受到限制，chroot_list_file 无作用
* **chroot_local_user=NO, chroot_list_enable=NO**<br/>
所有用户都不受限制，chroot_list_file 无作用

**如果用户受到禁锢，那么要添加 allow_writeable_chroot=YES，或者取消主目录的写权限**

#### 3. 虚拟用户
* anonymous_enable=NO        禁止匿名访问
* local_enable=YES        开启本地本地用户访问
* guest_enable=YES        开启虚拟用户访问
* guest_username=virtftp        指定虚拟用户使用的系统用户
* pam_service_name=ftpvuser       指定PAM文件
* user_config_dir=/etc/vsftpd/vusers_conf        指定虚拟用户配置文件
* allow_writeable_chroot=YES        允许写入禁锢的目录

```
[root@server ~]# useradd virtftp -s /sbin/nologin
[root@server ~]# vim /etc/vsftpd/vuser.list
virtuser1
redhat
virtuser2
redhat
[root@server ~]# db_load -T -t hash /etc/vsftpd/vuser.list /etc/vsftpd/vuser.db
[root@server ~]# vim /etc/pam.d/ftpvuser
auth       required     pam_userdb.so db=/etc/vsftpd/vuser
account    required     pam_userdb.so db=/etc/vsftpd/vuser
[root@server ~]# mkdir /etc/vsftpd/vuser_conf
[root@server ~]# vim /etc/vsftpd/vuser_conf/virtuser1
local_root=/home/virtftp/virtuser1
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
[root@server ~]# mkdir /home/virtftp/virtuser1
[root@server ~]# chmod 777 /home/virtftp/virtuser1
```
### desktop 使用 ftp 客户端工具访问

## <font color=red>NFS 共享</font>

### server
#### 安装软件包 nfs-utils
```
[root@server ~]# yum -y install nfs-utils
```

#### 编辑 `/etc/exports` 文件，添加共享
```
[root@server ~]# vim /etc/exports
/nfsshare 192.168.3.*(OPTIONS)
```

常用 OPTIONS 包括：
* `ro` 只读<br/>
`rw` 读写 
* `root_squash`    映射 root 为 nfs 匿名用户<br/>
`no_root_squash`    不映射 root 为 nfs 匿名用户<br/>
`all_squash`    映射所有用户为 nfs 匿名用户<br/>
`no_all_squash`   不映射 nfs 匿名用户，访问用户与本地匹配（UID）

* `sync` 同步<br/>
`async` 异步

#### 重启服务，开放防火墙服务
```
[root@server ~]# systemctl restart nfs-server
[root@server ~]# systemctl enable nfs-server
[root@server ~]# firewall-cmd --add-service=mountd --add-service=rpc-bind --add-service=nfs
[root@server ~]# firewall-cmd --add-service=mountd --add-service=rpc-bind --add-service=nfs --permanent
[root@server ~]# exportfs -rfv
```

### desktop
#### 临时挂载
```
[root@desktop ~]# yum -y install nfs-utils
[root@desktop ~]# showmount -e 192.168.3.11
[root@desktop ~]# mkdir /mnt/nfsmount
[root@desktop ~]# mount -t nfs 192.168.3.11:/nfsshare /mnt/nfsmount
```
#### 开机自动挂载
```
[root@desktop ~]# vim /etc/fstab
192.168.3.11:/nfsshare    /mnt/nfsmount    nfs    defaults    0 0
```
#### autofs 挂载
```
[root@desktop ~]# yum -y install autofs
[root@desktop ~]# vim /etc/auto.master
/netshare（挂载目录上级目录） /etc/netauto（映射文件）
[root@desktop ~]# vim /etc/nfsauto
nfsmount -fstype=nfs,rw 192.168.3.11:/nfs
```

## <font color=red>samba 共享</font>
### server
#### 安装 samba 
```
[root@server ~]# yum -y install samba
```
#### 新建共享
```
[root@server ~]# vim /etc/samba/smb.conf
workgroup = WORKGROUP
hosts allow = 127. 192.168.3.

[public]    ##共享名
comment = Public Stuff  ##说明
path = /smbpub  ##路径
public = no    ##公开，可匿名访问
writable = yes   ##是否可写
printable = no   ##是否是打印机，如果共享目录必须为no
write list = smbrw  ##可写名单
read list = smbro   ##只读名单
browseable = yes  ##可浏览
```
#### 创建共享目录，设置权限
```
[root@server ~]# mkdir /smbpub
[root@server ~]# chmod 777 /smbpub
[root@server ~]# semanage fcontext -a -t samba_share_t '/smbpub(/.*)?'
[root@server ~]# restorecon -Rv /smbpub/
```
#### 创建 samba 用户
**samba 用户必须在系统中存在，通过 pdbedit 和 smbclient 均可创建 samba 用户**

```
[root@server ~]# useradd -s /sbin/nologin smbrw
[root@server ~]# useradd -s /sbin/nologin smbro
[root@server ~]# (echo redhat;echo redhat) | pdbedit -a smbrw
[root@server ~]# (echo redhat;echo redhat) | smbpasswd -a smbro
[root@server ~]# systemctl restart smb nmb
[root@server ~]# systemctl enable smb nmb
```
#### 开放防火墙
```
[root@server ~]# firewall-cmd --add-service=samba --add-service=samba-client
[root@server ~]# firewall-cmd --add-service=samba --add-service=samba-client --permanent
```
### desktop
#### 临时挂载
```
[root@desktop ~]# yum -y install cifs-utils samba-client
[root@desktop ~]# mkdir /mnt/smbmount
[root@desktop ~]# mount -t cifs -o username=smbrw,password=redhat //192.168.3.11/public /mnt/smbmount
```
#### 多用户挂载，开机自动挂载
**用只读用户挂载，不同的用户可以更新不同凭证**<br/>
**/etc/fstab所有用户可读，将 samba 用户和密码保存到 root 家目录可提高安全性**
```
[root@desktop ~]# vim /etc/fstab
//192.168.3.11/public /mnt/smbpmnt cifs credentials=/root/smbpass,multiuser,sec=ntlmssp 0 0
[root@desktop ~]# echo "username=smbro"  >> /root/smbpass
[root@desktop ~]# echo "password=redhat"  >> /root/smbpass
[root@desktop ~]# mkdir /mnt/smbpub
[root@desktop ~]# mount -a

[user@desktop ~]# cifscreds add -u smbrw 192.168.3.11  ##更新用户凭证
```
#### autofs 挂载
```
[root@desktop ~]# yum -y install autofs
[root@desktop ~]# vim /etc/auto.master
/netshare（挂载目录上级目录） /etc/netauto（映射文件）
[root@desktop ~]# vim /etc/netauto
smbmount -fstype=cifs,username=smbro,password=redhat ://192.168.3.11/public
```
#### samba-client 上传下载
```
[root@desktop ~]# yum -y install samba-client
[root@desktop ~]# smbclient //172.25.0.11/public -U smbro%redhat
smb: \> lcd /etc
smb: \> put passwd
NT_STATUS_ACCESS_DENIED opening remote file \passwd

[root@desktop ~]# smbclient //172.25.0.11/public -U smbrw%redhat
smb: \> lcd /etc
smb: \> put passwd
putting file passwd as \passwd (2091.6 kb/s) (average 2091.8 kb/s)
```