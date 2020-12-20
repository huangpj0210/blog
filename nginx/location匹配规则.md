# location匹配规则

##### 1. 默认匹配规则

```shell
location / {
	root html;
	index index.html index.htm;
}
```

##### 2.精确匹配

```shell
location = / {
	root html;
	index index.html;
}
```

> 只能精确匹配对象的资源文件

##### 3.正则匹配

```
location ~* \.(GIF|png|bmp|jpg|jpeg) {
	root /home/static;
}
```

> *代表不区分大小写

```
location ^~ /static/img {
	root /home;
}
```

> ^~以某个字符路径开头请求