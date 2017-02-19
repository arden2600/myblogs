## 本机上使用nginx反向代理的设置cookie失败问题<br>
在本地机器上使用nginx配置127.0.0.1:port方向代理的时候，遇到这样一个问题：在部署在tomcat上的java EE项目中设置cookies的时候，若是不配置特定的nginx属性，那么该cookies将无法保存在本地。<br>
<br>
主要原因是：在nginx设置方向代理的时候，默认情况下，`nginx不会将代理的域名信息携带到tomcat应用服务器`中，以至于在tomcat容器中通过域名设置cookie的时候，会设置失败。<br>
<br>
解决办法，在nginx配置文件中，需要配置将域名携带到tomcat服务器:<br>
```xml
server {
  listen 80;
  server_name  blog.arden2600.com;
  
  #charset koi8-r;
  #access_log logs/host.access.log main;
  
  proxy_set_header X-Forwarded-Server $host;
  proxy_set_header X-Forwarded-Server $proxy_add_x_forward_for;
  
  #设置方向代理，将域名信息携带到tomcat服务器
  proxy_set_header Host $host;
  location / {
    #root html;
    #index index.html index.htm;
    proxy_pass http://localhost:8887;
    proxy_connect_timeout 600;
    proxy_read_timeout 600;
  }
}
```
<br>
有时候不同的系统环境和nginx版本也许会有不同的问题，需要结合自己实际环境来调试。
