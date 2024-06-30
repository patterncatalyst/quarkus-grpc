# Quarkus gRPC Add Interceptor.

Let’s add to the implementation of the gRPC service. The implementation class should be annotated with `@GrpcService`. 

In order to interact with a database reactively we should also consider annotating with `@WithSession.`  It creates a Mutiny session for each gRPC method inside. 

We can also register optional interceptors, for example, to log incoming requests and outgoing responses.

There are limitless reasons why this is useful as you can probably already infer, but to list a few common use cases:

- Tracing (i.e. viewing the flow of data through an application from request to
response, time spent processing on the server and more ([read our tracing blog here!](https://edgehog.blog/tutorial-how-to-implement-jaeger-and-opentracing-as-tracing-middleware-e3e693ee0802))
- Authorization (making sure that an authenticated user is actually allowed to perform the action)
- Adding metadata to a request such as environment configuration of the client.

Let’s add that for a logger and then add the implementation of the logger.

Add the /org/acme/LogInterceptor Java Class.

```jsx
package org.acme;

import io.grpc.*;
import jakarta.enterprise.context.ApplicationScoped;
import org.jboss.logging.Logger;

@ApplicationScoped
public class LogInterceptor  implements ServerInterceptor {

    Logger log;

    public LogInterceptor(Logger log) {
        this.log = log;
    }

    @Override
    public <ReqT, RespT> ServerCall.Listener<ReqT> interceptCall(ServerCall<ReqT, RespT> call, Metadata headers, ServerCallHandler<ReqT, RespT> next) {

        ServerCall<ReqT, RespT> listener = new ForwardingServerCall.SimpleForwardingServerCall<ReqT, RespT>(call) {
            @Override
            public void sendMessage(RespT message) {
                log.infof("[Sending message] %s",  message.toString().replaceAll("\n", " "));
                super.sendMessage(message);
            }
        };

        return new ForwardingServerCallListener.SimpleForwardingServerCallListener<ReqT>(next.startCall(listener, headers)) {
            @Override
            public void onMessage(ReqT message) {
                log.infof("[Received message] %s", message.toString().replaceAll("\n", " "));
                super.onMessage(message);
            }
        };
    }
}
```

Add the following to the HelloService class or any class you want this to intercept messages for.

```jsx
@RegisterInterceptor(LogInterceptor.class)
```

With Quarkus gRPC we can implement a server interceptor by creating the `@ApplicationScoped` bean implementing `ServerInterceptor`. 

In order to apply an interceptor to all exposed services, we should annotate it with `@GlobalInterceptor`. 

In our case, the interceptor is registered to a single service with `@RegisterInterceptor` annotation. Then we will use the `SimpleForwardingServerCall` class to log outgoing messages, and `SimpleForwardingServerCallListener` to log outgoing messages.