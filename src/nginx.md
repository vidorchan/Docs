Nginx

高性能的HTTP服务器：网页静态服务器

	5 万并发链接

虚拟主机

反向代理、负载均衡



nginx.conf 6大组件

    全局配置
    events 运行模式
    http
    	upstream 服务分组
    	server 虚拟主机
    		location
    			proxy_pass: url; url加/代表绝对根路径；没有，代表相对路径



worker_process 8; // CPU核心数/ *2 

	-- 查看CPU核心数：

		M1: > top 然后按1

		M2: > grep ^processor /proc/cpuinfo | wc -l

worker_rlimit_profile 65535; // 最好= ulimit -n的值。理论上=ulimit -n的值 / worker_process 



user nginx nginx;

use epoll; // linux采用epoll

work_connections 65535; //单个worker运行客户端最大连接数

multi_accept: on; // 默认 =on表示一个连接过来，只有一个worker唤醒，适用于吞吐量小；=off表示多个worker都唤醒，适用于吞吐量大

sendfile on；开启高效文件传输模式 （sendfile函数）如果图片显示不正常把这个改成off 

tcp_nopush on；在sendfile下才生效，防止网路阻塞，积极的减少网络报文段的数量（将响应头和正文的开始部分一起发送，而不一个接一个的发送。） 

gzip on; 开启压缩

server_tokens     off;    #关闭版本号



root&alias: 设置资源路径

    [root]
    
        语法：root path
        默认值：root html
        配置段：http、server、location、if
    
    [alias]
        语法：alias path
        配置段：location
        
        
     ------------------------------------------
     alias会把后面的路径丢掉，所以，path最后一定要加/
     alias在使用正则匹配时，必须捕捉要匹配的内容并在指定的内容处使用
     alias只能位于location块中。（root可以不放在location中）



正向代理&反向代理：

正向代理代理的是客户端，反向代理代理的是服务器（客户端不用做任何配置）。

举例：VPN访问Google为正向代理；中介找房，反向代理；



负载均衡：默认轮询

    ip_hash; 对应server的状态不能是down,backup
    least_conn; 最少连接数
    weight=XX;
    fair; 需要插件 upstream_fair模块 最小响应时间
    url_hash; 需要插件 hash软件包

服务器状态：

    down 不参加负载均衡
    backup 非backup出现故障或忙的时候才有请求
    max_fails 最大请求失败次数，超过返回proxy_next_upstream中定义的错误
    fail_timeout 经历max_fails后，暂停服务的时间



Server:

    listen 443 ssl;  # 监听443，并且标识为SSL模式
    server_name *.123.com www.123.* 
    	# 通配符只能用在由三段字符组成的首段或者尾端，或者由两端字符组成的尾端。
    	# 也可以使用IP
    location 匹配URL
    	# uri包含正则表达式，则必须有~/~*不区分大小写
    proxy_pass 设置代理服务器地址
    index 设置网站的默认首页



限流熔断：

    令牌桶算法：
    	1.固定速度产生令牌，放到令牌桶中，如果令牌桶满了，则多余的令牌丢弃
    	2.多余的请求会缓存到**队列**	
    漏桶算法：
    	桶满了，多余的请求的会被丢弃！
    	请求匀速流出处理
    计数器：
    
    滑动窗口：



limit_zone zone_name $variable the_size  限制会话数

    limit_zone one $binary_remote_addr 1m;
    	# $remote_addr 的长度为 7 至 15 bytes，会话信息的长度为 32 或 64 bytes。 而 $binary_remote_addr 的长度为 4 bytes，会话信息的长度为 32 bytes
    	# 当size=1M，可以运行记录32000个会话信息 1*1024*1000/32



limit_conn zone_name the_size  限制一个会话的最大连接数，超过返回503

    location /download/ {
    	limit_conn one 1;
    }
    # 对于下载/download/目录下，一个IP只能发起一个连接



    #限制用户连接数来预防DOS攻击
    limit_conn_zone $binary_remote_addr zone=perip:10m;
    limit_conn_zone $server_name zone=perserver:10m;
    #限制同一客户端ip最大并发连接数
    limit_conn perip 2;
    #限制同一server最大并发连接数
    limit_conn perserver 20;
    #限制下载速度，根据自身服务器带宽配置
    limit_rate 300k; 





并发请求测试工具安装

    #ab运行需要依赖apr-util包，安装命令为：
    yum install apr-util
    #安装依赖 yum-utils中的yumdownload 工具，如果没有找到 yumdownload 命令可以
    yum install yum-utils
    cd /opt
    mkdir abtmp
    cd abtmp
    yum install yum-utils.noarch
    yumdownloader httpd-tools*
    rpm2cpio httpd-*.rpm | cpio -idmv
    #操作完成后 将会产生一个 usr 目录 ab文件就在这个usr 目录中
    #简单使用说明
    ./ab -c 100 -n 10000 http://127.0.0.1/index.html
    #-c 100 即：每次并发100个
    #-n 10000 即： 共发送10000个请求



动静分离

    location ~* \.(jpg|gif)$ {             # location匹配将图片交给Image处理
    	proxy_pass http://10.10.0.23:80;      # Image服务器要开启web服务
    }



镜像：

    #启用缓存到本地的功能 静态文件存储到本地
    proxy_store on；
    
    proxy_temp_path 缓存目录;
    #在上面的配置之后，虽然文件被缓存到了本地磁盘上，但每次请求仍会向远端拉取文件，为了避免去远端拉取文件，还必须增加：
    if ( !-e $request_filename) { // 匹配缓存目录中的文件与源文件是否存在
    	proxy_pass http://192.168.10.10; // 源服务器的地址
    }
    
    --------------举例--------------
      location ~*\.(gif|jpg|jepg|png|bmp)$ {
        expires 3d;               //所有链接，浏览器缓存过期时间为3天
        proxy_set_header Accept-Encoding '';
        root /home/mpeg/nginx;          //此目录为服务器的根目录，下面的if语句就是判断此目录下是否有响应的文件
        proxy_store on;
        proxy_store_access user:rw group:rw all:rw;//表示用户读写权限
        proxy_temp_path /home/mpeg/nginx;    //此处为文件的缓存路径，这个路径是和url中的文件路径一致的
        if ( !-e $request_filename) {
            proxy_pass http://192.168.0.1;  //此处为要被代理的服务器的地址
        }
      }



热备部署 : keepalived 

安装：

    yum install nginx keepalived pcre-devel  -y

 配置：

     global_defs {
        vrrp_garp_interval 0
        vrrp_gna_interval 0
     }
     vrrp_instance VI_1 {
         state MASTER    #备用机 修改为 BACKUP
         interface enp0s8
         virtual_router_id 50
         priority 100   # 参数 备用比主机低就可以了 
         advert_int 1
         authentication {
             auth_type PASS
             auth_pass 1111
         }
         virtual_ipaddress {
             192.168.56.120
         }
     }
    ~        

启动：

    systemctl start nginx
    service keepalived start



认证：

    auth_basic
    auth_basic_user_file
    
    yum install httpd-tools -y
    htpasswd -bc /usr/local/nginx/conf/nginxpasswd username password // -c创建
    
    location / {
    	auth_basic "Please Input UserName And Password!";
    	auth_basic_user_file nginxpasswd;
    }



防盗链：防止别人直接从你网站引用图片等链接

    location ~*^.+\.(jpg|gif|png|swf|flv|wma|wmv|asf|mp3|mmf|zip|rar)$ {
        valid_referers none blocked *.benet.com benet.com;
        if($invalid_referer) {
          #return 302 http://www.benet.com/img/nolink.jpg;
          # rewrite ^/ http://www.benet.com/error.png;
          return 404;
          break;
        }
        access_log off;
    }
    
    valid_referers: 配置允许值
    none  ：意思是不存在的Referer头(表示空的，也就是直接访问，比如直接在浏览器打开一个图片)。
    blocked  ：意为根据防火墙伪装Referer头，如：“Referer:XXXXXXX”。浏览器中refer不为空的情况，但是值被代理或防火墙删除了，这些值不以http://或 https://开头。
    server_names  ：为一个或多个服务器的列表，0.5.33版本以后可以在名称中使用“*”通配符。
    
    ！！！需要放在网页缓存的location配置之前才能生效



连接数ulimit -n: 

// ulimit -a 查看当前系统的所有限制值

解决error: too many open files ：  

/etc/security/limits.conf 

    *               soft    nofile           65535
    *               hard   nofile           65535
    *               soft    noproc         65535
    *               hard   noproc         65535



网页压缩：

    gzip  on;                      #开启gzip压缩输出
    gzip_buffers 4 64k;           #表示申请4个单位为64kB的内存作为压缩结果流缓存
    gzip_http_version 1.1;      #用于设置http协议版本，默认是1.1
    gzip_comp_level 2;         #指定gzip压缩比，压缩比最小，处理速度最快
    gzip_min_length 1k;       #设置允许压缩的页面最小字节数
    gzip_vary on;            #让前端的缓存服务器缓存经过gzip压缩的页面
    gzip_types text/plain text/javascript application/x-javascript text/css text/xml application/xml application/xml+rss text/jpg text/png; #压缩类型


