### LVS

##### 一.LVS的工作模式

[LVS 四种工作模式原理、优缺点比较及简单配置](https://blog.csdn.net/Running_free/article/details/77981201)

##### 二.LVS安装

lvs虚拟机 local_ip(192.168.126.129) vip(192.168.126.50)

node1  local_ip(192.168.126.131) vip(192.168.126.50)

node1  local_ip(192.168.126.132) vip(192.168.126.50)

1.停用虚拟网络配置管理器

```
systemctl stop NetworkManager
systemctl disable NetworkManager
```

2.LVS主机虚拟网卡配置

```shell
  # 进入网卡目录
cd /etc/sysconfig/network-scripts/
# 复制一份网卡ens33，也可能是eth0
cp  ifcfg-ens33 ifcfg-ens33:1
# 修改网卡配置 
vim ifcfg-ens33:1
BOOTPROTO=static
DEVICE=ens33:1
ONBOOT=yes
IPADDR=192.158.126.50
NETMASK=255.255.255.0
# 刷新网卡配置
 service network restart
 

ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:c0:04:f6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.126.129/24 brd 192.168.126.255 scope global dynamic ens33
       valid_lft 1783sec preferred_lft 1783sec
    inet 192.158.126.50/24 brd 192.158.126.255 scope global ens33:1
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fec0:4f6/64 scope link
       valid_lft forever preferred_lft forever
```

3.安装ipvsadm

```
yum install ipvsadm
```

3.node1和node2主机虚拟网卡配置

```shell
# 进入虚拟网卡目录
cd /etc/sysconfig/network-scripts/
# 复制一份虚拟网卡
cp ifcfg-lo ifcfg-lo:1
# 配置虚拟网卡
vim ifcfg-lo:1
DEVICE=lo:1
IPADDR=192.168.126.50
NETMASK=255.255.255.255
NETWORK=127.0.0.0
# If you're having problems with gated making 127.0.0.0/8 a martian,
# you can change this to something else (255.255.255.255, for example)
BROADCAST=127.255.255.255
ONBOOT=yes
NAME=loopback
# 刷新网卡
ipup lo

 lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 192.168.126.50/32 brd 192.168.126.50 scope global lo:1
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
```

4.arp-ignore和announce配置(node1,node2)

arp-ignore: ARP响应级别(处理请求)

* 0：只要本机配置了ip，就能响应
* 1： 请求的目标地址到达对应的网络接口，才会响应

arp-announce: ARP通告行为(返回响应)

* 0：本机上任何网络接口都向外通告，所有的网卡都能接受到通告
* 1：尽可能避免本网卡与不匹配的目标进行通告
* 2：只在本网卡通告

```
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.default.arp_ignore = 1
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.lo.arp_announce = 2
```

修改配置

```shell
vim /etc/sysctl.conf
# 把下面内容添加到文件尾部
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.default.arp_ignore = 1
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.lo.arp_announce = 2
# 刷新配置
sysctl -p 
# 添加路由
route add -host 192.168.126.50 dev lo:1
# 报错 -bash: route: command not found
yum install net-tools
# 查看添加结果
route -n

Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.126.2   0.0.0.0         UG    0      0        0 ens33
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 ens33
192.168.126.0   0.0.0.0         255.255.255.0   U     0      0        0 ens33
192.168.126.50  0.0.0.0         255.255.255.255 UH    0      0        0 lo
# 添加开机自动添加路由
echo "route add -host 192.168.126.50 dev lo:1" >> /etc/rc.local
```

5. 配置ipvs

   ```shell
   # 添加集群规则
   ipvsadm -A -t 192.168.126.50:80 -s rr
   # 添加集群
   ipvsadm -a -t 192.168.126.50:80 -r 192.168.126.131:80 -g
   ipvsadm -a -t 192.168.126.50:80 -r 192.168.126.132:80 -g
   ```
   
   最好是本地网络和虚拟网络在一个网段否则，可能无法访问到lvs的虚拟ip

三.LVS和Keepalived双主热备

lvs1(192.168.126.129) lvs主(同上面lvs配置)

lvs2(192.168.126.133) lvs备(lvs1的克隆)

node1(192.168.126.131) nginx1

node2(192.168.126.132) nginx2

lvs1的keepalived配置

```
! Configuration File for keepalived

global_defs {
   router_id keep_129
}

vrrp_script check_nginx_alive {
        # 脚本存放位置
        script "/etc/keepalived/check_nginx_is_runding.sh"
        # 执行间隔
        interval 2
        # 脚本运行成功权重+10
        weight 10
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.126.50
    }
}

# 配置集群的访问的IP+端口,端口与nginx保持一致，都是80
virtual_server 192.168.126.50 80{
    # 健康检查的时间，单位：秒
    delay_loop 6
    # 配置负载均衡的算法，默认是轮询
    lb_algo rr
    # 设置LVS的模式 NAT|TUN|DR
    lb_kind DR
    # 设置会话持久化的时间
    persistence_timeout 5
    # 协议
    protocol TCP

    # 负载均衡的真实服务器，nginx节点具体ip
    real_server 192.168.126.131 80 {
        # 轮询的默认权重配比为1
        weight 1
        # 设置健康检查
        TCP_CHECK {
          # 检查的端口
          connect_port 80
          # 超时时间 2s
          connect_time 2
          # 重试次数 2次
          nb_get_retry 2
          # 间隔时间 3s
          delay_before_retry 3
        }
    }
    real_server 192.168.126.132 80 {
        weight 1
        # 设置健康检查
        TCP_CHECK {
          # 检查的端口
          connect_port 80
          # 超时时间 2s
          connect_time 2
          # 重试次数 2次
          nb_get_retry 2
          # 间隔时间 3s
          delay_before_retry 3
        }
    }

}
```

lvs2的keepalived配置

```
! Configuration File for keepalived

global_defs {
   router_id keep_133
}

vrrp_script check_nginx_alive {
        # 脚本存放位置
        script "/etc/keepalived/check_nginx_is_runding.sh"
        # 执行间隔
        interval 2
        # 脚本运行成功权重+10
        weight 10
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.126.50
    }
}

# 配置集群的访问的IP+端口,端口与nginx保持一致，都是80
virtual_server 192.168.126.50 80{
    # 健康检查的时间，单位：秒
    delay_loop 6
    # 配置负载均衡的算法，默认是轮询
    lb_algo rr
    # 设置LVS的模式 NAT|TUN|DR
    lb_kind DR
    # 设置会话持久化的时间
    persistence_timeout 5
    # 协议
    protocol TCP

    # 负载均衡的真实服务器，nginx节点具体ip
    real_server 192.168.126.131 80 {
        # 轮询的默认权重配比为1
        weight 1
        # 设置健康检查
        TCP_CHECK {
          # 检查的端口
          connect_port 80
          # 超时时间 2s
          connect_time 2
          # 重试次数 2次
          nb_get_retry 2
          # 间隔时间 3s
          delay_before_retry 3
        }
    }
    real_server 192.168.126.132 80 {
        weight 1
        # 设置健康检查
        TCP_CHECK {
          # 检查的端口
          connect_port 80
          # 超时时间 2s
          connect_time 2
          # 重试次数 2次
          nb_get_retry 2
          # 间隔时间 3s
          delay_before_retry 3
        }
    }

}
```

