# Quarkus gRPC Blocking

By default, the Quarkus gRPC extension invokes service methods on an event-loop thread.
See the [Quarkus Reactive Architecture documentation](https://quarkus.io/guides/quarkus-reactive-architecture) for further details on this topic.

But, you can also use the [@Blocking](https://javadoc.io/doc/io.smallrye.reactive/smallrye-reactive-messaging-api/latest/io/smallrye/reactive/messaging/annotations/Blocking.html) annotation to indicate that the service is *blocking* and should be run on a worker thread.

As a consequence, you must **not** block.
If your service logic must block, annotate the method with `io.smallrye.common.annotation.Blocking.`

Add the following to our proto file Greeter service:

```jsx
  rpc SayHelloBlocking(HelloRequest) returns (HelloReply) {}
```

Now let’s add the implementation in our GreeterService Java class:

```jsx
    @Override
    @Blocking
    public Uni<HelloReply> sayHelloBlocking(HelloRequest request) {
        // Do something blocking before returning the Uni
        return Uni.createFrom().item(() ->
                HelloReply.newBuilder().setMessage("Hello " + request.getName() + " from sayHelloBlocking").build()
        );
    }
```

Now let’s make sure we can see and run our new blocking method:

```jsx
grpcurl --plaintext localhost:9000 list  
grpcurl --plaintext localhost:9000 describe helloworld.Greeter.SayHelloBlocking  
grpcurl --plaintext -d '{"name":"Eric Deandrea"}' localhost:9000 helloworld.Greeter.SayHelloBlocking   
```