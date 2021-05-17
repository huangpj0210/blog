## Keepalived配置

##### 一.安装Keepalived

1. [下载](https://www.keepalived.org/download.html)Keepalived，上传到服务器

2. 解压Keepalived软件包

   ```shell
   tar -zxvf keepalived-2.2.2.tar.gz
   ```

   

3. 配置Keepalived

   ```shell
   ./configure --prefix=/usr/local/keepalived --sysconf=/etc
   ```

   `--prefix`指定安装目录，`--sysconf`核心配置文件存放目录

   ```shell
   # 出现WARNING - this build will not support IPVS with IPv6. Please install libnl/libnl-3 dev libraries to support IPv6 with IPVS.提示需要安装libnl/libnl-3,然后重新configure
   yum -y install libnl libnl-devel
   ./configure --prefix=/usr/local/keepalived --sysconf=/etc
   ```

4. 安装Keepalived

   ```
   make && make install
   ```

5. 查看keepalived安装位置

   ```shell
   whereis keepalived
   ```

6. 把Keepalived注册为系统服务

   ```shell
   # 进入keepalived解压目录
   cd /home/software/keepalived-2.2.2/keepalived/etc
   cp init.d/keepalived /etc/init.d/ 
   cp sysconfig/keepalived /etc/sysconfig/
   systemctl daemon-reload
   # 启动keepalived服务
   systemctl start keepalived
   # 查看keepalived服务状态
   systemctl status keepalived
   # 停止keepalived服务
   systemctl stop keepalived
   ```

   

##### 二.Keepalived配置文件

```
# 全局配置
global_defs {
   # 通知邮件列表	
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   # 发邮件邮箱
   notification_email_from Alexandre.Cassen@firewall.loc
   # 邮件服务器ip
   smtp_server 192.168.200.1
   # 邮件服务器连接超时事件
   smtp_connect_timeout 30
   # 路由id: keepalived主机唯一标识符,全局唯一
   router_id keep_129
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
# 计算机节点配置
vrrp_instance VI_1 {
	# 表示当前节点是MASTER还是BACKUP
    state MASTER
    # 当前实例绑定的网卡
    interface ens33
    # 虚拟路由id,保证主备节点一致
    virtual_router_id 51
    # 优先级/权重
    priority 100
    # 主备之间同步检查的时间间隔，默认1s
    advert_int 1
    # 认证
    authentication {
    	# 认证方式
        auth_type PASS
        # 认证密码
        auth_pass 1111
    }
    # 虚拟ip
    virtual_ipaddress {
        192.168.126.100
    }
}
```

启动keepalived

```shell
 /usr/local/keepalived/sbin/keepalived
 ip addr
 
 ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:c0:04:f6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.126.129/24 brd 192.168.126.255 scope global noprefixroute dynamic ens33
       valid_lft 1267sec preferred_lft 1267sec
    inet 192.168.126.100/32 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::a219:8d67:4947:8a44/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
       
```

##### 三.Keepalived监控Nginx自动重启

1.编写监控nginx shell脚本

```shell
#!/bin/bash

count=`ps -C nginx --no-header | wc -l`
# 判断nginx进程id是否存在如果不存在重启nginx服务器
if [ $count -eq 0 ];then
   /usr/local/nginx/sbin/nginx
   # 等待一段时间判断nginx是否启动成功,没有则停止keepalived,切换到备用机
   sleep 3
   if [ `ps -C nginx --no-header | wc -l ` -eq 0 ];then
        killall keepalived
   fi
fi
```

编写完成需要使用`chmod +x check_nginx_is_runding.sh`添加脚本执行权限

2.在Keepalived配置文件添加检查脚本

```
vrrp_script check_nginx_alive {
	# 脚本存放位置
	script "/etc/keepalived/check_nginx_is_runding.sh"
	# 执行间隔
	interval 2
	# 脚本运行成功权重+10
	# 负数就-10
	weight 10
}

vrrp_instance VI_1 {
	......
	track_script {
		check_nginx_alive #检查nginx脚本
	}
}
```

##### 四.keepalived主备配置

准备2台服务: 例如node1(192.168.126.129) node2(192.168.126.131)  keepalived vip(192.168.126.100)

node1配置文件:

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
    track_script {
        check_nginx_alive
    }
    virtual_ipaddress {
        192.168.126.100
    }
}
```

node2配置文件:

```
! Configuration File for keepalived

global_defs {
   router_id keep_131
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
        192.168.126.100
    }
}
```



##### 五.Keepalived双主热备

node1服务器配置:

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
    track_script {
        check_nginx_alive
    }
    virtual_ipaddress {
        192.168.126.100
    }
}

vrrp_instance VI_2 {
    state BACKUP
    interface ens33
    virtual_router_id 52
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.126.101
    }
}
```

node2服务器配置:

```
! Configuration File for keepalived

global_defs {
   router_id keep_131
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
        192.168.126.100
    }
}

vrrp_instance VI_2 {
    state MASTER
    interface ens33
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.126.101
    }
}
```

修改配置完成重启2台服务器，如果是公网ip需要做dns负载均衡来实现双主热备