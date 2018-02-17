# why
Personal notes and code samples for the Y-combinator.

## Links
- http://mvanier.livejournal.com/2897.html - A fantastic detailed look
  at the Y-combinator, implementation in scheme, as well as some
  mathematical background on it.
- https://medium.com/@ayanonagon/the-y-combinator-no-not-that-one-7268d8d9c46 -
  Short and sweet look at the Y-combinator and also introduces lambda
  calculus which is a plus.
- https://en.wikipedia.org/wiki/Fixed-point_combinator

## Technology Related Concerns
Most examples are in scheme and only work if the particular
implementation of scheme is "lazy". I used racket as my lazy scheme
interpreter so download that if you want to follow along:
- http://racket-lang.org/
- http://docs.racket-lang.org/lazy/index.html

## Notes
Recursion is the act of defining something in terms of itself. For a
function this means that the function calls itself as part of its
definition. For example here is the factorial function in scheme:
```scheme
(define factorial
  (lambda (n)
    (cond
     ((= n 0) 1)
     (else (* n (factorial (- n 1)))))))
```

It turns out that even if our language has no `define` or looping
constructs we can still create a `factorial` function (or generally
any function that needs to do any sort of repetition). Let that
sentence sink in a bit, because I think it's freakin' crazy.

In going through the process of defining this "non-recursive"
`factorial` function we will learn about the Y-combinator.

### High level definition of the Y-combinator
On a high level the Y-combinator is this function:
```
Y f = f (Y f)
```

Or in scheme:
```scheme
(define Y
  (lambda (f)
    (f (Y f))))
```

So it is a function that takes another function `f` and applies `f` to
the result of `Y f`. It looks a little non-sensical but when given the
proper function `f` it creates a recursive version of `f`. So if we
define some function `almost-factorial` then `(Y almost-factorial)`
**is** our `factorial` function.

### Using that Y-combinator
Here is the function `almost-factorial` which when passed into `Y`
will create the function `factorial`:
```scheme
(lambda (factorial)
  (lambda (n)
    (cond
     ((= n 0) 1)
     (else (* n (factorial (- n 1)))))))
```

So, assuming we have `Y` at our disposal, our `factorial`
function looks like this:
```scheme
(Y
 (lambda (factorial)
   (lambda (n)
     (cond
      ((= n 0) 1)
      (else (* n (factorial (- n 1))))))))
```

Note there is **no recursion** because there is no `define`. `Y` is
recursive at this point in the example, but I've only done that to
have a high level understanding of what `Y` does. Once we go through
an example and convince ourselves that if a construct like `Y` exists
then we can get "recursion without recursion" we can define `Y` in
terms of just `lambda`'s and be done.

### An example
Let's compute factorial of 3:
```scheme
((Y
  (lambda (factorial)
    (lambda (n)
      (cond
       ((= n 0) 1)
       (else (* n (factorial (- n 1))))))))
 3)
```

Before applying 3, we must first evaluate:
```scheme
(Y
 (lambda (factorial)
   (lambda (n)
     (cond
      ((= n 0) 1)
      (else (* n (factorial (- n 1))))))))
```

Remember that `(Y f) == (f (Y f))` so first we get:
```scheme
((lambda (factorial)
   (lambda (n)
     (cond
      ((= n 0) 1)
      (else (* n (factorial (- n 1)))))))
 (Y
  (lambda (factorial)
    (lambda (n)
      (cond
       ((= n 0) 1)
       (else (* n (factorial (- n 1)))))))))
```

For conciseness let's label the `(lambda (factorial) ...)` bit `f`.
So the previous example could be rewritten as:
```scheme
((lambda (factorial)
   (lambda (n)
     (cond
      ((= n 0) 1)
      (else (* n (factorial (- n 1)))))))
 (Y f))
```

We apply `(Y f)` to the outermost lambda giving us (and keep in mind that
`f` stands for that `(lambda (factorial) ...)` bit:
```scheme
(lambda (n)
  (cond
   ((= n 0) 1)
   (else (* n ((Y f) (- n 1))))))
```

Now we apply 3 to the above lambda:
```scheme
((lambda (n)
   (cond
    ((= n 0) 1)
    (else (* n ((Y f) (- n 1))))))
 3)
```

`3 != 0` so we go to the `else` which produces:
```scheme
(* 3 ((Y f) (- 3 1)))
```

Expanding the `f` to be the full `(lambda (factorial) ...)` that it
really is and evaluating `(- 3 1)` the above line is really:
```scheme
(* 3
   ((Y (lambda (factorial)
	 (lambda (n)
	   (cond
	    ((= n 0) 1)
	    (else (* n (factorial (- n 1))))))))
    2))
```

So we are multiplying 3 times the result of:
```scheme
((Y
  (lambda (factorial)
    (lambda (n)
      (cond
       ((= n 0) 1)
       (else (* n (factorial (- n 1))))))))
 2)
```

Before applying 2, we must first evaluate:
```scheme
(Y
 (lambda (factorial)
   (lambda (n)
     (cond
      ((= n 0) 1)
      (else (* n (factorial (- n 1))))))))
```

Wait, this looks familiar... we already did this evaluation! And it
produced:
```scheme
(lambda (n)
  (cond
   ((= n 0) 1)
   (else (* n ((Y f) (- n 1))))))
```

Now we continue evaluation:
```scheme
(* 3
   ((lambda (n)
      (cond
       ((= n 0) 1)
       (else (* n ((Y f) (- n 1))))))
    2))
```

`2 != 0` so we'll have to evaluate:
```scheme
(* 2 ((Y f) (- 2 1)))
```

Expanding the `f` to be the full `(lambda (factorial) ...)` that it
really is and evaluating `(- 2 1)` the above line is really:
```scheme
(* 2
   ((Y (lambda (factorial)
	 (lambda (n)
	   (cond
	    ((= n 0) 1)
	    (else (* n (factorial (- n 1))))))))
    1))
```

Adding back the `(* 3` the full picture is:
```scheme
(* 3
   (* 2
      ((Y (lambda (factorial)
	    (lambda (n)
	      (cond
	       ((= n 0) 1)
	       (else (* n (factorial (- n 1))))))))
       1)))
```

At this point I hope you're convinced that what we have here is a
proper factorial function. And more generally I hope you're convinced
that if we have a function `Y` we can create recursive functions such
as `factorial`.

### Defining Y without recursion
The above example is really cool but we used a `Y` that was defined
recursively. To complete the circle here is a definition of `Y`
without explicit recursion:
```scheme
(lambda (f)
  ((lambda (g) (f (g g))) (lambda (g) (f (g g)))))
```

This can also be written with a little less repetition as:
```scheme
(lambda (f)
  ((lambda (g) (g g))
   (lambda (g) (f (g g)))))
```

But we'll use the first definition, as it is easier to see the parallels with our
high level definition of `Y`. These are the parallels:
```
(lambda (f) ---- (define Y (lambda (f)
  ((lambda (g) (f (g g))) (lambda (g) (f (g g))))) ---- (f (Y f))
```

Let's pass in some arbitrary function called `fn` to convince
ourselves that this works:
```scheme
((lambda (f)
   ((lambda (g) (f (g g))) (lambda (g) (f (g g)))))
 fn)
```

Passing in `fn` we get:
```scheme
((lambda (g) (fn (g g))) (lambda (g) (fn (g g))))
```

And evaluating that lambda (note that a function is passing itself as
an argument!!!):
```scheme
(fn ((lambda (g) (fn (g g))) (lambda (g) (fn (g g)))))
```

Hopefully you can see the paralles and convince yourself that we have
ourselves a definition of `Y` without any explicit recursion. We now
have recursion without `define`!
```
(fn ((lambda (g) (fn (g g))) (lambda (g) (fn (g g)))))
     |                                              |
 fn  +-------------------- Y fn --------------------+
```

### Putting it all together
Putting everything together our factorial function sans recursion is:
```scheme
((lambda (f)
   ((lambda (g) (g g))
    (lambda (g) (f (g g)))))
 (lambda (factorial)
   (lambda (n)
     (cond
      ((= n 0) 1)
      (else (* n (factorial (- n 1))))))))
```

### On laziness vs. strictness
The above example only works for a lazy implementation of scheme. If
it is entered into a strict implementation then when `(Y f)` is
evaluated it will produce `f (Y f)` and since all arguments to a
function are evaluated before the function is applied we'll need to
evaluate `(Y f)` which will never end.

To get around this realize that `(Y f)` for our factorial function
produces a function of one argument (if you revisit our previous
examples it produces the `(lambda (n) ...)` function). So we can
rewrite `(Y f)` as `(lambda (x) ((Y f) x))` since for function `f` of
one argument, just `f` is equivalent to `(lambda (x) (f x))`. With
this addition, the strict version of the Y-combinator (which only
works for "recursive" functions of one argument) is:
```scheme
(lambda (f)
  ((lambda (g) (g g))
   (lambda (g) (f (lambda (x) ((g g) x))))))
```

So in a strict scheme our factorial function looks like:
```scheme
((lambda (f)
   ((lambda (g) (g g))
    (lambda (g) (f (lambda (x) ((g g) x))))))
 (lambda (factorial)
   (lambda (n)
     (cond
      ((= n 0) 1)
      (else (* n (factorial (- n 1))))))))
```
