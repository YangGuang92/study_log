# nginx

### 1.nginx基本概念

##### 1）nginx是什么

​		nginx是一个高性能的HTTP和反向代理服务器，特点是占用内存少，并发能力强，事实上nginx的并发能力确实在同类型的网站服务器中表现较好

​		nginx专门为性能优化而开发，性能是其最重要的考量，是线上非常重视效率，能经受高负载的考验，有报告表明能支持高达50000个并发连接数

##### 2）反向代理

​		<!--正向代理：在客户端（浏览器）配置代理服务器，通过代理服务器进行互联网访问-->

​		我们只需要将请求发送到反向代理服务器，有反向代理服务器去访问目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外来说就是一个服务器，暴露的是反向代理服务器的地址，隐藏了真实服务器的地址

##### 3）负载均衡

​		单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，就是负载均衡

##### 4）动静分离

​		为了加快网站的解析速度，可以把动态资源和静态资源有不同的服务器来解析，加快解析速度，降低单个服务器的压力



### 2.nginx常用操作命令

- 查看nginx版本： `nginx -v`  
- 启动nginx： `systemctl start nginx`或`/usr/sbin/nginx`（我的nginx启动文件在/us/sbin目录下）
- 关闭nginx：`nginx -s stop`
- 检查nginx配置文件：`nginx -t`
- 重新加载nginx(配置文件)：`nginx -s reload`或`systemctl reload nginx`



### 3.nginx配置文件

<!--nginx配置文件一般在/usr/local/nginx目录中，如果没有，可以通过whereis nginx 查找一下-->

nginx配置文件由三部分内容构成：

##### 1）全局块

​		从配置文件开始到events块之间的内容，主要会设置一些影响nginx服务器整体运行的配置命令，如: 	

```nginx
user nginx;

#支持的并发数
worker_processes auto;
#error_log的位置
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
#加载的其他配置文件
include /usr/share/nginx/modules/*.conf;

```

##### 2）events块

​		events块涉及的指令主要影响nginx服务器与用户的网络连接 ，如:

```nginx
#支持的最大连接数（单个进程并发量）
worker_connections 1024;
```

##### 3）http块

<!--nginx服务器中配置最频繁的部分-->

http块包含两个部分：**http全局块**，**server块**（具体设置描述看下一章）



### 3.nginx配置实例

##### 1）配置虚拟机

<!--这里主要展示server块中的配置-->

```nginx
server {
    	#监听的端口后
        listen       80;
    	#域名解析
        server_name  _;
    	#网站根目录
        root         /usr/share/php;
        # 加载其他配置文件
        include /etc/nginx/default.d/*.conf;

        #除下面提及的需要添加的配置信息外，其他配置保持默认值即可。
        location / {
            #在location大括号内添加以下信息，配置网站被访问时的默认首页
            index index.php index.html index.htm;
        }
        #添加下列信息，配置Nginx通过fastcgi方式处理您的PHP请求
        location ~ .php$ {
        	#Nginx通过本机的9000端口将PHP请求转发给PHP-FPM进行处理
            fastcgi_pass 127.0.0.1:9000;   
            fastcgi_index index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include fastcgi_params;   #Nginx调用fastcgi接口处理PHP请求
        }
		#配置错误页面
        error_page 404 /404.html;
            location = /40x.html {
        }
		#配置错误页面
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

```

##### 2）配置缓存

<!--这里主要展示server块中的配置-->

nginx 通过配置可以告知浏览器，返回数据的有效期，浏览器就可以根据数据的有效期来判断是否需要请求服务器，如果没有超过有效期，就可以不用请求服务器，**这样可以减少服务器请求，并且降低带宽压力**

```nginx
#各种图片的缓存时间（30天）
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
{
	expires      30d;
}
#js和css文件缓存时间(12小时)
location ~ .*\.(js|css)?$
{
	expires      12h;
}
```

##### 3）gzip压缩设置

压缩资源，通过网络传输的资源变小，带宽占用变小，可以快速访问

<!--服务器进行压缩，浏览器需要进行解压，目前大部分浏览器是支持解压的-->

```nginx
		#该部分设置在http块中的的全局块中，也就是server之前
		gzip on;
		#http协议版本
        gzip_http_version 1.1;
		#需要压缩的文件格式
        gzip_types     text/plain application/json application/javascript application/x-javascript text/javascript text/css application/xml application/xml+rss;
        gzip_vary on;
        gzip_proxied   expired no-cache no-store private auth;
		#如果是IE1-6，就关闭压缩，因为IE1-6不支持压缩
        gzip_disable   "MSIE [1-6]\.";
```

##### 4）负载均衡设置

分发段配置：

```nginx
#该部分设置在http块中的的全局块中
# upstream后的test要和 server块中location中的代理的相同
upstream test1{
    #保证同一个ip访问同一台服务器，这样就可以避免之前访问a服务器，之后访问b服务器session丢失的问题
    ip_hash;
    # 分发到的服务器    分发权重优先级(1:1)  最大失败次数   失败超时时间
	server 192.168.174.130 weight=1 max_fails=3 fail_timeout=20s; #注意，真实服务器需要开启nginx并监听80端口
    server 192.168.174.130 weight=1 max_fails=3 fail_timeout=20s;
}

server {
    	#监听的端口后
        listen       80;
    	#域名解析
        server_name  test1.123.com;
        #除下面提及的需要添加的配置信息外，其他配置保持默认值即可。
        location / {
           #分发代理 (http://后面需要和前面upstream后面的相同)
           proxy_pass: http://test1;
        }
    }
```





  