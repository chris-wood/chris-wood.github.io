---
layout: post
title: Rusty Results
---

Over the past couple of months, I've been building [a CCNx forwarder](https://github.com/chris-wood/iris) 
with Rust in my free time. What started as a small endeavor turned out to be 
a meaningful learning exercise. Programming in Rust forces you to change the way you 
think about code structure and data ownership -- for the better. Its syntax
leads to concise and expressive code that, for the most part, is quite easy to 
reason about. I am by no means fluent in the language, but I do believe that I 
have enough experience to discuss some of its features. In particular, I'd like
to talk about Results and their relation to error handling techniques. 

Technically, Results are of the type ```Result<T, E>```, where ```T``` is the type
of the result that some value is expected to contain (or wrap) under normal circumstances 
and ```E``` is the type of error that the value holds if something went wrong. To be more 
specific, the [Rust documentation website](https://doc.rust-lang.org/std/result/) defines a Result
as follows.

```
enum Result<T, E> {
   Ok(T),
   Err(E)
}
```

To see how this is useful, consider the following code. 

{% gist cfc1c646ef3f2500fc2f %}

Even without knowing Rust, the code should be easy to follow. We have
a function called ```quotient``` that takes two integers and attempts
to find their quotient. Before doing so, it does some checks. First, if 
the divisor is 0, then it returns a ```NumericError::DivideByZero``` error.
Second, if the remainder is non-zero, it returns a ```NumericError::NotEvenlyDivisible```. 
Otherwise, the function happily returns the evenly divisible quotient result. 
In all cases, a Result wraps the output of the function. As defined above
and in the code, this Result either carries an integer (i32) or a ```NumericError```.

The main function invokes the ```quotient``` function three separate times 
with a different set of parameters. We use pattern matching to check the result
of the function to determine how to proceed. If the result is "okay", i.e., ```Ok(q)```, then 
we print the integer quotient q. Otherwise, an error ```e``` occurred, i.e., ```Err(e)```,
and we print that instead.

Of course, this is a trivial example of how one might use ```Result```s. Despite
its simplicity, consider the immediate benefits:

- Conditionals that check for different types of errors do not clutter the code.
Rust pattern matching encourages concise code for handling wrapped results. 
- The programmer is forced to either manually unwrap the result of the function
(via the ```unwrap()``` function) or systematically handle and respond to each possible
error. Handling an error can either swallow it or propagate it upwards in 
yet another ```Result```.

To appreciate these benefits, let's turn to error handling in C. In general, 
errors are expressed through the output of a function. The actual values to be 
returned from a function are passed as "in and out" parameters, i.e., pointers
to values that are dereferenced and modified within the function. More often
than not, errors returned from functions are encoded as integers (0 is ubiquitous for OK and a non-zero
value is code for some error). These types of functions are especially prevalant in 
cases where multiple return types are needed. 

The code below mimics the Rust program above with an "in and out" parameter to store the quotient result.

{% gist a5709c129cf883664205 %}

The problem with this code is not the function signature itself. Rather, to me,
the deeper issue is how the function is invoked and its result is consumed. If you examine
the code in the main function above, you'll see that I used a series of cascading 
conditionals to check for all result types. Functionally, this is equivalent to the Rust
match expression, but it's an eyesore.

However, consider this slightly changed version of the Rust code.

{% gist 5ea3096cdb9ecb710ffe %}

In this version, the match expression explicltly checks against a type of ```NumericError```
to allow for more decisive handling. Fortunately, this fails to compile because the 
pattern matching is non-exhaustive (I omitted the ```NotEvenlyDivisible``` case). 
The compiler will ensure that we've at least written some code to handle each error
condition. The same is not true in C. If I omitted one of the conditional checks then 
the code would still happily compile. Sad, but true.

To make the situation a little better, it would be nice if we could have the readability
of Rust's pattern matching even if we cannot get the same compile-time pattern completeness checks.
To that end, I put together a little set of preprocessor macros that implement
Rust-like ```Results```. For me, they permit expressive code that is much easier to read than
the cascading conditional counterpart. Maybe others will find it useful too. 

{% gist cb2dc502d2a17acf5081 %}

You can see them being used in the program below.

{% gist 6e61797888b6b185a151 %}


