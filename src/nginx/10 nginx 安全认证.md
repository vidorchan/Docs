 ngx_http_auth_basic_module允许通过使用"HTTP基本身份认证"协议验证用户名和密码来限制对资源的访问。坦白点来说，如果想对某目录设置访问权限，可以使用ngx_http_auth_basic_module提供的功能。

  

  **基本身份认证模块 语法及语义**

  

  **auth_basic**

  

   语法：auth_basic string | off;

  

   语义：使用"HTTP基本身份认证"协议启用用户名和密码的验证。指定的参数用作realm，参数值可以包含变量（1.3.10、1.2.7）。设置特殊值off将关闭身份认证。

  

   参数值会作为提示显示在认证对话框标题栏中。

  

  **auth_basic_user_file**

  

   语法：auth_basic_user_file file;

  

   语义：

  

   指定存储用户名和密码的文件格式：

  

```
# comment
name1:password1
name2:password2:comment
name3:password3
1234
```

  

   密码支持以下类型：

  

   **·** 使用crypt()函数加密。可以使用Apache Http Server发行版中的“htpasswd”实用程序或“openssl passwd”命令生成。

  

   **·** 使用基于MD5的密码算法（apr1）的Apache变体进行散列；可以使用相同的工具生成。

  

   **·** 由RFC2307中描述的"{scheme}data"语法（1.0.3+）指定。当前实现方案包括文本（用于示例，不应使用）、SHA(1.3.13)(SHA-1哈希文本，不应使用)、SSHA（SHA-1加盐哈希，被OpenLDAP、Dovecot等软件包使用）。

  

  **htpasswd 生成密码文件**

  

  htpasswd是开源Http服务器Apache Http Server的一个命令工具，所以本机如果没有该命令，需要先安装。

  

```shell
yum install httpd-tools -y
1
```

  

  htpasswd指令用来创建和更新用于基本认证的用户认证密码文件。htpasswd指令必须对密码文件有读写权限，否则会返回错误码。

  

  htpasswd参数列表：

  

| 参数 | 参数说明                                     |
| ---- | -------------------------------------------- |
| -b   | 密码直接写在命令行中，而非使用提示输入的方式 |
| -c   | 创建密码文件，若文件存在，则覆盖文件重新写入 |
| -n   | 不更新密码文件，将用户名密码进行标准输出     |
| -m   | 使用MD5算法对密码进行处理                    |
| -d   | 使用CRYPT算法对密码进行处理                  |
| -s   | 使用SHA算法对密码进行处理                    |
| -p   | 不对密码进行加密处理，使用明文密码           |
| -D   | 从密码文件中删除指定用户记录                 |

 

  htpasswd生成Nginx密码文件：

  

```shell
htpasswd -bc /usr/local/nginx/conf/nginxpasswd Securitit 000000
1
```

  

  此时查看/usr/local/nginx/conf/nginxpasswd文件：

  

```shell
Securitit:$apr1$nuJ/GIEt$nH8z8kk0EFVq5oo9.qRzI/
1
```

  

  若要在已有Nginx密码文件中追加用户，则无需-c参数：

  

```shell
htpasswd -b /usr/local/nginx/conf/nginxpasswd Csdn 111111
1
```

  

  此时查看/usr/local/nginx/conf/nginxpasswd文件：

  

```shell
Securitit:$apr1$nuJ/GIEt$nH8z8kk0EFVq5oo9.qRzI/
Csdn:$apr1$1IWZsiJl$q1K5CwAboegG1LO18Jdta0
12
```

  

  **基本身份认证模块 示例**

  

  基于默认nginx.conf进行修改，使用上面生成的密码文件进行认证：

  

```nginx
worker_processes  1;

error_log  logs/error.log;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  securitit;

        location / {
	    	auth_basic "Please Input UserName And Password!";
		
	    	auth_basic_user_file nginxpasswd;
        }

    }

}
1234567891011121314151617181920212223242526272829
```

  

  通过./nginx -s reload平滑重启Nginx，通过浏览器访问http://192.168.20.9/，会出现如下的效果：

  

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201027131945336.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NlY3VyaXRpdA==,size_16,color_FFFFFF,t_70#pic_center)

  

  注：图标红色标记的即是auth_basic配置的参数值。

  

  此时，需输入用户名和密码访问资源，若点击"取消"，则会提示访问受限：

  

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201027131956672.png#pic_center)

  

  输入正确的用户名和密码，可以正确访问目标资源：

  

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201027132008350.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NlY3VyaXRpdA==,size_16,color_FFFFFF,t_70#pic_center)

  

  **基本身份认证模块 目录检索示例**

  

  参照[Nginx 目录浏览模块 中文乱码 访问认证 ngx_http_autoindex_module](https://blog.csdn.net/securitit/article/details/109154694)，进行配置，Nginx可以进行目录检索，针对不同的目录，设置不同的权限，实现资源访问控制。

  

  访问时，进行身份认证：

  

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020102713202360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NlY3VyaXRpdA==,size_16,color_FFFFFF,t_70#pic_center)

  

  身份认证成功后，可以访问对应的目录及资源：

  

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201027132032290.png#pic_center)

  

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201027132042185.png#pic_center)

  

  此时，对于目录的分层、分类、权限分离就显得很重要了。

  

  **总结**

  

  应用系统中，对于目录的访问权限设置同样重要，但是一般不会使用"HTTP基本身份认证"这种方式。首先，面对大众用户，其表现形式显得很不友好，与现代Web UI的富表现技术相比，过于单薄。再者，使用密码文件的方式管理权限，过于笨重，当待管理的权限体量过大时，会造成很大的不变。