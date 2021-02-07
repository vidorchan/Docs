nginx 热备部署  双主模式 

**用nginx做负载均衡，作为架构的最前端或中间层，随着日益增长的访问量，需要给负载均衡做高可用架构，利用keepalived解决单点风险，一旦 nginx宕机能快速切换到备份服务器**

安装 keepalived

![1606392057823](C:\Users\QiYuan\AppData\Roaming\Typora\typora-user-images\1606392057823.png)



安装   keepalived 

```xml
yum install nginx keepalived pcre-devel  -y
```

**配置keepalived高可用，修改主配置文件**

　两台均备份

```xml
cp /etc/keepalived/keepalived.conf keepalived.conf.bak
```

![1606392217107](C:\Users\QiYuan\AppData\Roaming\Typora\typora-user-images\1606392217107.png)

```xml
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

```

启动 两台的nginx

```xml
systemctl start nginx
```

启动 keepalived 

```xml
service keepalived start
```

检查下状态 

![1606392438737](C:\Users\QiYuan\AppData\Roaming\Typora\typora-user-images\1606392438737.png)

检查下IP

![1606392527754](C:\Users\QiYuan\AppData\Roaming\Typora\typora-user-images\1606392527754.png)

 检查页面访问 

![1606392544374](C:\Users\QiYuan\AppData\Roaming\Typora\typora-user-images\1606392544374.png)







