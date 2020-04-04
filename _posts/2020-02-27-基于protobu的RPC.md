---
title: 基于protobuf的RPC
categories: server
---
## 1. protobuf对RPC的支持

在`.proto`文件中在可以定义service接口。
```c
option java_generic_services = true;
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

设置`java_generic_services`为`true`，`protoc`会根据service的定义生成相应代码。生成的代码不和任意特定实现的RPC关联，只是相关的抽象类和接口。我们可以实现`RpcChannel`和`RpcController`接口来将生成的代码和特定实现的RPC系统进行关联。

除了上面的方式，还能以提供[protc插件](https://developers.google.com/protocol-buffers/docs/reference/cpp#google.protobuf.compiler)的方式，来生成service定义的代码。比如`grpc`就提供了代码生成插件，会根据定义生成单独一个类，用于RPC操作。
```bash
protoc --plugin=protoc-gen-grpc-java=protoc-gen-grpc-java.exe --grpc-java_out=..\java\  "A.proto" 
```

## 2. RPC的一些具体实现

protobuf官方文档列举了多个基于protobuf实现的[RPC](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md),其中使用最广泛的就是google自己实现的`GRPC`。


### 谷歌grpc

grpc是谷歌根据内部Stubby RPC系统开源出来的。并且利用PDY, HTTP/2, and QUIC等公开的标准，使其可以在手机，物联网，云上使用。grpc支持多种语言。

![](/assets/proto1.png)

grpc主要使用http2协议。http2协议兼容http 1.1，并提供了新的数据封装和传输方式，用来更高效的数据交换。http2的多路复用，在一个TCP连接上可以发起多个请求。

grpc的stream类型。在面对需要传输大量数据的时候，可以不断的持续发送数据，降低http的延时和header占用的字节。
```bash
rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
```

**综上：grpc使用http2的协议进行传输，更多的适合面向微服务和web的使用场景；其http头的占用，和不支持双向RPC（后端不能主动推送），不适合用来实现在实时游戏中的前后端交互。**

### 其它实现

其它基于protobuf的实现一般都只支持一种语言。其中java端的实现主要有蚂蚁金服开源的`SOFARPC`和百度开源的`Jprotobuf-rpc-socket`。

sofarpc，支持`bolt`，`dubbo`，`RESTful`等多种协议，舍弃proto定义服务的方式，以`xml`和`Annotation`的方式定义服务，以api的方式使用protobuf，作为序列化的工具。

Jprotobuf-rpc-socket，基于更底层的TCP，沿用proto定义服务的方式。

**这些实现分别因为下列一个或多个原因：缺少多语言，不支持双向RPC，传输协议的耦合，更新停滞；不能很好的满足我们的需求，但是还是有值得借鉴的地方。**

## 3. 结论

**后续会继续寻找，分析更多的RPC库，最后再探讨是否需要自实现一套高效RPC系统，来满足我们的需求。**


## 参考
>1. Java Generated Code: https://developers.google.com/protocol-buffers/docs/reference/java-generated#service
>2. gRPC: https://grpc.io/
>3. http2: https://en.wikipedia.org/wiki/HTTP/2
>4. gRPC Motivation and Design Principles: https://grpc.io/blog/principles/
>5. proto2 Defining Services: https://developers.google.com/protocol-buffers/docs/proto#services
>6. Support 2-way RPC on the same connection: https://github.com/grpc/grpc-go/issues/484