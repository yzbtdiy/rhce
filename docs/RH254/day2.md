# 第2天
## <font color=red>DNS 服务</font>

> <font color=red>主服务器</font>

#### 安装软件包
```
[root@server ~]# yum -y install bind bind-untils
```
#### 对外提供服务，默认只解析 localhost
```
[root@server ~]# vim /etc/named.conf
... ...
 listen-on port 53 { any; };
 listen-on-v6 port 53 { any; };
 allow-query     { any; };
... ...

zone "google.com" IN {
        type master;
        file "google";
};

zone "3.128.192.in-addr.arpa" IN {
        type master;
        file "google-reverse";
}
```
#### 创建解析数据文件
```
[root@server ~]# chmod g+s /var/named
[root@server ~]# cd /var/named
[root@server named]# cp named.empty google
[root@server named]# cp named.empty google-reverse
[root@server named]# vim google 
$TTL 3H
@       IN SOA  ns.google.com. root.google.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      ns.google.com.
ns      A       192.168.3.11
@       A       192.168.3.11
www     A       192.168.3.11
[root@server named]# vim google-reverse
$TTL 3H
@       IN SOA  ns.google.com. root.google.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      ns.google.com.
ns      A       192.168.3.11
11      PTR     www.google.com.
11      PTR     google.com.
```
#### 检测配置文件语法
```
[root@server named]# named-checkconf
[root@server named]# named-checkzone google.com google
[root@server named]# named-checkzone 3.128.192.in-addr.arpa google-reverse
[root@server named]# systemctl restart named
[root@server named]# systemctl enable named
```
#### 客户端修改本地 DNS
```
echo "nameserver 192.168.3.11"  > /etc/resolv.conf
```

> <font color=red>主从服务器</font>

* 主服务器添加从服务器参数<br/>
`allow-transfer { SLAVE_IP; };`<br/>
`allow-update { none; };`<br/>
`also-notify { SLAVE_IP; };`

* 从服务器创建与主服务器相同的 zone<br/>
`type slave;`<br/>
`masters { MASTER_IP };`<br/>
`masterfile-format text;`
`allow-notify { MASTER_IP; };`

## <font color=red>WEB 服务器</font>
#### 安装 apache
```
[root@server0 ~]# yum -y install httpd
```
#### 拷贝虚拟主机模板
```
[root@server0 ~]# cp /usr/share/doc/httpd-2.4.6/httpd-vhosts.conf  /etc/httpd/conf.d/vhosts.conf
```
#### 创建网站目录并添加首页
```
[root@server0 ~]# echo "<h1>this is /var/www/html</h1>" > /var/www/html/index.html
[root@server0 ~]# mkdir /www
[root@server0 ~]# echo "<h1>this is /www</h1>" > /www/index.html
[root@server0 ~]# mkdir /var/www/webapp
[root@server0 ~]# echo "<h1>this is /var/www/webapp</h1>" > /var/www/webapp/index.html
```
#### 配置虚拟主机
```
[root@server0 ~]# vim /etc/httpd/conf.d/vhosts.conf
```
######  1.基于 domain

```
<VirtualHost *:80>
    DocumentRoot "/var/www/html"
    ServerName server0.example.com
</VirtualHost>

<VirtualHost *:80>
    DocumentRoot "/www"
    ServerName www0.example.com
</VirtualHost>

<VirtualHost *:80>
    DocumentRoot "/var/www/webapp"
    ServerName webapp0.example.com
</VirtualHost>
<Directory "/www">
    Require all granted
</Directory>

[root@server0 ~]# firewall-cmd --add-port=80/tcp
[root@server0 ~]# firewall-cmd --add-port=80/tcp --permanent
[root@server0 ~]# semanage fcontext -a -t httpd_sys_content_t '/www(/.*)?'
[root@server0 ~]# restorecon -Rv /www/
```

###### 2.基于 ip
```
<VirtualHost *:80>
    DocumentRoot "/var/www/html"
    ServerName 172.25.0.11
</VirtualHost>
<VirtualHost *:80>
    DocumentRoot "/www"
    ServerName 172.25.0.12
</VirtualHost>
<VirtualHost *:80>
    DocumentRoot "/var/www/webapp"
    ServerName 172.25.0.13
</VirtualHost>
<Directory "/www">
    Require all granted
</Directory>
```
###### 3.基于 port
```
<VirtualHost *:80>
    DocumentRoot "/var/www/html"
    ServerName 172.25.0.11
</VirtualHost>
listen 888
<VirtualHost *:888>
    DocumentRoot "/www"
    ServerName 172.25.0.11
</VirtualHost>
listen 8888
<VirtualHost *:8888>
    DocumentRoot "/var/www/webapp"
    ServerName 172.25.0.11
</VirtualHost>
<Directory "/www">
    Require all granted
</Directory>

[root@server0 ~]# semanage port -a -t http_port_t -p tcp 888
[root@server0 ~]# semanage port -a -t http_port_t -p tcp 8888
[root@server0 ~]# firewall-cmd --add-port=888/tcp --add=port=8888/tcp
[root@server0 ~]# firewall-cmd --add-port=888/tcp --add=port=8888/tcp --permanent
```
#### 修改配置文件务必重启服务应用新配置
```
[root@server0 ~]# systemctl restart httpd
```

#### http 的 SSL/TLS 加密（https）
* X.509是一个标准，规范了公开秘钥认证、证书吊销列表、授权凭证、凭证路径验证算法等。
* X.509证书包含三个文件：key，csr，crt
* key是服务器上的私钥文件，用于对发送给客户端数据的加密，以及对从客户端接收到数据的解密
* csr是证书签名请求文件，用于提交给证书颁发机构（CA）对证书签名
* crt是由证书颁发机构（CA）签名后的证书，或者是开发者自签名的证书，包含证书持有人的信息，持有人的公钥，以及签署者的签名等信息

1. 第一步：生成客户端的密钥，即客户端的公私密钥对，且要保证私钥只有客户端自己拥有。
2. 第二步：用客户端的私钥加密客户端客户端自身的信息(国家、机构、域名、邮箱等)，生成 csr 证书请求文件。其中客户端的公钥和客户端信息是明文保存在证书请求文件中的，而客户端私钥的作用是对客户端公钥及客户端信息做签名，自身是不包含在证书请求中的。然后把证书请求文件发送给CA机构。
3. 第三步：CA机构接收到客户端的证书请求文件后，首先校验其签名，然后审核客户端的信息，最后CA机构使用自己的私钥为证书请求文件签名，生成证书文件，下发给客户端。此证书就是客户端的身份证，来表明用户的身份。


```
# ca 生成私钥和公钥
openssl genrsa -out /etc/pki/CA/private/cakey.pem 2048
openssl rsa -in /etc/pki/CA/private/cakey.pem -pubout -out /etc/pki/CA/private/ca.pub 
# ca 自签名证书
openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3650
# index.txt 是索引文件，用与匹配证书编号
# serial 是证书序列号文件，只在首次生成证书时赋值
touch /etc/pki/CA/{index.txt,serial}
echo 01 > /etc/pki/CA/serial
# ca 为服务器签发证书
openssl ca -in server.csr -out server.crt -cert /etc/pki/CA/cacert.pem -keyfile /etc/pki/CA/private/cakey.pem -days 365
```
```
# 服务器生成私钥和公钥
openssl genrsa -out /etc/pki/tls/private/server.key 2048
openssl rsa -in /etc/pki/tls/private/ca.key -pubout -out /etc/pki/tls/private/ca.pub
openssl req -new -key /etc/pki/tls/private/server.key -out server.csr
```


## 客户端
> 客户端在浏览器中通过访问网站内容，注意 http 和 https
