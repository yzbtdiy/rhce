# 第4天
## <font color=red>Firewalld 防火墙</font>
### 查看规则
```
# 查看所有区域
firewall-cmd --get-zones
# 查看默认区域
firewall-cmd --get-default-zone
# 修改默认区域
firewall-cmd --set-default-zone=ZONE_NAME
# 查看默认区域规则
firewall-cmd --list-all
# 查看指定区域规则
firewall-cmd --list-all --zone=ZONE_NAME
# 查看所有区域规则
firewall-cmd --list-all-zones
# 查看所有防火墙可开放的服务
firewall-cmd --get-services
```

### 添加规则
#### 运行时生效
```
# 添加端口到指定区域
firewall-cmd --add-port=PORT/PROTOCOL --zone=ZONE_NAME
# 添加服务到指定区域
firewall-cmd --add-service=SERVICE_NAME --zone=ZONE_NAME
# 添加网络到指定区域
firewall-cmd --add-source=IP_ADDR/NETMASK --zone=ZONE_NAME
# 添加网卡设备到指定区域
firewall-cmd --add-interface=DEVICE --zone=ZONE_NAME
# 添加富规则
firewall-cmd --add-rich-rule='rule FIREWALLD.RICHLANGUAGE'
```
##### 常用富规则
* family=    协议簇 ipv4/ipv6
* source address=    源地址
* service name=    服务名
* port port=    protocol=    协议加端口号
* fordward-port port=    protocol=   转发端口
* to-port=    目标端口

#### 重载后生效
```
firewall-cmd --permanent --add-port=PORT/PROTOCOL --zone=ZONE_NAME --zone=ZONE_NAME
firewall-cmd --permanent --add-service=SERVICE_NAME --zone=ZONE_NAME --zone=ZONE_NAME
firewall-cmd --permanent --add-source=IP_ADDR/NETMASK --zone=ZONE_NAME --zone=ZONE_NAME
firewall-cmd --permanent --add-interface=DEVICE --zone=ZONE_NAME --zone=ZONE_NAME
firewall-cmd --permanent --add-rich-rule='rule FIREWALLD.RICHLANGUAGE'
```
