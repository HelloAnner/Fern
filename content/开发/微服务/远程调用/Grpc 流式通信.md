gRPC 是一款高性能、开源的RPC框架，产自 Google，基于 ProtoBuf 序列化协议进行开发，支持多种语言（Golang、Python、Java等）。

一个最简单的通信就是一次服务端方法调用，对应的proto描述文件为：
```proto
package helloworld;
 
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}
 
// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}
 
// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

在实际应用中，部分场景下我们需要大数据量传输，在java中我们会使用 InputStream 或者 OutputStream 来流式传输数据，例如大文件的上传与下载，在gRPC中也有对应的流式调用。

gRPC中的流式接口与java中的Stream略有不同，在参数或者返回值中加入了stream关键字：
```proto
// 客户端往服务端发送流数据
service ForwardStreamingGreeter {
  rpc SayHelloStreaming (stream HelloRequest) returns (HelloReply) {}
}
 
// 服务端往客户端返回流数据
service ReverseStreamingGreeter {
  rpc SayHelloStreaming (HelloRequest) returns (stream HelloReply) {}
}
 
// 双向流数据
service StreamingGreeter {
  rpc SayHelloStreaming (stream HelloRequest) returns (stream HelloReply) {}
}
```

流通信在java中是通过 StreamObserver 来实现的，逻辑比较简单：
![[Pasted image 20231113194433.png|600]]

对于客户端往服务端发送的流，服务端实现observer接口，用于接收数据；客户端调用observer接口，用于发送数据。

对于服务端往客户端发送的流正好相反，客户端实现observer接口，用于接收数据；服务端调用observer接口，用于发送数据。

以双向流传输模式为例，代码调用如下：

服务端：
![[Pasted image 20231113194449.png]]

客户端：
![[Pasted image 20231113194500.png]]

逻辑很简单，但是代码看上去可能会有些奇怪，是因为在proto描述文件中我们定义的方法是：
```code
service StreamingGreeter {
  // Streams a many greetings
  rpc SayHelloStreaming (stream HelloRequest) returns (stream HelloReply) {}
}
```

![[Pasted image 20231113194558.png]]

这是由于gRPC支持很多语言，流式接口会针对每种编程语言生成最合适的形式，go语言中的格式跟java的就不一样，只需要按照生成代码的方法签名来实现和调用就行

使用流式接口时，gRPC只支持异步模式，即 Observer 中的逻辑在一个特定的线程池执行。
![[Pasted image 20231113194651.png|600]]

服务端与客户端有所不同，服务端的方法处理与后续的流消息处理都是在工作线程中串行执行；客户端发起方法调用与发送消息在主线程中执行（即调用方法的线程），而接收的流消息响应则与服务端一样在工作线程中串行执行。

服务端采用工作线程串行执行方法处理和消息处理，以确保方法调用和消息的顺序性和一致性。客户端在主线程中发起方法调用和消息发送，但在工作线程中串行执行接收的流式消息，以保持与服务端的同步。这种设计能够提供简洁高效的流处理能力，并适应不同的应用场景和需求。


https://kms.fineres.com/pages/viewpage.action?pageId=515312430



- [ ] Grpc 流式通信 复习时间 (@2023-11-27 21:00)