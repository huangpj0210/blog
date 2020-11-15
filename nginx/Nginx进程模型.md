# Nginx进程模型

##### 1.查看nginx进程

```
ps -ef|grep nginx
root      45271      1  0 11:40 ?        00:00:00 nginx: master process ./nginx
nobody    45272  45271  0 11:40 ?        00:00:00 nginx: worker process
root      48622  41426  0 12:46 pts/0    00:00:00 grep --color=auto nginx
```

nginx默认只有一个master进程和一个worker进程，worker进程数量可以修改nginx.conf文件进行配置

```shell
vi nginx.conf
worker_processes  2;

../sbin/nginx -s reload
ps -ef | grep nginx

root      45271      1  0 11:40 ?        00:00:00 nginx: master process ./nginx
nobody    49426  45271  0 13:02 ?        00:00:00 nginx: worker process
nobody    49427  45271  0 13:02 ?        00:00:00 nginx: worker process
root      49441  41426  0 13:02 pts/0    00:00:00 grep --color=auto nginx
```

##### 2.进程解析

* master主进程

  负责worker进程的管理，还可以接收一些信号:

  > nginx -s stop
  >
  > nginx -s quit
  >
  > nginx -s reload,
  >
  > nginx -t

  ![nginx进程模型图](http://static.xiany.top/markdown/20201115203523.png)

##### 3.worker抢占机制

![worker锁抢占机制](http://static.xiany.top/markdown/20201115185321.png)

##### 4.nginx事件处理机制

![nginx事件处理机制](http://static.xiany.top/markdown/20201115203655.png)

每个worker进程的最大连接数修改nginx.conf文件进行配置

```shell
events {
    # 默认使用epoll(liunx系统)
    use epoll;
    # 每个worker允许连接的最大连接数
    worker_connections  10240;
}
```

