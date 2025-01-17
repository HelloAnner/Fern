## http/1.1 存在的问题
### TCP连接数限制

对于同一个域名，浏览器最多只能同时创建 6~8 个 TCP 连接 (不同浏览器不一样)。为了解决数量限制，出现了 “域名分片” 技术，其实就是资源分域，将资源放在不同域名下 (比如二级子域名下)，这样就可以针对不同域名创建连接并请求，以一种讨巧的方式突破限制，但是滥用此技术也会造成很多问题，比如每个 TCP 连接本身需要经过 DNS 查询、三步握手、慢启动等，还占用额外的 CPU 和内存，对于服务器来说过多连接也容易造成网络拥挤、交通阻塞等，对于移动端来说问题更明显。

![[f224aa39c472ab764628b232703e9d8c_MD5.jpeg|500]]

在图中可以看到新建了六个 TCP 连接，每次新建连接 DNS 解析需要时间(几 ms 到几百 ms 不等)、TCP 慢启动也需要时间、TLS 握手又要时间，而且后续请求都要等待队列调度

### 线头阻塞 (Head Of Line Blocking) 问题
每个 TCP 连接同时只能处理一个请求 - 响应，浏览器按 FIFO 原则处理请求，如果上一个响应没返回，后续请求 - 响应都会受阻。

## http/2 优势

### 二进制分帧层 (Binary Framing Layer)

帧是数据传输的最小单位，以二进制传输代替原本的明文传输，原本的报文消息被划分为更小的数据帧：
![[6cb79f62a1c54d20cf3243e330799b93_MD5.jpeg]]

二进制格式相比文本格式往往有着更小的体积。举个例子：2147483647这个数字，在文本编码下需要占用10个Byte（[50, 49, 52, 55, 52, 56, 51, 54, 52, 55]），可二进制编码下只需要占用4个Byte（[127, -1, -1, -1]）。

### 多路复用 (MultiPlexing)

在一个 TCP 连接上，我们可以向对方不断发送帧，每帧的 stream identifier 的标明这一帧属于哪个流，然后在对方接收时，根据 stream identifier 拼接每个流的所有帧组成一整块数据。

把 HTTP/1.1 每个请求都当作一个流，那么多个请求变成多个流，请求响应数据分成多个帧，不同流中的帧交错地发送给对方，这就是 HTTP/2 中的多路复用。

流的概念实现了单连接上多请求 - 响应并行，解决了线头阻塞的问题，减少了 TCP 连接数量和 TCP 连接慢启动造成的问题。

所以 http2 对于同一域名只需要创建一个连接，而不是像 http/1.1 那样创建 6~8 个连接，这样对于一些加载元素较多的复杂页面可以大幅提升效率，减少延迟时间（比如客户做的复杂frm模板动辄几百个请求的场景）。

### 服务端推送 (Server Push)

浏览器发送一个请求，服务器主动向浏览器推送与这个请求相关的资源，这样浏览器就不用发起后续请求：
Server-Push 主要是针对资源内联做出的优化，相较于 http/1.1 资源内联的优势：

- 客户端可以缓存推送的资源
- 客户端可以拒收推送过来的资源
- 推送资源可以由不同页面共享
- 服务器可以按照优先级推送资源


### Header 压缩 (HPACK)

http/2 使用 HPACK 算法来压缩首部内容

HTTP2 头部使用的也是键值对形式的值，而且 HTTP1 当中的请求行以及状态行也被分割成键值对，还有所有键都是小写，不同于 HTTP1。除此之外，还有一个包含静态索引表和动态索引表的索引空间，实际传输时会把头部键值表压缩，使用的算法即 HPACK，其原理就是匹配当前连接存在的索引空间，若某个键值已存在，则用相应的索引代替首部条目，比如 “:method: GET” 可以匹配到静态索引中的 index 2，传输时只需要传输一个包含 2 的字节即可；若索引空间中不存在，则用字符编码传输，字符编码可以选择哈夫曼编码，然后分情况判断是否需要存入动态索引表中。

其中静态索引表是 http/2 规范直接确定的固定值组合；而动态索引表是一个 FIFO 队列维护的有空间限制的表，里面含有非静态表的索引，动态索引表是需要连接双方维护的，其内容基于连接上下文，一个 HTTP2 连接有且仅有一份动态表。

### 应用层的重置连接

对于 HTTP/1 来说，是通过设置 tcp segment 里的 reset flag 来通知对端关闭连接的。这种方式会直接断开连接，下次再发请求就必须重新建立连接。

HTTP/2 引入 RST_STREAM 类型的 frame，可以在不断开连接的前提下取消某个 request 的 stream，开销更小。

### 请求优先级设置

HTTP/2 里的每个 stream 都可以设置依赖 (Dependency) 和权重，可以按依赖树分配优先级，解决了关键请求被阻塞的问题。

## http/2 潜在问题

### 视觉上完成度的延迟

虽然 http/2 的加载效率更高，但因为没有 HTTP/1.x 同时连接数量的限制，http/2 可以同时发起多张图片的请求，服务器可以同时响应图片的负载，但由于网络带宽的限制，每张图片的加载速度都会有一定程度的下降，导致视觉效果比较差（页面加载了很久才一起显示图片）。这种情况下 http/1 的连接数限制反而起到了下载队列的作用。

### Server-Push 无效资源推送

Server-Push 满足条件时便会发起推送，可是客户端已经有缓存了就会发送 RST 拒收（例如html中引用的js和css文件），而服务器在收到 RST 之前已经推送资源了，这样就会导致这部分推送无效并且占用带宽资源。

## http/2 应用

### web服务器

http/2最典型的场景就是web服务器，可以突破浏览器连接数限制，并通过服务端推送的方式加速页面的加载，充分利用带宽资源。

### rpc通信

rpc往往用于多进程、微服务间的通信，对效率和实时性有着较高的要求，使用 http/2 可以有效降低传输成本（二进制编码）和提升网络资源使用效率（多路复用）。


- [ ]  复习 Http 1 & 2 (@2023-12-11)