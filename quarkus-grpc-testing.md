# Quarkus gRPC Testing

### Test using Postman

- Open up Postman
- Start a new grpc test
- Insert url=localhost:9000
- Under service definition, import the .proto file from the projects src/main/proto directory
- Choose the sayHello service
- Insert the following code

```jsx
{
	"name":"Erice Deandrea"
}
```

Save to a new project we will continue to use: quarkus-grpc-demo.

Then run the test.

### **Test using grpcurl.**

Use grpcurl to list the proto services.

```jsx
$ grpcurl --plaintext localhost:9000 list
```

Use grpcurl to print the list of methods exposed by the service.

```jsx
$ grpcurl --plaintext localhost:9000 list helloworld.Greeter
```

Use grpcurl to describe the details about each method.

```jsx
$ grpcurl --plaintext localhost:9000 describe helloworld.Greeter.SayHello
```

Use grpcurl to call the endpoint

$  grpcurl --plaintext -d  '{ "name":"Eric Deandrea" }'   localhost:9000 helloworld.Greeter.SayHello 

### Unit Testing

Let’s add some unit testing.

We will need @QuarkusTest and @GrpcClient and use Quarkus dev services and built-in integration with Testcontainers.  By default Quarkus starts gRPC endpoints on port 9000.  The Quarkus gRPC client works in the reactive mode, so we will leverage the Completable Future class to obtain and verify the results in the tests.

Let’s add jandex and surefire to our plugins in order to unit test this example.

```jsx

            <plugin>
                <groupId>io.smallrye</groupId>
                <artifactId>jandex-maven-plugin</artifactId>
                <version>3.1.7</version>
                <executions>
                    <execution>
                        <id>make-index</id>
                        <goals>
                            <goal>jandex</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${surefire-plugin.version}</version>
                <configuration>
                    <systemPropertyVariables>
                        <java.util.logging.manager>org.jboss.logmanager.LogManager</java.util.logging.manager>
                        <maven.home>${maven.home}</maven.home>
                    </systemPropertyVariables>
                </configuration>
            </plugin>
```

Add the unit test file HelloGrpcServiceTest:

```jsx

import static org.junit.jupiter.api.Assertions.assertEquals;

import java.time.Duration;

import io.quarkus.example.Greeter;
import io.quarkus.example.HelloReply;
import io.quarkus.example.HelloRequest;
import io.quarkus.grpc.GrpcClient;
import io.quarkus.test.junit.QuarkusTest;

import org.junit.jupiter.api.Test;

@QuarkusTest
public class HelloGrpcServiceTest {

    @GrpcClient
    Greeter greeter;

    @Test
    public void testHello() {
        HelloReply reply = greeter
                .sayHello(HelloRequest.newBuilder().setName("Neo").build()).await().atMost(Duration.ofSeconds(5));
        assertEquals("Hello Neo!", reply.getMessage());
    }

}
```

The test should fail because the original code does not add a bang “!” at the end of the name.

## DO NOT DO RIGHT NOW

### Testing with the Quarkus UI

We can also test using the Quarkus Dev UI.

We should add a dedicated profile to assist us with this.  Add to your pom.xml file.

```jsx
        <profile>
            <id>development</id>
            <dependencies>
                <dependency>
                    <groupId>io.quarkus</groupId>
                    <artifactId>quarkus-vertx-http</artifactId>
                </dependency>
            </dependencies>
        </profile>

```

Goto: [http://localhost:8080/q/dev-ui](http://localhost:8080/q/dev-ui) after running:

```jsx
$ mvn quarkus:dev -Pdevelopment
```

Click on the services link inside the gRPC tile.  We will be redirected to the site with a list of available gRPC services.