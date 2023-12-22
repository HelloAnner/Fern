

X-Forwarded-For:简称XFF头，它代表客户端，也就是HTTP的请求端真实的IP，只有在通过了HTTP 代理或者负载均衡服务器时才会添加该项。它不是RFC中定义的标准请求头信息

```bash
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

X-Forwarded-For 表示 Nginx 接收到的头，原样的转发过来（假如不转发，Web 服务器就不能获取这个头）。 X-Real-IP，这是一个内部协议头（就是反向代理服务器和 Web 服务器约定的），这个头表示连接反向代理服务器的 IP 地址（这个地址不能伪造），其实个人觉得为了让 PHP 代码保持无二义性，