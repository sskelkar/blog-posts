## Representing natural numbers in lambda calculus
One of the joys of reading [SICP](https://mitpress.mit.edu/sicp/full-text/book/book.html) is that apart from the main subject matter, we come across many tangential topics that are interesting in their own right. One such topic is mentioned in `Exercise 2.6`: Church numerals. Named after the mathematician Alonzo Church, Church numerals are a way of representing natural numbers in lambda calculus. But what is λ-calculus?

From a programming perspective, λ-calculus can be thought of as the smallest universal programming language. It lacks some of the common features that one would expect in a programming language like, primitives, booleans, numbers etc. In this language, variable substitution and functions are used as the building blocks to express everything else. Even numbers! In this post we will get a glimpse of how this is achieved.

As a programmer, understanding code is much easier for me than trying to parse a bunch of mathematical notations. So for each λ-notation used in this post, I’ll give an equivalent JavaScript representation using its ES6 arrow functions.

### λ-notation
Before diving into λ-calculus, let’s first familiarise ourselves with λ-notations and how they are evaluated. A λ-notation is just a mathematical representation of a function. It starts with a list of arguments to the function, followed by its body. Here are some examples:
```
              λx. x+1            |             x => x + 1
             λx.λy. x+y          |           (x,y) => x + y
```
An alternative way of writing a λ-notation with multiple arguments is `λxy.x+y`. To see how variable substitution works in λ-calculus, we can evaluate the second expression by passing some real values into it:
```
(λx.λy. x+y) 1 2 = (λy. 1 + y) 2 
                 = 1 + 2 
                 = 3
```
### Booleans
As stated above, everything in λ-calculus is a function. So how do we represent boolean values? Using functions! True and False values are represented as functions that take in two arguments. `tru` simply returns the first argument and `fal` returns the second.
```
          tru = λx.λy. x        |           tru = (x, y) => x
          fal = λx.λy. y        |           fal = (x, y) => y
```
### Pairs
Now we can use these two functions as building blocks to represent more advanced data structures. Let’s consider a 2-tuple or an ordered pair. An ordered pair is a simple data structure that holds two values. A simple application of an ordered pair can be to hold the x-y coordinates of a point in a two dimensional space. The pair should be able to provide us the value of each of these coordinates on demand.

Here’s a functional representation of a pair in λ-calculus and JavaScript:
```
      pair = λx.λy.λf. f x y     |      pair = (x, y) => (f) => f(x,y)
```
The λ-notation of the `pair` function takes three arguments. `x` and `y` are coordinate values while the third argument is a function that is applied on the first two arguments. The JavaScript representation of this function is a curried form of the λ-notation. It accepts two arguments, as we would expect from a function representing a pair, and returns a lambda expression that accepts a function and applies it on the `x` and `y` arguments passed to pair.

We need to define two more functions to return the first and the second value of a pair:
```
           first = λp. p tru     |        first = (p) => p(tru)
          second = λp. p fal     |       second = (p) => p(fal)
```
`first` is a function that accepts a `pair` function and passes the `tru` function into it! Similarly, `second` passes `fal` into the `pair` passed into it.

Here’s how it works. Suppose we pass values 2 and 3 into a `pair` function, we expect 2 to be returned when `first` is applied on that pair. Here’s how the lambda expression would be evaluated:
```
first (pair 2 3) = first ((λx.λy.λf. f x y) 2 3)  //expand pair function
                 = first ((λy.λf. f 2 y) 3)       //substitute x with 2
                 = first (λf. f 2 3)              //substitute y with 3
                 = (λp. p tru) (λf. f 2 3)        //expand first 
                 = (λf. f 2 3) tru                //substitute p with the simplified pair expression
                 = tru 2 3                        //substitute f with tru. tru will always return its first arg
                 = (λx.λy.x) 2 3                  //expand tru. 
                 = (λy.2) 3                       //substitute x with 2. We get a constant function
                 = 2
```
We can verify this result in JavaScript by calling `first(pair(2,3))`.
### Church numerals
So we have seen how to use functions to represent boolean values. We will now go one step further and use functions to represent numbers!

Church numerals are a set of functions that can be used to formulate a number system. Just like the `tru` and `fal` above didn’t represent a concrete boolean value, Church numerals are not actual numerical values, but functional representations of whatever numerical system we want to build. We can use them to represent natural numbers, whole numbers, Roman numerals or even something akin to a basket of apples.

Any natural number system needs a starting *zero* value, from where the counting begins. For Roman numerals it is the value `I`. For whole numbers it is the value 0. For a basket of apples, a zero value can be an empty basket.

Apart from the zero value, a number system needs a way to compute the successor of a given numeric value. In integers, we get the successor of a number by adding 1 to it. In the basket of apples, we get to the next value by physically putting another apple into the basket.

So we can say that a number system has two properties: a starting point and an increment function to compute the next value. So if we have a zero value, we can get the next value by applying increment function on zero. To get its next value, we need to apply the increment function again on the result of the previous step. If we want to get the nth value in a number system, we need to apply the increment function n times to the zero value of that number system.

Let’s write some λ-notations to represent the first few values of an abstract number system:
```
        zero = λf.λz. z          |        zero = (inc, z) => z
         one = λf.λz. f z        |         one = (inc, z) => inc(z)
         two = λf.λz. f(f z)     |         two = (inc, z) => inc(inc(z))
       three = λf.λz. f(f(f z))  |       three = (inc, z) => inc(inc(inc(z)))
```
Here `f` is the increment function and `z` is the concrete zero value of our particular number system, as described above. Again, remember that `zero`, `one`, `two` and `three` above are functions and not concrete values. We can define a generalised function that takes a number `n` and generates the functional representation of its successor:
```
next = λn.λf.λz. f (n f z)       |       next = (n, f, z) => f(n(f, z))
```
We can pass `two` as `n` in the `next` function and expand the resulting λ-expression to demonstrate that on simplification, it comes out to be same as the function `three` above.
```
(next two) = (λn.λf.λz. f (n f z)) two            //expand next                     
           = λf.λz. f (two f z)                   //substitute n with two
           = λf.λz. f ((λa.λb. a(a b)) f z)       //expand two
           = λf.λz. f ((λb. f (f b)) z)           
           = λf.λz. f (f (f z))
           = three
```
We can pass concrete values into the corresponding JavaScript functions to verify that this number system works. To represent whole numbers, we can have an increment function as `inc = (x) => x + 1` and the zero value would be simply 0. So `zero(inc, 0)` comes out as 0. Calling `next(zero, inc, 0)` successively would yield the values 1, 2, 3 and so on.

### Addition
If we look at the Church numerals above, the representation of a number `n` simply applies the function `f` on the zero value `z`, *n* number of times. So if we want to add two numbers `a` and `b`, all we need to do is apply `f` on `z`, *(a+b)* times.

This is how `add` function is represented in λ-calculus:
```
add = λa.λb.λf.λz. a f (b f z)   | add = (a, b) => (f, z) => a(f, b(f, z))
```
What we are doing here is, for two numbers `a` and `b`, at first we are applying `f` on `z`, *b* number of times. Using its result as the new `z` value, we are applying the function `f` on it a further *a* times. For example, if we run `add(one, two, inc, 0)` in JavaScript, `inc` would be applied on `0`, 2 + 1 = 3 times. 

