### Nginx配置

##### 1.upsteam指令参数

1. max_conns 最大连接数
2. slow_start  当不健康的服务器变成[健康]()状态时，或者服务器在一段时间被认为不可用后变得可用时，设置服务器将其权重从零恢复到标称值的时间（`time`）。默认值为零，即禁用慢启动
3. down 标识服务不可用
4. backup 标识为备用机
5. max_fails 最大失败数量
6. fail_timeout 失败等待时间

##### 2.keepalive配置

```nginx
upstream server1{
    server 192.168.100.1:8080;
    server 192.168.100.2:8080;
    # 保持最大连接数
    keepalive 32;
}

server {
    listen 80;
    server_name server1;
    
    location / {
        proxy_pass http://sever1;
        proxy_http_version 1.1;
        proxy_set_header Connection "" 
    }
}
```

