### Redis主从复制

##### 一. 主从复制原理

![主从复制原理](http://static.xiany.top/markdown/20210518110332.png)

### 二.主从复制服务搭建

node1(192.168.126.131 master)

node2(192.168.126.132 slove)

```shell
# node2配置主节点配置
replicaof 192.168.126.131 6379
# 配置主节点密码
masterauth qaz123
```

### 三.无磁盘化复制

![无磁盘化复制](http://static.xiany.top/markdown/20210518142733.png)

开启无磁盘化复制

```
# 开启使用socket复制
repl-diskless-sync no
```

##### 四.Redis过期策略和淘汰机制

[Redis的过期策略和内存淘汰策略最全总结与分析](https://segmentfault.com/a/1190000023060522)

[Redis的数据过期清除策略 与 内存淘汰策略](https://blog.csdn.net/a745233700/article/details/85413179)

[一篇看懂Redis的过期策略和内存淘汰策略](https://juejin.cn/post/6869396128228442119)

[LRU 策略详解和实现](https://leetcode-cn.com/problems/lru-cache/solution/lru-ce-lue-xiang-jie-he-shi-xian-by-labuladong/)

