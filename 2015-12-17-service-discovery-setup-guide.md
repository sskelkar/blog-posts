## Step by step guide to set up a service discovery environment
A step by step guide to set up a eureka service, to which we will register a demo server service and *discover* it from a demo client service.

### Steps to configure a eureka service
1. Create a new Gradle project for the eureka service. In https://start.spring.io/, select the starters for Eureka server.

2. In your project, navigate to `src/main/resources`. Rename the automatically generated `application.properties` file to `bootstrap.yml`.
3. Configure the `bootstrap.yml` as following:
```python
spring:
  application:
    name: demo-eureka-server
server:
  port: 8002
eureka:
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://localhost:${server.port}/eureka/
```
Here we declared the application name and port number for our eureka server as `8002`. Rest of the configuration tells this application to not to register itself with the available Eureka instance, which is this case is itself!

4. Annotate the application’s main class with `@EnableEurekaServer`.

5. Run the application. Your Eureka server is ready.

### Steps to configure a server service that will register itself to Eureka
1. Create a new Gradle project for the server service. In https://start.spring.io/, select the starters for web and Eureka Discovery.

2. In your project, navigate to `src/main/resources`. Rename the automatically generated `application.properties` file to `bootstrap.yml`.
3. Configure your `bootstrap.yml` like following:
```python
spring:
  application:
    name: demo-server-service
server:
  port: 8080
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8002/eureka/
```
Spring Cloud based services have a `spring.application.name` property. It is used to pull down configuration from the Configuration server, to identify the service to Eureka, and is referenceable in numerous other contexts when building Spring Cloud based applications. This value typically lives in `src/main/resources/bootstrap.yml`, which is picked up earlier in the initialization than the normal `src/main/resources/application.yml`.
4. Annotate the application’s main class with `@EnableDiscoveryClient`.

5. Let's also quickly write a rest endpoint that we will call from the client service. Create a controller class with following content:
```java
@RestController
public class DemoServerController {
    @RequestMapping(value="/greet",method=RequestMethod.GET)
    public String greet(@RequestParam String name) {
        return "Hello " + name;
    }
}
```
6. Run the application. In the command prompt you will see the service registering itself to the Eureka service.

### Steps to configure a client service
1. Create a new Gradle project for the server service. In https://start.spring.io/, select the starters for web, Feign and Eureka Discovery.

2. In your project, navigate to `src/main/resources`. Rename the automatically generated `application.properties` file to `bootstrap.yml` and configure it as following:
```python
spring:
  application:
    name: demo-client-service
    
server:
  port: 8111 
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8002/eureka/

```
3. Annotate the application’s main class with `@EnableDiscoveryClient` and `@EnableFeignClients`.

4. Our aim is to call `demo-server-service`’s API via our client service. So we need to create an interface that will acts as our REST client for talking to `demo-server-service`. Create an interface as following:
```java
@FeignClient("demo-server-service")
public interface DemoServerClient {
    @RequestMapping( method=RequestMethod.GET, value="/greet")
    public String greet(@RequestParam("name") String name); 
}
```
We have annotated the interface with `@FeignClient` that takes the name of the target service as it is registered at the Eureka service. The signature of greet method is identical to the way it is declared in `demo-server-service`’s public API.

5. Now to verify that everything is working fine, lets create a rest endpoint in our client service, which will delegate its call to `demo-server-service`’s `greet` method.
```java
@RestController
public class DemoClientController {
    @Autowired
    DemoServerClient demoServerClient;
  
    @RequestMapping(value="/hello",method=RequestMethod.GET)
    public String hello(@RequestParam String name) {
        return demoServerClient.greet(name);
    }
}
```
Note that Spring will automatically inject an implementation for `DemoServerClient` interface.

6. Hit http://localhost:8111/hello?name=World from your browser and it will print “Hello World”.

> Important: Before trying the API from client service, make sure that both `demo-server-service` and `demo-client-service` are registered at the Eureka service.

