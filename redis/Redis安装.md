### Redis安装

##### 一. 下载Redis

[下载地址](https://redis.io/)

##### 二. 安装

```shell
# 解压缩
tar -zxvf redis-6.2.3.tar.gz
# 编译redis(需要提取安装gcc-c++)
make
# 安装
make install
# 修改redis启动脚本
cd utils
cp redis_init_script /etc/init.d/
cd /etc/init.d/
vim redis_init_script
# 修改配置文件路径
CONF="/usr/local/redis/redis.conf"
# 文件开头添加变量
USERNAME="default"
PASSWORD="qaz123"
# 修改停止服务命令
$CLIEXEC --user $USERNAME --pass $PASSWORD -p $REDISPORT shutdown

# 移动配置文件到/etc/local/redis下面
mkdir -p /usr/local/redis
cp redis.conf /usr/local/redis/
cd /usr/local/redis/
vim redis.conf
# 修改redis为后台运行
daemonize yes
# 配置redis的工作空间 需要先创建db这个目录(mkdir db)
dir /usr/local/redis/db
# 修改redis绑定ip bind 127.0.0.1 -::1
bind 0.0.0.0
# 修改redis密码
requirepass qaz123

# 启动redis服务
cd /etc/init.d/
./redis_init_script start
```

##### 三.redis开机自启动

```shell
vim /etc/init.d/redis_init_script
# 添加配置
#chkconfig: 22345 10 90
#description: Start and Stop redis

#完整配置见下面

#注册到开机自启动(文件需要在/etc/init.d/目录下面)
chkconfig redis_init_script on
```

```
#!/bin/sh
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

### BEGIN INIT INFO
# Provides:     redis_6379
# Default-Start:        2 3 4 5
# Default-Stop:         0 1 6
# Short-Description:    Redis data structure server
# Description:          Redis data structure server. See https://redis.io
### END INIT INFO

#chkconfig: 22345 10 90
#description: Start and Stop redis

REDISPORT=6379
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/usr/local/redis/redis.conf"
USERNAME="default"
PASSWORD="qaz123"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC --user $USERNAME --pass $PASSWORD -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```

