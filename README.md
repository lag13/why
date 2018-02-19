# why
Personal notes and code samples for the Y-combinator. I do not try to
derive the Y-combinator or give much analysis of it. These notes are
more "practical" and show:
- What the Y-combinator is.
- An example of the Y-combinator in action to get a sense for how/why
  it works.

## Links
- http://mvanier.livejournal.com/2897.html - Derives the Y-combinator
  and implements it in scheme. A fantastic read.
- https://medium.com/@ayanonagon/the-y-combinator-no-not-that-one-7268d8d9c46 -
  Short and sweet look at the Y-combinator with an introduction to
  lambda calculus.
- https://en.wikipedia.org/wiki/Fixed-point_combinator
- https://mitpress.mit.edu/books/little-schemer

## Technology Related Concerns
Most examples are in scheme and only work if the particular
implementation of scheme is "lazy". I used racket as my lazy scheme
interpreter so download that if you want to follow along:
- http://racket-lang.org/
- http://docs.racket-lang.org/lazy/index.html

## Here we go
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

Creating this recursive function is possible because of our ability to
`define` variables. It turns out though that even if our language has
no `define` (or looping constructs) we can still create a `factorial`
function using only anonymous functions. Let that sentence sink in a
bit, because I think that is freakin' crazy. **Without** `define` we
can still, effectively, create recursive functions by using the
Y-combinator. That is cool to me for 2 reasons:
1. It means your core language can be more minimal in terms of
   features but still be just as powerful. I love this minimalism. I
   love the idea of being able to do the same amount of work with less
   tools.
2. As you program you start to think of certain constructs in a
   language as being fundamental-truths/axioms/
   something-that-is-necessary-to-get-anything-done. For example, when
   first learning to program you might think of looping constructs
   (for, while, etc...) as a necessary part of any language. Later
   though you learn that you can just use recursion and this result
   changes your perception on what is strictly necessary in a
   language. I love it when a result changes my perception of the
   world and I especially love it when a result demonstrates that
   something I condidered to be an "axiom" is not.

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

It is a function that takes a function `f` and applies `f` to the
result of `(Y f)`. It looks a little non-sensical but when given the
proper function `f` it creates a recursive version of `f`. So if we
are able to create some function "`almost-factorial`" then `(Y
almost-factorial)` **is** our `factorial` function.

Note that this implementation of `Y` only works for lazy languages
because when `(Y f)` is evaluated it will produce `(f (Y f))` and the
`(Y f)` argument to `f` will **not** be evaluated until its value is
needed. Later we will define a `Y` which also works for strict
languages but since the lazy version is more concise I'm using it for
the examples.

### Using the above Y-combinator
Here is the function `almost-factorial` which when passed into `Y`
will create the function `factorial`:

```scheme
(lambda (factorial)
  (lambda (n)
    (cond
     ((= n 0) 1)
     (else (* n (factorial (- n 1)))))))
```

Assuming we have `Y` at our disposal this is the `factorial` function:

```scheme
(Y
 (lambda (factorial)
   (lambda (n)
     (cond
      ((= n 0) 1)
      (else (* n (factorial (- n 1))))))))
```

Note there is **no recursion** in our `almost-factorial` function
because there is no `define`. `Y` **is** recursive at this point but I
did that to aid in understanding what `Y` does instead of getting
bogged down with its non-recursive implementation. Once we go through
an example and convince ourselves that we can achieve recursion if a
construct like `Y` exists then we can define `Y` in terms of just
`lambda`'s and be done!

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

Remember that `(Y f) == (f (Y f))` and in this example `f` is `(lambda
(factorial) ...)` so first we get:

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

For conciseness let's label the second `(lambda (factorial) ...)` `f`.
Rewriting the previous example we get:

```scheme
((lambda (factorial)
   (lambda (n)
     (cond
      ((= n 0) 1)
      (else (* n (factorial (- n 1)))))))
 (Y f))
```

We invoke `(lambda (factorial) ...)` with `(Y f)` as the argument (and
keep in mind that `f` stands for `(lambda (factorial) ...)`) and get:

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

Adding back the `(* 3` the full evaluation so far is:

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

At this point I hope you see the pattern and are convinced that what
we have here **is** the factorial function. More generally I hope
you're convinced that if we have this function `Y` we can create
recursive functions such as `factorial`.

### Defining Y without recursion
The above example is really cool but we used a `Y` that was defined
recursively which feels a bit like cheating. So here in all its glory
is the definition of `Y` without explicit recursion:

```scheme
(lambda (f)
  ((lambda (g) (g g))
   (lambda (g) (f (g g)))))
```

The above definition is nice because it is concise and has no
repetition but we will be using this equivalent definition (acheived
by invoking the `(lambda (g) (g g))`) since it requires a little less
work to see the parallels with the high level understanding of `Y`:

```scheme
(lambda (f)
  ((lambda (g) (f (g g))) (lambda (g) (f (g g)))))
```

Let's pass in some arbitrary function called `fn` to convince
ourselves that this is indeed equivalent to our recursive
implementation of `Y`:

```scheme
;; Remember that this is like doing (Y fn) which should yeild (fn (Y fn))
((lambda (f)
   ((lambda (g) (f (g g))) (lambda (g) (f (g g)))))
 fn)
```

Substituting `fn` for all the `f`'s:

```scheme
((lambda (g) (fn (g g))) (lambda (g) (fn (g g))))
```

We evaluate that lambda (side note: this function is passing itself as
an argument to itself!!! crazy):

```scheme
(fn ((lambda (g) (fn (g g))) (lambda (g) (fn (g g)))))
```

Now, notice that:

```scheme
((lambda (g) (fn (g g))) (lambda (g) (fn (g g))))
```

Is equivalent to:

```scheme
;; We've got (Y fn) again!!
((lambda (f)
   ((lambda (g) (f (g g))) (lambda (g) (f (g g)))))
 fn)
```

Adding back the `(fn` we see that this indeed another implementation
of `Y` with **no recursion**:

```scheme
;; We've achieved (fn (Y fn)) !!
(fn ((lambda (f)
       ((lambda (g) (f (g g))) (lambda (g) (f (g g)))))
     fn))
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
The above examples only works for a lazy implementation of scheme. If
it is entered into a strict implementation then when `(Y f)` is
evaluated it will produce `f (Y f)` and, since all arguments to a
function are evaluated before the function is applied, we'll need to
evaluate `(Y f)` which will never end.

To get around this, realize that `(Y f)` for our factorial function
produced this function of one argument:

```scheme
(lambda (n)
  (cond
   ((= n 0) 1)
   (else (* n ((Y f) (- n 1))))))
```

So we can rewrite `(Y f)` as `(lambda (x) ((Y f) x))` since for any
function `f` of one argument, `f` is equivalent to `(lambda (x) (f
x))`. Delaying the evaluation of `(Y f)` by hiding it behind this
lambda makes `Y` work for strict lanugages as well.

With this addition, the strict version of the Y-combinator (which
unfortunately only works for "recursive" functions of one argument)
is:

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
