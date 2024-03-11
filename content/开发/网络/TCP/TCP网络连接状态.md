

![[attachments/697f80ad86c8d28f612d67e8c5cef24e_MD5.jpeg]]

## 一、LISTEN

LISTEN：表示侦听来自远方的TCP端口的连接请求。首先服务端需要打开一个socket 进行监听，状态为LISTEN。 有提供某种服务才会处于LISTEN状态， TCP状态变化就是某个端口的状态变化，提供一个服务就打开一个端口。例如FTP服务启动后首先处于侦听(LISTEN)状态。处于侦听LISTEN状态时，该端口是开放的，等待连接，但还没有被连接。

## 二、SYN-SENT

在客户端发送连接请求后，等待匹配的连接请求。比如客户端tcp发送一个SYN以请求建立一个连接，之后状态置为SYN_SENT。

## 三、SYN-RECEIVED

在收到和发送一个连接请求后等待对方对连接请求的确认。当服务器收到客户端发送的同步信号时，将标志位ACK和SYN置1发送给客户端，此时服务器端处于SYN_RCVD状态， 如果连接成功了就变为ESTABLISHED，正常情况下SYN_RCVD状态非常短暂。

## 四、ESTABLISHED

ESTABLISHED状态是表示两台机器正在传输数据，观察这个状态最主要的就是看哪个程序正在处于ESTABLISHED状态。

![[attachments/c1fa5ab677d34264b0d65767316cbc3b_MD5.jpeg]]

## 五、FIN-WAIT-1

等待远程TCP连接中断请求，或先前的连接中断请求的确认。主动关闭(active close)端应用程序调用close，于是其TCP发出FIN请求主动关闭连接，之后进入FIN-WAIT-1状态。


## 六、FIN-WAIT-2

主动关闭端接到ACK后，就进入了FIN-WAIT-2，这时处于等待远程TCP等待连接中断请求，这就是著名的<mark style="background: #FF5582A6;">半关闭</mark>的状态了，这是在关闭连接时，客户端和服务器两次握手之后的状态。在这个状态下，应用程序还有接受数据的能力，但是已经无法发送数据，但是也有一种可能是，客户端一直处于FIN_WAIT_2状态，而服务器则一直处于WAIT_CLOSE状态，而直到应用层来决定关闭这个状态。


## 七、CLOSE-WAIT

等待从本地用户发来的连接中断请求，被动关闭(passive close)端TCP接到FIN后，就发出ACK以回应FIN请求(它的接收也作为文件结束符传递给上层应用程序)，并进入CLOSE_WAIT状态。

**对方主动关闭连接或者网络异常导致连接中断，这时我方的状态会变成CLOSE_WAIT** 此时我方要调用close()来使得连接正确关闭。

### 为什么会存在大量的 closed-wait 

**CLOSE_WAIT是被动关闭端在等待应用进程的关闭**

就需要问一下server ， 为什么你会一直在等待关闭呢？

- **程序问题**：如果代码层面忘记了 close 相应的 socket 连接，那么自然不会发出 FIN 包，从而导致 CLOSE_WAIT 累积；或者代码不严谨，出现死循环之类的问题，导致即便后面写了 close 也永远执行不到。
- 响应太慢或者超时设置过小：如果连接双方不和谐，一方不耐烦直接 timeout，另一方却还在忙于耗时逻辑，就会导致 close 被延后。响应太慢是首要问题，不过换个角度看，也可能是 timeout 设置过小。
- BACKLOG 太大：此处的 backlog 不是 syn backlog，而是 accept 的 backlog，如果 backlog 太大的话，设想突然遭遇大访问量的话，即便响应速度不慢，也可能出现来不及消费的情况，导致多余的请求还在[队列](http://jaseywang.me/2014/07/20/tcp-queue-%E7%9A%84%E4%B8%80%E4%BA%9B%E9%97%AE%E9%A2%98/)里就被对方关闭了。

如果是我们自己写的一些程序，比如用 HttpClient 自定义的蜘蛛，那么八九不离十是程序问题，如果是一些使用广泛的程序，比如 Tomcat 之类的，那么更可能是响应速度太慢或者 timeout 设置太小或者 BACKLOG 设置过大导致的故障。





## 八、CLOSING

等待远程TCP对连接中断的确认。时间短，一般很少见。

## 九、LAST-ACK

等待原来的发向远程TCP的连接中断请求的确认，被动关闭端一段时间后，接收到文件结束符的应用程序将调用CLOSE关闭连接。这导致它的TCP也发送一个 FIN，等待对方的ACK，就进入了LAST-ACK状态。

## 十、TIME-WAIT

等待足够的时间以确保远程TCP接收到连接中断请求的确认，在主动关闭端接收到FIN后，TCP就发送ACK包，并进入TIME-WAIT状态。我方主动调用close()断开连接，收到对方确认后状态变为TIME_WAIT。TCP协议规定TIME_WAIT状态会一直持续2MSL(即两倍的分段最大生存期)，以此来<mark style="background: #FF5582A6;">为了防止发送出去的 ACK 服务端没有收到</mark>。处于TIME_WAIT状态的连接占用的资源不会被内核释放，所以作为服务器，在可能的情况下，尽量不要主动断开连接，以减少TIME_WAIT状态造成的资源浪费。

> **Maximum Segment Lifetime** 报文最大生存时间，它是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃


## 十一、CLOSED

没有任何连接状态。


注意，通常情况下这11种状态不是客户端和服务器都有的，其中

只存在于客户端的状态有：（1）SYN_SENT （2）FIN_WAIT_1 （3）FIN_WAIT_2 （4）CLOSING （5）TIME_WAIT 。

只存在于服务器端的状态有：（1）LISTEN （2）SYN_RCVD （3）CLOSE_WAIT （4）LAST_ACK 。

客户端和服务器端共有的状态有：（1）CLOSED （2）ESTABLISHED 。


## 参考

[线上大量CLOSE\_WAIT的原因深入分析 - 掘金](https://juejin.cn/post/6844903734300901390)
[TCP连接的11种状态 - Jcpeng\_std - 博客园](https://www.cnblogs.com/JCpeng/p/15016816.html)