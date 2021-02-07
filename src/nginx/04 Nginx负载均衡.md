# Nginx负载均衡



# nginx负载均衡介绍

负载均衡的意思是在服务器集群中，需要有一台服务器作为调度者，客户端所有的请求都由调度者接收，调度者再根据每台服务器的负载情况，将请求分配给对应的服务器去处理；

 在这个过程中，调度者如何合理分配任务，保证所有服务器将性能充分发挥，从而保持服务器集群的整体性能最优，这就是负载均衡的问题了。

# nginx负载均衡的方式 

##  轮询      

轮询方式是Nginx负载默认的方式，顾名思义，所有请求都按照时间顺序分配到不同的服务上，如果服务Down掉，可以自动剔除，如下配置后轮训10001服务和10002服务。



```undefined
upstream  dalaoyang-server {
       server    localhost:10001;
       server    localhost:10002;
}
```

##  权重

指定每个服务的权重比例，weight和访问比率成正比，通常用于后端服务机器性能不统一，将性能好的分配权重高来发挥服务器最大性能，如下配置后10002服务的访问比率会是10001服务的二倍。



```undefined
upstream  dalaoyang-server {
       server    localhost:10001 weight=1;
       server    localhost:10002 weight=2;
}
```

##  iphash

每个请求都根据访问ip的hash结果分配，经过这样的处理，每个访客固定访问一个后端服务，如下配置（ip_hash可以和weight配合使用）。



```undefined
upstream  dalaoyang-server {
       ip_hash; 
       server    localhost:10001 weight=1;
       server    localhost:10002 weight=2;
}
```

## 最少连接

将请求分配到连接数最少的服务上。



```undefined
upstream  dalaoyang-server {
       least_conn;
       server    localhost:10001 weight=1;
       server    localhost:10002 weight=2;
}
```

## fair

按后端服务器的响应时间来分配请求，响应时间短的优先分配。 需要插件来帮我们实现  



```undefined
upstream  dalaoyang-server {
       server    localhost:10001 weight=1;
       server    localhost:10002 weight=2;
       fair;  
}
```

# Nginx配置

以轮询为例，如下是nginx.conf完整代码。

```cpp
worker_processes  1;

events {
    worker_connections  1024;
}


http {
   upstream  dalaoyang-server {
       server    localhost:10001;
       server    localhost:10002;
   }

   server {
       listen       10000;
       server_name  localhost;

       location / {
        proxy_pass http://dalaoyang-server;
        proxy_redirect default;
      }
    }
}
```

