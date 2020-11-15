# Nginx日志文件切割

##### 1.创建切割日志`cut_nginx_log.sh`脚本

```shell
#!/bin/bash
LOG_PATH="/var/log/nginx"
RECORD_TIME=$(date -d "yesterday" +%Y-%m-%d+%H:%M)
PID=/var/run/nginx/nginx.pid
mv ${LOG_PATH}/access.log ${LOG_PATH}/access.${RECORD_TIME}.log
mv ${LOG_PATH}/error.log ${LOG_PATH}/error.${RECORD_TIME}.log
#向nginx主进程发送信号，用于重新打开日志文件
kill -USR1 `cat $PID`
```

##### 2.为脚本添加可执行权限

```shell
chmod +x cut_nginx_log.sh
```

##### 3.安装定时任务

```shell
yum install crontabs
```

##### 4.添加定时任务

```shell
crontab -e
*/1 * * * * /usr/local/nginx/sbin/cut_nginx_log.sh
```

##### 5.crontab常用命令

| 命令                  | 作用             |
| --------------------- | ---------------- |
| service crond start   | 启动定时任务     |
| service crond stop    | 停止定时任务     |
| service crond restart | 重启定时任务     |
| service crond reload  | 重新加载配置     |
| crontab -e            | 编辑定时任务     |
| crontab -l            | 查看定时任务列表 |

