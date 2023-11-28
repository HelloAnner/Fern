源码下载  http://nginx.org/en/download.html

pcre https://sourceforge.net/projects/pcre/postdownload

openssl https://www.openssl.org/source/

zlib http://zlib.net/


```shell
./configure  --with-http_ssl_module --with-pcre=/Users/anner/third/pcre-8.45 --with-zlib=/Users/anner/third/zlib-1.3 --with-openssl=/Users/anner/third/openssl-3.0.12
```

Nginx 的安装目录和源码目录不能相同 --prefix 应该是其他目录


```shell
# Compile NGINX 
make 
# Install NGINX 
sudo make install
```

```bash
echo /usr/local/nginx/sbin | sudo tee -a /etc/paths
```

```
sudo nginx
```