# Quarkus gRPC Streaming

Let’s add a stream to our HelloWorld.

Add the following to our pom.xml so we can use the common types if we haven’t already.

Well known types:

[https://protobuf.dev/reference/protobuf/google.protobuf/](https://protobuf.dev/reference/protobuf/google.protobuf/)

Well known types Maven repo:

[https://mvnrepository.com/artifact/com.google.protobuf/protobuf-java](https://mvnrepository.com/artifact/com.google.protobuf/protobuf-java)

Common types:

[https://github.com/googleapis/api-common-protos?tab=readme-ov-file](https://github.com/googleapis/api-common-protos?tab=readme-ov-file)

Common Types Maven repo:

[https://mvnrepository.com/artifact/com.google.api.grpc/proto-google-common-protos](https://mvnrepository.com/artifact/com.google.api.grpc/proto-google-common-protos)

```jsx
        <dependency>
            <groupId>com.google.api.grpc</groupId>
            <artifactId>proto-google-common-protos</artifactId>
        </dependency>
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java</artifactId>
        </dependency>
```

In the proto file add the following:

```jsx
// Add above package
import "google/protobuf/empty.proto";

// Then add at the bottom
service Streaming {
  rpc Source(google.protobuf.Empty) returns (stream Item) {} // Returns a stream
  rpc Sink(stream Item) returns (google.protobuf.Empty) {}   // Reads a stream
  rpc Pipe(stream Item) returns (stream Item) {}  // Reads a streams and return a streams
}

// Define the item
message Item {
  string value = 1;
}
```

Now, let’s implement the service.

Add a StreamingService class that implements Streaming from protobuf.

```jsx
package org.acme;

import com.google.protobuf.Empty;
import io.quarkus.example.Item;
import io.quarkus.example.Streaming;
import io.quarkus.grpc.GrpcService;
import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.Uni;

import java.time.Duration;

@GrpcService
public class StreamingService implements Streaming {

    @Override
    public Multi<Item> source(Empty request) {
        // Just returns a stream emitting an item every 2ms and stopping after 10 items.
        return Multi.createFrom().ticks().every(Duration.ofMillis(2))
                .select().first(10)
                .map(l -> Item.newBuilder().setValue(Long.toString(l)).build());
    }

    @Override
    public Uni<Empty> sink(Multi<Item> request) {
        // Reads the incoming streams, consume all the items.
        return request
                .map(Item::getValue)
                .map(Long::parseLong)
                .collect().last()
                .map(l -> Empty.newBuilder().build());
    }

    @Override
    public Multi<Item> pipe(Multi<Item> request) {
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

Let’s test using grpcurl accumulating values:

```jsx
grpcurl --plaintext -d @ localhost:9000 helloworld.Streaming.Pipe   
```