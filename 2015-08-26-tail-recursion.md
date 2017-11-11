## Tail Recursion
Tail recursion is one of those functional programming concepts that are likely to be unknown to someone coming from a Java background, like me. I encountered this term while skimming through the first few pages of [SICP](https://mitpress.mit.edu/sicp/full-text/book/book.html). After some quick R&D (i.e. googling), the following is a summary of what I have learnt.

Before understanding tail recursion, we need to be familiar with the term **tail call**. Simply put, if in a function definition, the last instruction before returning is a function call, then that function call is called a tail call.

Examples:
```java
public int nextValueOf(int i) {
  System.out.println("The next value is: ");
  return increment(i); // tail call
}

public void nextToNextValueOf(int i) {
  System.out.println("The next to next value is: ");
  return 1 + increment(i); // not a tail call
}
```
**Tail recursion** is a special case of recursion where a function calls itself via a tail call.

### Normal recursion
We know that in normal recursion, a function calls itself repeatedly until the exit condition is met. For every function invocation, a new frame is created on the call stack. When the exit condition is satisfied, the call stack starts to *unwind*, thereby successively freeing up the space occupied by each stack frame. If we run out of memory while making a large number of stack frames, we get a `StackOverflow` exception.

As an example, suppose we want to write a function to compute factorial of a number. For this, we can write a recursive function like this:
```java
int factorial(int n) {
  if(n == 0)
    return 1;
  else
    return n * factorial(n-1);
}
```
If we call `factorial(4)`, stack frames will be created in following order:
```
factorial(4) -> factorial(3) -> factorial(2) -> factorial(1) -> factorial(0)
```
When n = 0, the recursion will stop and values will be returned in following order.
```
return 4 * 6 <- return 3 * 2 <- return 2 * 1 <- return 1 * 1 <- return 1
```
Finally, `24` will be returned from the first stack frame as the result of the call to `factorial(4)`. Notice that while we were stacking up function frames, the actual operation of multiplication, required to calculate factorial, was deferred till the exit condition was met.

### Tail recursion
We can rewrite the above method in tail recursive form in following way.
```java
int factorial(int n) {
  return factorialHelper(n, 1);
}

int factorialHelper(int counter, int product) {
  if(counter == 0)
    return product;
  else
    return factorialHelper(counter - 1, counter * product);
}
```
The call stack for `factorialHelper` would be something like this:
```
factorial(4, 1) -> factorial(3, 4) -> factorial(2, 12) -> factorial(1, 24) -> factorial(0, 24)
```
When `counter = 0`, the recursion will stop and the value `24` will simply be returned up the stack from each frame.

So far there is no difference in the implementation of the above two recursive methods in terms of space complexity. What makes tail recursion special is something called **tail call optimization**. Notice that in the tail recursion implementation no computation is performed after the tail call. We simply need to return the value that was returned by the tail call. What this means is that after the tail call has been made, there is no further need to maintain that frame on the call stack.

### Tail call optimization
It is a way to avoid allocating a new stack frame for tail call. So when a call to `factorial(3, 4)` is made in the above implementation, the same stack frame that was used for `factorial(4, 1)` will be reused. Instead of creating a new stack frame, the program counter would jump to the first instruction of the current stack frame and start executing with the values of counter and product as `3` and `4` respectively. Because we are using a single stack frame, there is no question of running out of memory no matter how many times the method is called recursively.

It is often much easier to formulate a wide variety of problems in terms of recursion, as compared to using loops. With tail call optimization, we get the benefit of recursion with the performance of iteration.

Java doesn’t support tail call optimization. In JavaScript, it has been introduced from [ES6](https://github.com/lukehoban/es6features#tail-calls).
