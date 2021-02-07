Nginx的proxy_store作用是直接把静态文件在本地硬盘创建并读取，类似于七牛或者又拍这样的镜像CDN功能，首次访问会自动获取源站的静态图片等文件，之后的访问就是直接从CDN服务器读取，加快了速度。

需要配置一下参数：

  ```xml
#启用缓存到本地的功能
  proxy_store on;
  #表示用户读写权限，如果在error中报路径不允许访问的话就用"chomod -R a+rw"将下面配置的路径改为相应的权限.
  proxy_store_access user:rw group:rw all:rw;
  #此处为文件的缓存路径，这个路径是和url中的文件路径一致的
  proxy_temp_path 缓存目录;
  #在上面的配置之后，虽然文件被缓存到了本地磁盘上，但每次请求仍会向远端拉取文件，为了避免去远端拉取文件，还必须增加：
  if ( !-e $request_filename) {
  proxy_pass http://192.168.10.10;
  }
  注："!-e $request_filename"正则表达式，匹配缓存目录中的文件与源文件是否存在。
 "http://192.168.10.10" 源服务器的地址，默认端口80，如监听其他端口，此处要指出，例如4000端口，http://192.168.10.10:4000
  ```



#

整体配置如下（修改nginx的配置文件nginx.conf）：

```xml
  location / {                 //这里的location是要换成自己经过精确匹配的location，比如要缓存图片要写成 "location ~*\.(gif|jpg|jepg|png|bmp)${"
    expires 3d;               //所有链接，浏览器缓存过期时间为3天
    proxy_set_header Accept-Encoding '';
    root /home/mpeg/nginx;          //此目录为服务器的根目录，下面的if语句就是判断此目录下是否有响应的文件
    proxy_store on;             //表示开启缓存
    proxy_store_access user:rw group:rw all:rw;//表示用户读写权限
    proxy_temp_path /home/mpeg/nginx;    //此处为文件的缓存路径，这个路径是和url中的文件路径一致的
    if ( !-e $request_filename) {
        proxy_pass http://192.168.0.1;  //此处为要被代理的服务器的地址
    }
  }
```

