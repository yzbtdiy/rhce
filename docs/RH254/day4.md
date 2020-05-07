# 第4天

## <font color=red>远程块存储 iSCSI</font>
### server
```
[root@server ~]# yum -y install targetcli
[root@server ~]# targetcli
	>/backstores/block create netdisk1 /dev/sdb
    >/iscsi create iqn.2019-06.com.example:server
    >/iscsi/iqn.2019-06.com.example:server/tpg1/acls create iqn.2019-06.com.example:desktop
    >/iscsi/iqn.2019-06.com.example:server/tpg1/luns create /backstores/server.lv1
    >/iscsi/iqn.2019-06.com.example:server/tpg1/portals delete 0.0.0.0 3260
    >/iscsi/iqn.2019-06.com.example:server/tpg1/portals create 192.168.3.11 3260
    >saveconfig
    >exit
[root@server ~]# systemctl restart target
[root@server ~]# systemctl enable target
[root@server ~]# firewall-cmd --add-service=iscsi-target
[root@server ~]# firewall-cmd --add-service=iscsi-target --permanent
```
### desktop
```
[root@desktop ~]# yum -y install iscsi-initiator-utils
[root@desktop ~]# vim /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2019-06.com.example:desktop（ACL）
[root@desktop ~]# iscsiadm -m discovery -t st -p 192.168.3.11
192.168.3.11:3260,1 iqn.2019-06.com.example:server
[root@desktop ~]# systemctl restart iscsi iscsid
[root@desktop ~]# iscsiadm -m node -T iqn.2019-06.com.example:server -l
[root@desktop ~]# systemctl enable iscsi iscsid
```

## <font color=red>邮件服务器</font>
> 常见的邮件协议：

* 简单邮件传输协议（Simple Mail Transfer Protocol，SMTP）：用于发送和中转发出的电子邮件，占用服务器的25/TCP端口。
* 邮局协议版本3（Post Office Protocol 3）：用于将电子邮件存储到本地主机，占用服务器的110/TCP端口。
* Internet消息访问协议版本4（Internet Message Access Protocol 4）：用于在本地主机上访问邮件，占用服务器的143/TCP端口。

> 电子邮件基本概念：

* MUA（Mail User Agent）接收邮件所使用的邮件客户端，使用IMAP或POP3协议与服务器通信
* MTA（Mail Transfer Agent） 通过SMTP协议发送、转发邮件
* MDA（Mail Deliver Agent）将MTA接收到的邮件保存到磁盘或指定地方，通常会进行垃圾邮件及病毒扫描
* MRA（Mail Receive Agent）负责实现IMAP与POP3协议，与MUA进行交互

* 常用的MUA有：outlook、thunderbird、mutt
* 常用的MTA服务有：sendmail、postfix
* 常用的MDA有：procmail、dropmail
* 常用的MRA有：dovecot

### server
#### 安装软件
```
[root@server0 ~]# yum -y install postfix dovecot bind
```
#### 配置 DNS 解析
```
##配置端口监听和创建解析域
[root@server0 ~]# vim /etc/named.conf
listen-on port 53 { any; };
listen-on-v6 port 53 { any; };

allow-query     { any; };
        
zone "server.com" IN {
        type master;
        file "server";
};

zone "desktop.com" IN {
        type master;
        file "desktop";
};
[root@server0 ~]# cp -p /var/named/named.localhost /var/named/server
[root@server0 ~]# cp -p /var/named/named.localhost /var/named/desktop
[root@server0 ~]# vim /var/named/server
$TTL 1D
@       IN SOA server.com. root.server.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        IN NS    ns.server.com.
ns      IN A     172.25.0.11
mail    IN A     172.25.0.11
@       IN A     172.25.0.11
        IN MX 10 mail.server.com.
[root@server0 ~]# vim /var/named/desktop
$TTL 1D
@       IN SOA desktop.com. root.desktop.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        IN NS    ns.desktop.com.
ns      IN A     172.25.0.10
mail    IN A     172.25.0.10
@       IN A     172.25.0.10
        IN MX 10 mail.desktop.com.
```
#### 检测是否存在语法错误
```
[root@server ~]# named-checkconf
[root@server ~]# named-checkzone server.com /var/named/server
[root@server ~]# named-checkzone desktop.com /var/named/desktop
```
#### 修改本地 DNS
```
[root@server ~]# nmcli con modify "System eth0" ipv4.dns 172.25.0.11 ipv4.ignore-auto-dns yes
[root@server ~]# systemctl restart NetworkManager
```
#### 配置 postfix
```
[root@server ~]# hostnamectl set-hostname server.example.com
[root@server ~]# postconf -e 'myhostname = server.example.com'
[root@server ~]# postconf -e 'mydomain = mail.server.com'
[root@server ~]# postconf -e 'myorigin = $mydomain'
[root@server ~]# postconf -e 'inet_interfaces = all'
[root@server ~]# postconf -e 'mydestination = $myhostname, $mydomain, localhost'
```
#### 配置 dovecot
```
[root@server ~]# vim /etc/dovecot/dovecot.conf
protocols = imap pop3 lmtp
disable_plaintext_auth = no
login_trusted_networks = 172.25.0.0/24
[root@server ~]# vim /etc/dovecot/conf.d/10-mail.conf
mail_location = mbox:~/mail:INBOX=/var/mail/%u
[root@server ~]# mkdir -p mail/.imap/INBOX
[root@server ~]# su - student
[student@server ~]$ mkdir -p mail/.imap/INBOX
[root@server ~]# systemctl restart postfix dovecot
```
#### 防火墙开放服务
```
[root@desktop ~]# firewall-cmd --add-service=smtp --add-service=imap --add-service=pop3 --add-service=dns
[root@desktop ~]# firewall-cmd --add-service=smtp --add-service=imap --add-service=pop3 --add-service=dns --permanent
```

### desktop
#### 安装软件
```
[root@desktop ~]# yum -y install postfix dovecot
```
#### 修改本地 DNS
```
[root@desktop ~]# nmcli con modify "System eth0" ipv4.dns 172.25.0.11 ipv4.ignore-auto-dns yes
[root@desktop ~]# systemctl restart NetworkManager
```
#### 配置 postfix
```
[root@desktop ~]# hostnamectl set-hostname desktop.example.com
[root@desktop ~]# postconf -e 'myhostname = desktop.example.com'
[root@desktop ~]# postconf -e 'mydomain = mail.desktop.com'
[root@desktop ~]# postconf -e 'myorigin = $mydomain'
[root@desktop ~]# postconf -e 'inet_interfaces = all'
[root@desktop ~]# postconf -e 'mydestination = $myhostname, $mydomain, localhost'
```
#### 配置 dovecot
```
[root@desktop ~]# vim /etc/dovecot/dovecot.conf
protocols = imap pop3 lmtp
disable_plaintext_auth = no
login_trusted_networks = 172.25.0.0/24
[root@desktop ~]# vim /etc/dovecot/conf.d/10-mail.conf
mail_location = mbox:~/mail:INBOX=/var/mail/%u
[root@desktop ~]# mkdir -p mail/.imap/INBOX
[root@desktop ~]# su - student
[student@desktop ~]$ mkdir -p mail/.imap/INBOX
[root@desktop ~]# systemctl restart postfix dovecot
```
#### 防火墙开放服务
```
[root@desktop ~]# firewall-cmd --add-service=smtp --add-service=imap --add-service=pop3
[root@desktop ~]# firewall-cmd --add-service=smtp --add-service=imap --add-service=pop3 --permanent
```

### 测试
```
[root@desktop ~]# echo 'test from desktop' | mail -s 'desktop to server' root@mail.server.com
[root@server ~]# mail
[root@server ~]# echo 'test from server' | mail -s 'server to desktop' root@mail.desktop.coms
[root@desktop ~]# mail
```
