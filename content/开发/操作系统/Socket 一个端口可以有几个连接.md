### 服务器端

socket 服务端需要开一个端口监听（listen）socket 请求就行，然后一个端口，理论上来说，最大能支持2的32次方（ip 数）×2 的 16 次方（port 数）个连接，但是 linux 对 [打开文件数有限制](https://garden.3dot141.top/1-Inputs/Article/%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6)（65536个，每个 socket 连接占用一个文件）


服务端会产生两个 socket(listen_socket_fd 和 connect_socket_fd)

- 监听 socket 是服务器作为客户端连接请求的一个对端，只需创建一次即可，它存在于服务器的整个生命周期，可为成千上万的客户端服务
- 一旦一个客户端和服务器连接成功，完成了 TCP 三次握手，操作系统内核就为这个客户端生成一个已连接套接字（connect_socket_fd），让应用服务器使用这个 connect_socket_fd 和客户端进行通信。
- 如果应用服务器完成了对这个客户端的服务，那么关闭的就是已连接套接字，这样就完成了 TCP 连接的释放。请注意，这个时候释放的只是这一个客户端连接，其它被服务的客户端连接可能还存在。最重要的是，监听套接字一直都处于“监听”状态，等待新的客户请求到达并服务。
- 使用两个 socket，按职责分工，listen_socket_fd 专门负责响应客户端的请求，每个新的 connect_socket_fd 专门负责该次连接的数据交互，分层协作，提高服务端的性能。

```java
public static void serverAccept() throws Exception {  
    ServerSocket serverSocket = new ServerSocket();  
    serverSocket.bind(new InetSocketAddress(8080));  
    int i = 0;  
    while (true) {  
        Socket accept = serverSocket.accept();  
        System.out.printf("accept %d \n", i++);  
    }  
}  
 
public static void serverClose() throws Exception {  
    ServerSocket serverSocket = new ServerSocket();  
    serverSocket.bind(new InetSocketAddress(8080));  
    Socket accept = serverSocket.accept();  
    new Thread(() -> {  
        try {  
            OutputStream outputStream = accept.getOutputStream();  
            OutputStreamWriter outputStreamWriter = new OutputStreamWriter(outputStream);  
            while (true) {  
                outputStreamWriter.write(UUID.randomUUID().toString() + System.lineSeparator());  
                outputStreamWriter.flush();  
                Thread.sleep(1000);  
                System.out.println("写了一个");  
            }  
        } catch (IOException | InterruptedException e) {  
            e.printStackTrace();  
        }  
    }).start();  
    serverSocket.close();  
    System.out.println("连接关闭");  
    Thread.sleep(100000);  
}  
 
public static void client() throws Exception {  
    Socket socket = new Socket();  
    socket.connect(new InetSocketAddress("localhost", 8080));  
    InputStream in = socket.getInputStream();  
    BufferedReader reader = new BufferedReader(new InputStreamReader(in));  
    String msg;  
    while ((msg = reader.readLine()) != null) {  
        System.out.println("读了一个");  
        System.out.println(msg);  
    }  
    System.out.println("1111");  
    Thread.sleep(100000);  
}
 
```


结论 1：通过组合 `serverClose` 和 `client` 可以验证，`ServerSocket` 只是起到监听的作用。  
结论 2：通过组合 `serverAccpet` 和 `client` 可以验证，同一个端口，可以同时建立 n 个连接

### 客户端

对于客户端，网络通信过程中服务端监听一个固定的端口,客户端主动发起连接请求后要经过三次握手才能与服务器建立起一个 TCP 连接.客户端每次发起一个 TCP 连接时,系统会随机选取一个空闲的端口,该端口是独占的不能与其他 TCP 连接共享,因此理论上一台机器有多少空闲的端口,就能对外发起多少个 TCP 连接。根据 TCP/IP 协议,端口 port 使用16位无符号整数 unsigned short 来存储,因此本地端口一共有2^16=65536个,即0-65535,其中0~1023是预留端口,0有特殊含义不能使用,1024以下端口都是超级管理员用户(如 root)才可以使用,因此就算使用 root 权限,一台机器最多能使用的端口也只有65535个