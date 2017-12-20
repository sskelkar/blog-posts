## Logging with Spring AOP
Aspect oriented programming (AOP) is a way of separating the business login in your code from cross cutting concerns. What is a cross cutting concern?

Analogy time. A typical house has different rooms that have designated functions. We keep our stuff in the rooms where they make sense. The living room is an unlikely location for a dishwasher and a bathtub belongs in the bathroom. But the electric circuit runs throughout the house because it is not tied to the functionality of any specific room. Thus, the electric circuit is a cross-cutting concern.

Similarly, an application code is composed of the business logic and some infrastructural code that is independent of the problem domain and may be scattered throughout the application. This may include logging, security or transaction management.

In this blog post I will use AOP concepts to implement logging in a simple application. But first, lets get acquainted with some AOP jargon:
**Aspect**: A cross cutting concern. In our case it is logging.
**Join point**: The point in our program flow where the aspect kicks in. Spring AOP supports only method execution as join points.
**Advice**: Action taken by an aspect at a join point. In our case it will be a log statement. An advice can execute before and after a join point, and after a joint point completed normally or after a joint point exited with exception.
**Pointcut**: This is an expression that specifies some conditions about the location of the join points where the advice must be applied.

The complete code of the toy application used in this blog is available in [this github repo](https://github.com/sskelkar/demo-aop).

The main class is annotated with `@EnableAspectJAutoProxy` to enable support for handling of components marked with `@Aspect` annotation. This is important because we need to tell Spring that we are using annotation based AOP configuration.

There is a class named `SelfDrivingCar` that calls several methods from its helper classes `Engine` and `Navigator` to finish a test drive. We want to log whenever these methods are called. If a method throws an exception, that must also be logged. If a method takes in an input, that must be logged as well.

I have created an aspect class for this purpose, `com.maxxton.demo.aop.Logging`. I have declared three advices. Each advice has an associated pointcut that tells Spring where to apply that advice.
```java
@Component
@Aspect
public class Logging {
  private static final Logger LOGGER = Logger.getLogger(Logging.class);

  @Before("execution(* com.maxxton.demo.car.*.*())")
  public void noArgumentMethodInvoked(JoinPoint joinPoint) {
    LOGGER.info("calling " + joinPoint.getSignature().getName());
  }
  
  @Before("execution(* com.maxxton.demo.car.Navigator.move(..)) && args(distance)")
  public void logDistance(JoinPoint joinPoint, Long distance) {
    LOGGER.info("moving by " + distance + "m");
  }
  
  @AfterThrowing(pointcut="within(com.maxxton.demo..*)", throwing="ex")
  public void logException(JoinPoint joinPoint, Exception ex) {
    LOGGER.error("Some problem occurred in " + joinPoint.getSignature().getName() + ", error message - " +  ex.getMessage());
  }
}
```
Consider the first advice `noArgumentMethodInvoked`. The pointcut expression for this advice is `execution(* com.maxxton.demo.car.*.*())`. It tells Spring to weave this advice to any method belonging to any class within the `com.maxxton.demo.car` package that doesn’t take any argument. The second advice has a more fine grained pointcut expression that contains two pointcut designators, `execution(* com.maxxton.demo.car.Navigator.move(..)) && args(distance)`. It specifies the method `move` of `Navigator`, which takes at least one argument. `args` can be used to capture a method’s arguments. In this case it is a single argument `distance`. When the application starts and `move` method is called, the value of `distance` passed to it will be captured and passed to the advice `logDistance` where it can be logged.

There is a third advice to log the exceptions thrown my any method belonging to any class within com.maxxton.demo package. The exception is captured by the advice its message can be logged inside the advice body.

Here’s a screenshot of the logs on running this application:
![log](https://github.com/sskelkar/blog-posts/raw/master/images/spring-aop-1.png)

