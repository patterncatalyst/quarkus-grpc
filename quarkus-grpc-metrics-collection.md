# Quarkus gRPC Metrics Collection

gRPC server metrics are automatically enabled when the application also uses the `[quarkus-micrometer](https://quarkus.io/guides/telemetry-micrometer)` extension.

Micrometer collects the metrics of all the gRPC services implemented by the application.

We can add the following to the pom.xml to enable micrometer.

```jsx
./mvnw quarkus:add-extension -Dextensions='micrometer-registry-prometheus'
```

Or

```jsx

<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-micrometer-registry-prometheus</artifactId>
</dependency>
```

If using micrometer and we want to disable the gRPC server metrics we can just add to the .properties file.

```jsx
quarkus.micrometer.binder.grpc-server.enabled=false
```

Let’s test the prometheus output.

Run the service again:

```jsx
grpcurl --plaintext -d  '{ "name":"Eric Deandrea" }'   localhost:9000 helloworld.Greeter.SayHello  
```

Then test in a browser and look for the service “Greeter” under *tags*.

```jsx
curl --request GET localhost:8080/q/metrics

or 

In a browser tab

localhost:8080/q/metrics 
```