## Hystrix – a simple example
Hystrix is a fault tolerance library that is very useful for managing failures in a distributed environment like microservices. Suppose we have a service `A` dependent  on service `B`, which is in turn dependent on service `C`.
```
A -> B -> C
```
Let's say a call is made from `A` to `B`. To serve this request, `B` needs to call `C` but there’s a communication failure between them. If the call from `B` to `C` is wrapped in Hystrix mechanism, we prevent the failure being propagated to `A`. Because `B` couldn’t fetch the actual information from `C`, Hystrix gives us the option of declaring a fallback value to be used in this case, if feasible.

In fact, Hystrix goes one step further. If we have a high volume of requests going from `B` to `C` per unit of time, and the rate of failure reaches some particular value say 50%, then Hystrix will stop any further requests for a period of time and instead return the fallback value immediately. In Hystrix terminology the circuit is said to be `open` in this case.

After a pre-configured time has elapsed, Hyxtrix will attempt to process the next request by calling service `C`. This is the `half-open` state. If the call still fails, the system revert backs to `open` state. But if it is successful, the circuit is `closed` and for all subsequent requests, service `C` is called.

