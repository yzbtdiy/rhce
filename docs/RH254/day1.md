# 第1天
## <font color=red>链路聚合和桥接</font>

* broadcast    传输来自所有端口的每个包
* roundrobin    轮训方式传输来自每个端口的包
* activebackup    主备模式，可用于故障转移
* loadbalance    负载均衡，监控流量使用哈希函数为数据包选择端口时达到均衡
* lacp    802.3ad 链路聚合控制协议，可以达到和 loadbalance 相同效果

```
# 新建连接并创建 team 设备 
nmcli con add con-name teamcon type team ifname teamdev config '{"runner":{"name":"activebackup"}}'
# 设置 IP 地址
nmcli con mod teamcon ipv4.method manual ipv4.addresses IP_ADDR/MASK ipv4.gateway GW_IP
# 添加附属端口
nmcli con add con-name team-port1 type team-slave ifname DEVICE_NAME master teamdev
nmcli con add con-name team-port2 type team-slave ifname DEVICE_NAME master teamdev

# 创建桥设备
nmcli con add con-name brcon type bridge ifname brdev
# 添加附属设备
nmcli con add con-name br-port1 type bridge-slave ifname DEVICE_NAME master brdev

# 禁止 team 自动获取 ip
nmcli con mod teamcon ipv4.method disabled ipv6.method ignore
# 桥接 team
nmcli con mod teamcon master brdev
```