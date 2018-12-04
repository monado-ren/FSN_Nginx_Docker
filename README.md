# FSN VeryNginx Docker
VeryNginx on Docker with TLS 1.3 / FGHRSH Service Node Infrastructure

### 特性

- 基于新版 Nginx 1.15 编译，集成 VeryNginx 脚本
- 支持 HTTPS 2 / TLS 1.3 / Brotli / Headers More 等

　
## 使用

### 举个栗子

- Hello World
  - `/opt/ssl/example.com.crt(key)` - 存放证书
  - `/data/wwwroot/example.com/web/` - 网站根目录
  - `/data/wwwlogs/example.com-xxx.log` - 网站日志记录
  - `/data/wwwroot/example.com/conf/nginx.conf` - 网站配置文件

```shell
docker run -d --name nginx --restart always \
 -p 80:80 -p 443:443 -h $(hostname) \
 -v /data/wwwroot:/data/wwwroot \
 -v /data/wwwlogs:/data/wwwlogs \
 -v /opt/ssl:/run/secrets:ro \
 -v /etc/localtime:/etc/localtime:ro \
 fghrsh/fsn_verynginx_docker
 ```

- Advanced Setting
  - `mkdir -p /root/docker_data/nginx/` 创建存放配置的目录，可自行修改
  - `curl -fSL https://raw.githubusercontent.com/fghrsh/FSN_VeryNginx_Docker/master/conf/nginx.conf > /root/docker_data/nginx/nginx.conf`
  - `touch /root/docker_data/nginx/verynginx.json`
  - `chmod 777 /root/docker_data/nginx/verynginx.json`
  - `vim /root/docker_data/nginx/nginx.conf` 编辑 nginx.conf

```shell
docker run -d --restart always \
 -p 80:80 -p 443:443 --name nginx \
 -h $(hostname) --link portainer \
 -v /data/wwwroot:/data/wwwroot \
 -v /data/wwwlogs:/data/wwwlogs \
 -v /opt/ssl:/run/secrets:ro \
 -v /etc/localtime:/etc/localtime:ro \
 -v /root/docker_data/nginx/nginx.conf:/etc/nginx/nginx.conf \
 -v /root/docker_data/nginx/verynginx.json:/opt/verynginx/verynginx/configs/config.json \
 fghrsh/fsn_verynginx_docker
 ```
 
 - 建议修改
   - `/etc/localtime` 用于同步 宿主机 时区设置
   - 编辑 `nginx.conf` 内 `more_set_headers 'Server: $hostname/FS5.online';`
   - `docker run --link portainer` 连接到 portainer 容器（视情况修改，无需要请去除

### nginx.vh.default.conf

```
server {
    listen 80;
    listen 443 ssl http2;
    
    #this line shoud be include in every server block
    include /opt/verynginx/verynginx/nginx_conf/in_server_block.conf;
    
    server_name example.com;
    root /data/wwwroot/example.com/web;
    index index.html index.htm index.php;
    ssl_certificate /run/secrets/example.com.ecc.crt;
    ssl_certificate_key /run/secrets/example.com.ecc.key;
    
    location ~ \.php$ {
        fastcgi_pass   unix:/data/wwwroot/example.com/tmp/php-cgi.sock;
        fastcgi_index  index.php;
        fastcgi_param  DOCUMENT_ROOT   /data/wwwroot/example.com/web;
        fastcgi_param  SCRIPT_FILENAME /data/wwwroot/example.com/web$fastcgi_script_name;
        include fastcgi.conf;
    }
    
    access_log /data/wwwlogs/example.com-access.log main;
    error_log /data/wwwlogs/example.com-error.log crit;
}
```

　
## Thanks
> (๑´ㅁ`) 都看到这了，点个 Star 吧 ~

[docker-nginx / @nginxinc][1]  
[LFS-Docker-Nginx / ©lwl12][2]  
[VeryNginx / ©alexazhou / LGPL-3.0][3]  

  [1]: https://github.com/nginxinc/docker-nginx "Official NGINX Dockerfiles"
  [2]: https://github.com/lwl12/LFS-Docker-Nginx "LWL Gen3 Server Infrastructure - Nginx"
  [3]: https://github.com/alexazhou/VeryNginx/ "VeryNginx is a very powerful and friendly nginx."
