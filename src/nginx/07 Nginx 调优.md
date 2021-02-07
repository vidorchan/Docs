## Nginx 性能调优 方案 

```xml
#设置用户的权限  root nobody 指定 用户名虚拟机内用户   或者 Ip访问 
#user  nobody;
#设置工作进程数 一般为 Cpu 核心*2  4*2 
worker_processes  8;  
# 日志输出参数 
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
# 进程ID 
#pid        logs/nginx.pid;

events {
#指定运行模型 
    use epoll;
# 工作连接数  默认512 根据自己的情况调整 
    worker_connections  1024;
}

#http模块 
http {
#  能够支持的类型 在 这个文件下写着  mime.types
    include       mime.types;
# 默认的类型  在 application/octet-stream;
    default_type  application/octet-stream;
# 日志的格式 
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
#访问日志记录 
    #access_log  logs/access.log  main;
#启动 发送文件 
    sendfile        on;
# 开启TCP 推送 
    #tcp_nopush     on;
# 连接超时时间 
    #keepalive_timeout  0;
    keepalive_timeout  65;
# 开启压缩文件 
    #gzip  on;
# 服务 
# 服务分组  反向代理的核心关键 
 upstream tuling {
# ip 方式 最大失败3个连接  间隔 30S  权重为 5
        server 127.0.0.1:8080       max_fails=3 fail_timeout=30s weight=5;
#根据ip 利用Hash算法决定访问哪台机器 
   ip_hash;
    }
    server {
        listen       80;
        server_name  localhost;
        #charset koi8-r;
#访问日志记录 以及位置  
        #access_log  logs/host.access.log  main;
# 匹配位置 支持正则表达式 
        location / {
# 寻找位置 默认在Nginx 目录下的  类型 
            root   html;
            index  index.html index.htm;
			 proxy_pass   http://127.0.0.1;
        }
#错误信息 页面 
        #error_page  404              /404.html;
#将服务器错误页重定向到静态页/50x.html
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
#实例 入 将访问尾缀为 \.php 跳转到 127.0.0.1
        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}
#将PHP脚本传递给正在侦听127.0.0.1:9000的FastCGI服务器
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}
#拒绝访问.htaccess文件，如果Apache的文档根
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
}
```




