# Nginx 安装

##### 1.下载[nginx](nginx.org)

##### 2.安装依赖环境

1. 安装gcc环境

   yum install gcc-c++

2. 安装PCRE库，用于解析正则表达式

   yum install -y pcre pcre-devel

3. zlib压缩和解压缩依赖

   yum install -y zlib zli-devel

4. SSL安全加密套接字协议https

   yum install -y openssl openssl-devel
   
##### 3.解压缩 

```shell
tar -zxvf nginx-18.1
```

##### 4.创建nginx临时目录，不创建启动报错

```shell
mkdir /var/temp/nginx -p
```

##### 5.生成makefile文件

```shell
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi
```

配置命令解析

| 命令                           | 解释                                 |
| ------------------------------ | ------------------------------------ |
| --prefix                       | 指定nginx安装目录                    |
| --pid-path                     | 指向nginx的pid文件位置               |
| --lock-path                    | 锁定安装文件，防止被恶意篡改或误操作 |
| --error-log-path               | 错误日志路径                         |
| --http-log-path                | http日志文件路径                     |
| --whit-http_gzip-static_module | 启用gzip模块，压缩请求文件           |
| --http-client-body-temp-path   | 设置客户端请求临时目录               |
| --http-proxy-temp-path         | 设置http代理临时目录                 |
| --http-fastcgi-temp-path       | 设置fastcgi临时目录                  |
| --http-uwsgi-temp-path         | 设置uwsgi临时文件目录                |
| --http-scgi-temp-path          | 设置scgi临时文件目录                 |

##### 6.编译nginx

```shell
make
```

##### 7.安装nginx

```shell
make install
```

##### 8.启动nginx

```shell
cd /usr/local/nginx/sbin
./nginx
```

| 命令                  | 作用                                |
| --------------------- | ----------------------------------- |
| ./nginx               | 启动nginx服务                       |
| ./nginx -s reopen     | 重启nginx服务                       |
| ./nginx -s stop       | 强制退出nginx                       |
| ./nginx -s quit       | 优雅退出nginx(等用户请求完成在退出) |
| ./nginx -t            | 检查nginx配置文件是否正确           |
| ./nginx -s reload     | 重新加载nginx配置                   |
| ./nginx -h/./nginx -? | 查看帮助                            |
| ./nginx -v            | 版本信息                            |
| ./nginx -V            | 版本加编译配置信息                  |

##### 9.关闭防火墙

- 查看防火墙状态

  ```shell
  firewall-cmd --state
  ```

  

- 停止防火墙

  ```shell
  systemctl stop firewalld.servcie
  ```

  

- 禁止开机启动

  ```
  systemctl disable firewalld.service
  ```
<!--如果是云服务要到控制台开启端口，默认80端口是打开的-->


