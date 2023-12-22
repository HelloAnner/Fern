
### NGINX的超时配置

keepalive_timeout timeout [ header_timeout ]

默认值 75s

上下文 http server location

第一个参数指定了与client的keep-alive连接超时时间。服务器将会在这个时间后关闭连接。

可选的第二个参数指定了在响应头Keep-Alive: timeout=time中的time值。这个头能够让一些浏览器主动关闭连接，这样服务器就不必要去关闭连接了。没有这个参数，nginx不会发送Keep-Alive响应头（尽管并不是由这个头来决定连接是否“keep-alive”）（服务器在返回数据给用户时，在头header文件中会添加keepalive字段，75s，浏览器在这个时间后能够主动关闭连接）

**proxy_connect_timeout**

默认值 60s

上下文 http server location

说明 该指令设置与upstream server的连接超时时间，有必要记住，这个超时不能超过75秒。

这个不是等待后端返回页面的时间，那是由proxy_read_timeout声明的。如果你的upstream服务器起来了，但是hanging住了（例如，没有足够的线程处理请求，所以把你的请求放到请求池里稍后处理），那么这个声明是没有用的，因为与upstream服务器的连接已经建立了。

这里存在一个超时重试的机制

`proxy_next_upstream off`

**proxy_read_timeout**

语法 proxy_read_timeout time

默认值 60s

上下文 http server location

说明 该指令设置与代理服务器的读超时时间。它决定了nginx会等待多长时间来获得请求的响应。这个时间不是获得整个response的时间，而是两次reading操作的时间。

如果连续的60s内没有收到1个字节, 连接关闭

**proxy_send_timeout**

默认值 60s

上下文 http server location

说明 这个指定设置了发送请求给upstream服务器的超时时间。超时设置不是为了整个发送期间，而是在两次write操作期间。如果超时后，upstream没有收到新的数据，nginx会关闭连接