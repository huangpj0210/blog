# Nginx跨域和防盗链设置

在浏览器中同一域名下的资源可以访问，访问不同域名的资源会被浏览器拦截CORS,其中解决方式有Jsonp,spring cors，nginx等方式都可以解决跨域问题

##### 1.SpringBoot解决跨域

```java
@Bean
public CorsFilter corsFiltler(){
    //1.添加cors配置信息
    CorsConfiguration config = new CorsConfiguration();
    config.addAllowedOrigin("http://localhost:8080");
    config.addAllowedOrigin("*");
    
    //设置是否运行cookie信息
    config.setAllowCredentials(true);
    //设置允许请求的方式
    config.setAllowedMethod("*");
    //设置允许header
    config.setAllowedHeader("*");
	
    //2.为url添加映射路径
    UrlBasedCorsConfigurationSource corsSource = new UrlBasedConfigurationSource();
    corsSource.registerCorsConfiguration("/**",config);
    return new CorsFilter(corsSource);
}
```



##### 2.Nginx解决跨域

```shell
server {
	listen 80;
	server_name localhost;
	
	# 允许跨域请求的域，*代表所有
	add_header 'Access-Control-Allow-Origin' *;
	# 允许带上cookie请求
	add_header 'Access-Control-Allow-Credentials' 'true';
	# 允许请求的方法，比如: GET/POST/PUT/DELETE
	add_header 'Access-Control-Allow-Methods' *;
	#允许请求的Header
	add_header 'Access-Control-Allow-Headers' *;
	
	location / {
		root /home/static;
		index index.html;
	}
}
```

##### 3.图片防盗链设置

```shell
#对源站点验证
valid_referers *.test.com;
#非法引入会进入下方判断
if($invalid_referer) {
	return 404;
}
```

