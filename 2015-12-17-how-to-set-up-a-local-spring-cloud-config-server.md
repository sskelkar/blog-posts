## How to set up a local spring cloud config server
### Steps to configure config server
1. Create a new Gradle project for the config server. In https://start.spring.io/, select the starters for config server.
2. In your project, navigate to `src/main/resources`. Rename the automatically generated `application.properties` file to `bootstrap.yml`.
3. Spring cloud config uses a Git backend by default to lookup property sources, i.e., config files. For this demo application, we will create a git repository on our local file system.
4. Create an empty folder in your file system. Lets name it `config-repo`. Navigate to this repository via command prompt and execute `git init .` command. This will create an empty git repository in this folder. 
5. We need to provide our config server the location of this git repository. The default strategy for locating property sources is to clone a git repository declared against `spring.cloud.config.server.git.uri` property. So in our `bootstrap.yml`, we need to define this property in yaml format like this:
```python
spring:
  cloud:
    config:
      server:
        git:
          uri: file://(folder path)/config-repo
  
server:
  port: 8003
  ```
We can also declare on which port number the config server should run. We have declared that as `8003` above.
6. In your application’s main class add `@EnableConfigServer` annotation at the class level.
7. When a microservice requests for its property source from a config server, the config server looks up for its property file in following manner. For example, if the name of the requesting service is `demo-service` and it is running in development profile, then the config server will look for a file with name `demo-service-development.yml` in the git repository to serve the properties for that service.
8. Currently there are no property files in our git repository. Suppose that we are going to run a microservice named `my-service` in production mode, we need to create a file named `my-service-production.yml` in the git repository folder. Remember that simply creating the file in the folder doesn’t add it in the repository. To do that we need to execute following commands:
```
git add -A .
git commit -m "some commit message"
```
9. Lets add a dummy property `sample.property` in this file that we will use in our microservice.
```
sample:
  property: foobar
```

### Steps to configure config client
1. Create another Gradle project. In https://start.spring.io/, select the starters for config client and web.
2. In your project, navigate to `src/main/resources`. Rename the automatically generated `application.properties` file to `bootstrap.yml`.
3. To fetch properties from the config server, we need declare to three things: our service name, the active profile and address of the config server. These tasks can be accomplished by adding following in `bootstrap.yml`:
```python
spring:
  application:
    name: my-service
  profiles:
    active: production
  cloud:
    config:
      uri: http://localhost:8003
```
4. Now, to use the properties in our application we can define a utility class by the name of `PropertyUtil` inside which we will read the external properties and using this class we can inject those property values wherever required throughout our application.
5. So create a `PropertyUtil` class in the same package where application’s main class is present, or in a sub-package.
6. The content of the `PropertyUtil` may be like this: 
```java
@Configuration
@ConfigurationProperties
public class PropertyUtil {
    @Value("${sample.property}")
    private String property;
 
    public String getProperty() {
        return property;
    }
}
```
`@Value` injects the external property into the private variable `property`. Note that `@Value` cannot be used with static variables.
7. To demonstrate that the external property is fetched correctly, we can set up a rest controller as following:
```java
@RestController
public class MyServiceController {
    @Autowired
    private PropertyUtil property;
 
    @RequestMapping(value="/", method=RequestMethod.GET)
    public String property() {
        return property.getProperty();
    }
}
```
8. Now start the config-server application followed by `my-service` and hit http://localhost:8080 on your browser. Text “foobar” will be displayed.
