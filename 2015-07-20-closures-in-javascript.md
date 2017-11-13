## Closures in JavaScript
A good understanding of closures is a must-have skill for any JavaScript programmer. So lets take a look at how they work with two simple examples.

In JavaScript, functions are first class citizens. This means a function can be passed as an argument to another function, returned as the value from a function, assigned to a variable and stored in a data structure.

We can even write a function within a function, and the inner function has access to the *environment* within which it was created. A closure is a combination of a function and the environment in which it was created. This means an inner function can hold the scope of parent function even if the parent function has returned. Following example will make it a little clear.

```javascript
function adderFactory(x) {
    return function(y) {
        return x + y;
    }
}
 
var addBy5 = adderFactory(5);
var addBy10 = adderFactory(10);
 
console.log(addBy5(2));          // prints "7"
console.log(addBy10(2));         // prints "12"
```
In the above code, we have an outer function `adderFactory` with a local variable `x`. From this function, we return an unnamed function which refers to the variable `x`.

We then call the `adderFactory` with value 5. `adderFactory` exits after returning the unnamed function, which we store in the variable `addBy5`. Now this variable `addBy5` is bound to a function that adds 5 to the number passed to it and returns the sum.

Even after `adderFactory` has exited, the returned function still holds the value of `x` as 5. This is because a closure has been created.

Similarly, after the next call to `adderFactory`, the returned function holds the value of `x` as 10. A call to this function will return 10 added to whatever number is passed to it.

Now, lets create a closure with a slightly different syntax.

```javascript
var doublify;
 
(function (multiplier) {
    doublify = function(y) {
        return multiplier * y;
    }
})(2);
 
console.log(doublify(12)); // prints "24"
console.log(doublify(50)); // prints "100"
```
First we have a variable declaration in the global scope. Then we have an **immediately invoked function expression** that takes a parameter named `multiplier`. Inside the function we bind the variable `doublify` to a function that takes a number and returns the product of that number and the multiplier.

After the anonymous function definition, we immediately call it and pass 2. This value is now bound to the inner function. Each time we call `doublify`, 2 is multiplied to the parameter passed to it.

For someone coming from OOP background, a similarity between closures and objects will be immediately noticeable. Indeed, closures have been called as poor man’s objects [and vice-versa](https://news.ycombinator.com/item?id=926140)! In fact when we create a class in TypeScript, it gets translated into a JavaScript closure.
