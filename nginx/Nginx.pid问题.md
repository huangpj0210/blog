# Nginx.pid打开失败

##### 1.nginx.pid文件不存在

```shell
nginx: [error] open() "/var/run/nginx/nginx.pid" failed (2: No such file or directory)

#解决办法
mkdir /var/run/nginx
```

##### 2.invalid PID number ""

```shell
nginx: [error] incalid PID number "" in "/var/run/nginx/nginx.pid"

#解决办法
./nginx -c /usr/local/nginx/conf/nginx.conf
```

