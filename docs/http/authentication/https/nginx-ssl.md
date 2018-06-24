
# Nginx 配置 HTTPS

## 证书文件获取

参见 [证书申请](authentication/https/ca.md)


## 判断 Nginx 有没有启动 SSL 模块

``` bash
nginx -V
```
查看 `configure arguments` 中 如果有 `with-http_ssl_module` 则表明开启了 SSL 模块。



## 安装时 启用 SSL 模块

``` bash
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_gzip_static_module --with-http_ssl_module --with-openssl=/usr/local/openssl-1.1.0g 

make && make install
```





## nginx.conf

`nginx/conf` 文件夹下新建文件夹 `cert` , 放入 `ca.kail.xyz.crt` 证书文件 和 `ca.kail.xyz.key` 私钥


```
server {
    listen       443 ssl;
    server_name  ca.kail.xyz;
    ssl_certificate      cert/ca.kail.xyz.crt;
    ssl_certificate_key  cert/ca.kail.xyz.key;
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;
    location / {
        root   html;
        index  index.html index.htm;
        autoindex on; 
    }
}
```




## 强制 HTTPS

``` 
server {
    listen 80;
    server_name ca.kail.xyz;
    rewrite ^(.*) https://$server_name$1 permanent;
}
```




# Read More

- [CentOS 7.4 实例配置 Nginx + HTTPS 服务](https://help.aliyun.com/knowledge_detail/42863.html)
- [nginx使用ssl模块配置HTTPS支持](https://www.centos.bz/2011/12/nginx-ssl-https-support/)
- [nginx配置ssl加密（单/双向认证、部分https）](http://seanlook.com/2015/05/28/nginx-ssl/)

