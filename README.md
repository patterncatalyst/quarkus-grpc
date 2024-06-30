# Quarkus gRPC Example

---

The purpose of this document is to create a composite location to quickly recreate
a Quarkus gRPC example using most of what would be required in a gRPC based application.

[Create a Quarkus gRPC Service](./doc/quarkus-grpc-service.md)

[Quarkus gRPC Testing](./doc/quarkus-grpc-testing.md)

[Quarkus gRPC Add Interceptor. ](./doc/quarkus-grpc-interceptor.md)

[Quarkus gRPC Blocking](./doc/quarkus-grpc-blocking.md)

[Quarkus gRPC Add Health Check](./doc/quarkus-grpc-health-check.md)

[Quarkus gRPC Metrics Collection](./doc/quarkus-grpc-metrics-collection.md)

[Quarkus gRPC Scaling](./doc/quarkus-grpc-scaling.md)

[Quarkus gRPC Streaming](./doc/quarkus-grpc-streaming.md)

[Quarkus gRPC Virtual Threads](./doc/quarkus-grpc-virtual-threads.md)

## Tooling Requirements to run example

---

> [!NOTE]
> The current environments are Mac M1 and Fedora AMD architectures.

| Tool    | macOS                                    | Fedora                                                                   | Notes                                                                                                                                                                                                                                                                                                                 |
|:--------|:-----------------------------------------|:-------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Quarkus | [macOS](https://quarkus.io/get-started/) | [Fedora](https://quarkus.io/get-started/)                                |                                                                                                                                                                                                                                                                                                                       |
| Postman | <nobr>brew install --cask postman</nobr> | flatpak install postman <br> or <br> snap install postman                | Make sure to have openssl installed if using Fedora 39+: <br> <nobr> $ cd ~/.var/app/com.getpostman.Postman/config/Postman/proxy/ </nobr> <br><nobr> $ openssl req -subj '/C=US/CN=Postman Proxy' -new -newkey rsa:2048 -sha256 -days 365 -nodes -x509 -keyout postman-proxy-ca.key -out postman-proxy-ca.crt </nobr> |
| grpcurl | brew install grpcurl                     | sudo dnf -y grpcurl                                                      |
| grpcui  | brew install grpcui                      | <nobr>go install github.com/fullstorydev/grpcui/cmd/grpcui@latest</nobr> |                                                                                                                                                                                                                                                                                                                       |


