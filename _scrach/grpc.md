
Each stub wraps a Channel
gRPC Java generates code for three types of stubs: asynchronous, blocking, and future

### Asynchronous Stub
异步回调的方式，不会阻塞当前线程

### Blocking Stub

在调用hasNext的时候会阻塞线程。可以指定超时`blockingStub.withDeadlineAfter(1000, TimeUnit.MILLISECONDS)`

## Service Stub

```java
public void serverStreamingExample(RequestType request,StreamObserver<ResponseType> responseObserver)
{
    responseObserver.onNext(ResponseType response).
}
```

In gRPC Java, there are three types of stubs: blocking, non-blocking, and listenable future. We have already seen the blocking stub in the client, and the non-blocking stub in the server. The listenable future API is a compromise between the two, offering both blocking and non-blocking like behavior. As long as we don’t block a thread waiting for work to complete, we can start new RPCs without waiting for the old ones to complete.


https://grpc.io/blog/optimizing-grpc-part-1/
https://www.coder.work/article/1802949

grpc链接好像不会自动断开