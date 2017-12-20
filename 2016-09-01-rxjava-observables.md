## Parallelizing time intensive operations with RxJava Observables
Recently I delved into the [RxJava](https://github.com/ReactiveX/RxJava) library. In this post I will demonstrate how RxJava Observables can be used to execute two long running tasks in parallel, so as to reduce their overall execution time.

While we can create threads for this purpose, an additional benefit of using Observables is that it provides a convenient way of collecting the results of the parallel tasks. With threads, this can get pretty complicated.
Lets consider a situation where we have a consumer class that depends on the result of two or more expensive independent tasks.
```java
public class Producer1 {
  public List<Integer> produce() {
    List<Integer> list = new ArrayList<Integer>();
    
    for(int i=0; i<5; i++) {
      System.out.println("Producer1 - " + i);
      try {
        Thread.sleep(1000);
      } catch(Exception e) {}
      list.add(i);
    }
    return list;
  }
}
```
```java
public class Producer2 {
  public List<Integer> produce() {
    List<Integer> list = new ArrayList<Integer>();
    
    for(int i=0; i<5; i++) {
      System.out.println("Producer2 - " + i);
      try {
        Thread.sleep(1000);
      } catch(Exception e) {}
      list.add(i);
    }
    return list;
  }
}
```
Note that the `produce()` method of each producer is going to take approx 5 seconds to execute. The time required to consume them sequentially would be the aggregate of the execution time of each producer.
```java
public class SequentialConsumer {
  public List<Integer> consume(Producer1 p1, Producer2 p2) {
    Long start = System.currentTimeMillis();
    
    List<Integer> result = new ArrayList<>();
    result.addAll(p1.produce());
    result.addAll(p2.produce());
    
    Long end = System.currentTimeMillis();
    
    System.out.println("Serial time elapsed: " + (end-start)/1000 + " seconds"); // prints 10 seconds
    return result;
  }
}
```
Now lets rewrite the above code using Observables. First, we need to identify the tasks that can be parallelized, and wrap them inside RxJava Observable objects. In our case, we will convert the `produce()` method invocations to Observable tasks by wrapping them in `Observable.just()` methods. Additionally, we would also like to defer the execution of our Observable tasks so that we can control when they get invoked. So we wrap our tasks inside `Observable.defer()`.

The way Observer model works is, we have an Observable which emits some information. And we have a subscriber or observer, that listens to the Observable and consumes the information emitted by it. So we need our Observable tasks to be subscribed on, so that the results emitted by them can be processed. We also need to execute them in parallel, which can be done by executing them in separate threads. This can be done by calling `subscribeOn(Schedulers.newThread())`.

Next, with the help of `Observable.zip()` function, we can specify how to collect or combine the results emitted by multiple Observables, once they are finished executing. The combined result itself is also returned as an Observable.

We pass our Observables to the zip method and a lambda expression whose input parameters are the individual results of the corresponding Observables in the same order as they were passed. For example, lets say we have three Observables o1, o2 and o3. They emit results o1Result, o2Result and o3Result. These results can be combined with help of the zip operator as following:
```java
Observable<Object> o1;
Observable<Object> o2;
Observable<Object> o3;

Observable.zip(o1, o2, o3, (o1Result, o2Result, o3Result) -> {
  // some code
});
```
In our example, the two Observables will each emit a list of Integers, which we can collect in another ArrayList and return from the `consume()` method.
We need to pause our main thread until all the parallel tasks are completed, so that we can collect their results. This can be done using the `toBlocking()` operator.
Finally, we  call the `single()` method to trigger the execution of our Observables and return the combined list of integers.

Here’s the complete code:
```java
public class ParallelConsumer {
  public List<Integer> consume(Producer1 p1, Producer2 p2) {
    Long start = System.currentTimeMillis();
    
    Observable<List<Integer>> task1 = Observable.defer(() -> Observable.just(p1.produce())).subscribeOn(Schedulers.newThread());
    Observable<List<Integer>> task2 = Observable.defer(() -> Observable.just(p2.produce())).subscribeOn(Schedulers.newThread());
    
    List<Integer> result = Observable.zip(task1, task2, (result1, result2) -> {
      List<Integer> list = new ArrayList<>();
      list.addAll(result1);
      list.addAll(result2);
      
      return list;
    }).toBlocking().single();
    
    Long end = System.currentTimeMillis();
    System.out.println("Serial time elapsed: " + (end-start)/1000 + " seconds"); // prints 5 seconds
    
    return result;
  }
}
```

