### Redis哨兵模式

##### 一.哨兵模式

![哨兵模式](http://static.xiany.top/markdown/20210518162633.png)

##### 二.哨兵模式配置

```shell
# 从安装包复制哨兵模式配置文件到/usr/local/redis
cp sentinel.conf /usr/local/redis/
# 关闭保护模式
protected-mode no
# 设置后台运行
daemonize yes
# 配置日志文件存放目录
logfile /usr/local/redis/sentinel/redis-sentinel.log
# 工作空间
dir /usr/local/redis/sentinel
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 192.168.126.131 6379 2
# 配置master密码
sentinel auth-pass mymaster qaz123
# 哨兵检查时间
sentinel down-after-milliseconds mymaster 10000
# 数据并行同步的数量
sentinel parallel-syncs mymaster 1
# 复制次文件到其他服务器
scp sentinel.conf root@192.168.126.132:/usr/local/redis
# 启动哨兵
redis-sentinel sentinel.conf
```

##### 三.图解哨兵机制

![哨兵监控](http://static.xiany.top/markdown/20210518171442.png)

![故障转移](http://static.xiany.top/markdown/20210518171343.png)

![选举leader](http://static.xiany.top/markdown/20210518171852.png)

![原master恢复](http://static.xiany.top/markdown/20210518172029.png)

哨兵模式的部署规定

* 哨兵节点至少要3个或者奇数个节点
* 哨兵分布式部署在不同的计算机节点
* 一组哨兵只监听一组主从

[为什么redis推荐奇数个节点](https://blog.csdn.net/qq32933432/article/details/105785571)

##### 五.Springboot集成Redis哨兵模式

```yaml
spring:
 redis:
  database: 1
  password: qaz123
  sentinel:
  	master: mymaster
  	nodes: 192.168.126.131:26379,192.168.126.132:26379
```

