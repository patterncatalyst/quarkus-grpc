# Quarkus gRPC Virtual Threads

By default, the Quarkus gRPC extension invokes service methods on an event-loop thread.
See the [Quarkus Reactive Architecture documentation](https://quarkus.io/guides/quarkus-reactive-architecture) for further details on this topic.
But, you can also use the [@Blocking](https://javadoc.io/doc/io.smallrye.reactive/smallrye-reactive-messaging-api/latest/io/smallrye/reactive/messaging/annotations/Blocking.html) annotation to indicate that the service is *blocking* and should be run on a worker thread.

The idea behind Quarkus Virtual Thread support for gRPC services is 
to offload the service method invocation on virtual threads, instead of 
running it on an event-loop thread or a worker thread.

[Quarkus Virtual Thread support for gRPC services](https://quarkus.io/guides/grpc-virtual-threads)

Let’s add an example of running on a virtual thread.

To enable virtual thread support on a service method, simply add the [@RunOnVirtualThread](https://javadoc.io/doc/io.smallrye.common/smallrye-common-annotation/latest/io/smallrye/common/annotation/RunOnVirtualThread.html)
 annotation to the method.
If the JDK is compatible (Java 19 or later versions - we recommend 21+) then the invocation will be offloaded to a new virtual thread.
It will then be possible to perform blocking operations without blocking the platform thread upon which the virtual thread is mounted.

Make sure this is in your pom.xml file:

```Java
<properties>
    <maven.compiler.source>21</maven.compiler.source>
    <maven.compiler.target>21</maven.compiler.target>
</properties>
```

Add this to your Quarkus plugin section:

```Java
<plugin>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-maven-plugin</artifactId>
    <version>${quarkus.version}</version>
    <executions>
        <execution>
            <goals>
                <goal>build</goal>
                <goal>generate-code</goal>
                <goal>generate-code-tests</goal>
            </goals>
        </execution>
    </executions>
    **<configuration>
      <source>21</source>
      <target>21</target>
    </configuration>**
</plugin>
```

Now let’s add a couple of virtual thread methods.

Add the following to our proto file:

```Java
// The greeting service on virtual threads
service VirtualThreadGreeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}  // Uni
  rpc Add(stream Item) returns (stream Item) {}  // Multi
}
```

Now let’s implement it in a new class:

```Java
import io.quarkus.example.HelloReply;
import io.quarkus.example.HelloRequest;
import io.quarkus.example.Item;
import io.quarkus.example.VirtualThreadGreeter;
import io.quarkus.grpc.GrpcService;
import io.smallrye.common.annotation.RunOnVirtualThread;
import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.Uni;

@GrpcService
public class VirtualThreadGreeterService implements VirtualThreadGreeter {

    @Override
    @RunOnVirtualThread
    public Uni<HelloReply> sayHello(HelloRequest request) {
        return Uni.createFrom().item(() ->
                HelloReply.newBuilder().setMessage("Hello " + request.getName()).build()
        );
    }

    @Override
    @RunOnVirtualThread
    public Multi<Item> add(Multi<Item> request) {
        // Reads the incoming stream, compute a sum and return the cumulative results
        // in the outbound stream.
        return request
                .map(Item::getValue)
                .map(Long::parseLong)
                .onItem().scan(() -> 0L, Long::sum)
                .onItem().transform(l -> Item.newBuilder().setValue(Long.toString(l)).build());
    }
}
```

And now let’s test the calls:

```Java
 grpcurl --plaintext localhost:9000 list  
 grpcurl --plaintext localhost:9000 describe helloworld.VirtualThreadGreeter 
 grpcurl --plaintext -d '{"name":"hot dog"}' localhost:9000 helloworld.VirtualThreadGreeter.SayHello  
 grpcurl --plaintext -d @ localhost:9000 helloworld.VirtualThreadGreeter.Add  
```