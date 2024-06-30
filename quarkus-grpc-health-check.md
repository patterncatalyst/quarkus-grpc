# Quarkus gRPC Add Health Check

Add some health checks based on the following .proto file.  Fortunately, Quarkus very nicely created this for us when we added grpc to the project under the grpc.health.v1 package name.  We can see it when we list out the services:

```jsx
grpcurl --plaintext localhost:9000 list  
grpcurl --plaintext localhost:9000 describe grpc.health.v1.Health  
```

Add the small rye health check dependency.  When Quarkus SmallRye Health is added to the application, a readiness check for the state of the gRPC services will be added to the MicroProfile Health endpoint response, that is `/q/health`.

```jsx
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-smallrye-health</artifactId>
</dependency>
```

```jsx
curl localhost:8080/q/health   
```

smallrye-health could also provide a number of other built in health checks such as:

- `/q/health/live` - The application is up and running.
- `/q/health/ready` - The application is ready to serve requests.
- `/q/health/started` - The application is started.
- `/q/health` - Accumulating all health check procedures in the application.

Clients can specify the fully qualified service name to get the health status of a specific service or skip specifying the service name to get the general status of the gRPC server.

Let’s check the “server” status and check our “service”:

```jsx
 grpcurl --plaintext  localhost:9000 grpc.health.v1.Health.Check  
 grpcurl --plaintext  -d '{"service":"helloworld.Greeter"}' localhost:9000 grpc.health.v1.Health.Check   
```

## Do Note Use Below:

Create the implementation of the health check service.

```jsx
syntax = "proto3";

option java_multiple_files = true;
option java_package = "io.quarkus.example";
option java_outer_classname = "HealthCheckProto";

package acme;

message HealthCheckRequest {
  string service = 1;
}

message HealthCheckResponse {
  enum ServingStatus {
    UNKNOWN = 0;
    SERVING = 1;
    NOT_SERVING = 2;
  }
  ServingStatus status = 1;
}

service HealthService {
  rpc Check(HealthCheckRequest) returns (HealthCheckResponse);

  rpc Watch(HealthCheckRequest) returns (stream HealthCheckResponse);
}
```

```jsx

import grpc.health.v1.HealthOuterClass.HealthCheckRequest;
import grpc.health.v1.Health;
import grpc.health.v1.HealthOuterClass.HealthCheckResponse;
import io.quarkus.grpc.GrpcService;
import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.Uni;

@GrpcService
public class GrpcHealthService implements Health  {

    @Override
    public Uni<HealthCheckResponse> check(HealthCheckRequest request) {

        return Uni.createFrom().failure(new RuntimeException("Error!"));
    }

    @Override
    public Multi<HealthCheckResponse> watch(HealthCheckRequest request) {
        return Multi.createFrom().failure(new RuntimeException("Error!"));
    }
}
```

Add the properties to expose the gRPC health service when using OpenShift.

```jsx
quarkus.openshift.readiness-probe.grpc-action=9000:grpc.health.v1.HealthService
quarkus.openshift.startup-probe.grpc-action=9000:grpc.health.v1.HealthService
quarkus.openshift.liveness-probe.grpc-action=9000:grpc.health.v1.HealthService

```