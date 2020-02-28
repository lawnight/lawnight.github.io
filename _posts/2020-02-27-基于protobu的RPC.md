---
title: RPC over protobuf
categories: server
---
# 1. protobuf对RPC的支持

在`.proto`文件中在可以定义service接口。
```c
option java_generic_services = true;
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

在设置`java_generic_services`为`true`后，会根据service的定义生成响应代码。能生成无关的抽象类和接口，不和任意特定实现的RPC关联。

基于`proto`服务定义的RPC，应该以插件的方式，来生成需要的代码。比如grpc就提供了代码生成插件，会根据service定义生成单独一个类，用于RPC操作。
```bash
protoc --plugin=protoc-gen-grpc-java=protoc-gen-grpc-java.exe --grpc-java_out=..\java\  "A.proto" 
```
# 2. RPC的具体实现

protobuf官方文档列举了多个基于protobuf实现的[RPC](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md),其中使用最广泛的就是google自己实现的GRPC。


## 谷歌grpc

grpc是谷歌根据内部Stubby RPC系统开源出来的。并且利用PDY, HTTP/2, and QUIC等公开的标准，使其可以在手机，物联网，云上使用。grpc支持多种语言。

![](/assets/proto1.png)

grpc主要使用http2协议。http2协议兼容http 1.1，并提供了新的数据封装和传输方式，用来更高效的数据交换。http2的多路复用，在一个TCP连接上可以发起多个请求。

grpc的stream类型。在面对需要传输大量数据的时候，可以不断的持续发送数据，降低http的延时和head占用的字节。
```bash
rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
```

**grpc用http2的协议进行传输，不支持服务端的推送，更多的使用场景是面向微服务和web，所以并不满足在游戏中前后端的交互。**

## 其它实现

其它实现基本都支持一种语言。其中java端的实现

蚂蚁金服开源的SOFARPC。

百度开源的




# 参考
>1. Java Generated Code: https://developers.google.com/protocol-buffers/docs/reference/java-generated#service
>2. gRPC: https://grpc.io/
>3. http2: https://en.wikipedia.org/wiki/HTTP/2
>4. gRPC Motivation and Design Principles: https://grpc.io/blog/principles/