# Nginx配置文件解析

##### 1.nginx.conf

* main 全局配置

  ```shell
  #nginx操作用户
  user  nobody;
  #worker进程数量
  worker_processes  2;
  
  # debug info notice warn error crit
  # nginx错误日志存放目录以及级别
  #error_log  logs/error.log;
  #error_log  logs/error.log  notice;
  #error_log  logs/error.log  info;
  
  # pid文件存放目录
  #pid        logs/nginx.pid;
  ```

  

* event 配置工作模式以及连接数

  ```shell
  events {
      # 默认使用epoll(liunx系统)
      use epoll;
      # 每个worker允许连接的最大连接数
      worker_connections  1024;
  }
  ```

  

* http 模块配置

  * server 虚拟主机配置

    * location 路由配置

    * upstream 集群，内网服务器

   ```shell
  http {
    # 导入指令
    include       mime.types;
  default_type  application/octet-stream;
  
    #请求日志格式化 
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
  #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #请求日志存放路径
    #access_log  logs/access.log  main;

    # 文件高效传输开关
    sendfile        on;
  # 和sendfile一起使用，批量发送数据包
    #tcp_nopush     on;

    # tcp保持连接时间
    #keepalive_timeout  0;
    keepalive_timeout  65;

    # 服务器压缩
  #gzip  on;
  
  server {
    	  #监听端口号
        listen       80;
        #监听ip地址
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
  
  	  #路由配置
        location / {
            root   html;
            index  index.html index.htm;
      }
  
        #error_page  404              /404.html;
  
        # redirect server error pages to the static page /50x.html
        # 
      # 错误页面
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
  
        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
      #}
  
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}
  
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
   ```