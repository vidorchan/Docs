# Nginx限流熔断 

*作为优秀的负载均衡模块，目前是我们工作中用到最多的。其实，该模块是提供了我们需要的后端限流功能的。通过官方文档介绍，

### 令牌桶算法

![令牌桶算法](https://images2018.cnblogs.com/blog/802666/201805/802666-20180502140242681-53841293.jpg)

算法思想是：

- 令牌以固定速率产生，并缓存到令牌桶中；
- 令牌桶放满时，多余的令牌被丢弃；
- 请求要消耗等比例的令牌才能被处理；
- 令牌不够时，请求被缓存。

### 漏桶算法

![漏桶算法](https://images2018.cnblogs.com/blog/802666/201805/802666-20180502140248838-284301317.png)

算法思想是：

- 水（请求）从上方倒入水桶，从水桶下方流出（被处理）；
- 来不及流出的水存在水桶中（缓冲），以固定速率流出；
- 水桶满后水溢出（丢弃）。
- 这个算法的核心是：缓存请求、匀速处理、多余的请求直接丢弃。
  相比漏桶算法，令牌桶算法不同之处在于它不但有一只“桶”，还有个队列，这个桶是用来存放令牌的，队列才是用来存放请求的。

从作用上来说，漏桶和令牌桶算法最明显的区别就是是否允许突发流量(burst)的处理，漏桶算法能够强行限制数据的实时传输（处理）速率，对突发流量不做额外处理；而令牌桶算法能够在限制数据的平均传输速率的同时允许某种程度的突发传输。

Nginx按请求速率限速模块使用的是漏桶算法，即能够强行保证请求的实时处理速度不会超过设置的阈值。

### 案例

通过查看nginx官方文档，https://www.nginx.cn/doc/

# HttpLimit zone

本模块可以针对条件，进行会话的并发连接数控制。（例如：限制每个IP的并发连接数。）

__配置示例__

```
http {
: limit_zone   one  $binary_remote_addr  10m;

: ...

: server {

: ...

: location /download/ {
: limit_conn   one  1;
: }
```

## 指令

- [#limit_zone limit_zone]
- [#limit_conn limit_conn]

## limit_zone

**语法：** *limit_zone zone_name $variable the_size*

**默认值：** *no*

**作用域：** *http*

本指令定义了一个数据区，里面记录会话状态信息。
$variable 定义判断会话的变量；the_size 定义记录区的总容量。

例子：

```
limit_zone   one  $binary_remote_addr  10m;
```

定义一个叫“one”的记录区，总容量为 10M，以变量 $binary_remote_addr 作为会话的判断基准（即一个地址一个会话）。


您可以注意到了，在这里使用的是 $binary_remote_addr 而不是 $remote_addr。

$remote_addr 的长度为 7 至 15 bytes，会话信息的长度为 32 或 64 bytes。 而 $binary_remote_addr 的长度为 4 bytes，会话信息的长度为 32 bytes。

当区的大小为 1M 的时候，大约可以记录 32000 个会话信息（一个会话占用 32 bytes）。

## limit_conn

**语法：** *limit_conn zone_name the_size*

**默认值：** *no*

**作用域：** *http, server, location*

指定一个会话最大的并发连接数。 当超过指定的最发并发连接数时，服务器将返回 "Service unavailable" (503)。

例子：

```
limit_zone   one  $binary_remote_addr  10m;

: server {
: location /download/ {
: limit_conn   one  1;
: }
```

定义一个叫“one”的记录区，总容量为 10M，以变量 $binary_remote_addr 作为会话的判断基准（即一个地址一个会话）。 限制 /download/ 目录下，一个会话只能进行一个连接。 简单点，就是限制 /download/ 目录下，一个IP只能发起一个连接，多过一个，一律503。

### ab 测试安装

```html
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
```