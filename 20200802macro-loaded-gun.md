Arbitrarily deeply nesting fully partially applied functions with Clojure macros.

tags: functional programming, higher order function, partial application, beginner, clojure, javascript, macro, clojure.core/partial 

To provide a Rosetta stone, I included JavaScript equivalents to Clojure code snippets.

In this blog post we will look at the basics of Clojure macros and the difference between function literals and the function `clojure.core/partial`. Both topics concern the _read_ and _eval_ phases of executing Clojure code.

A loaded gun is a 'fully partially applied' function.
Clojure _almost_ provides a way to arbitrarily deeply nest loaded guns.
I wrote a macro that lets me arbitrarily deeply nest loaded guns.

Partial application is creating a function by providing some arguments to a function call. In the example below, `add-5` is a function obtained by partially applying arguments to a call to `+`. Although in javascript + is not a function but an operator, the idea stays the same. The duplicate global variables are there to showcase the different function literals.

```clojure
(def  add-5 (fn [a] (+ 5 a)))
(def  add-5 #(+ 5 %))

(add-5 4) ; => 9
```
```javascript
add5 = function(a) { return 5 + a }
add5 = a => 5 + a

add5(4) // => 9
```

Clojure even has a core function `clojure.core/partial` to perform partial application:

```clojure
(defn partial ; *
  "Takes a function f and fewer than the normal arguments to f, and
  returns a fn that takes a variable number of additional args. When
  called, the returned function calls f with args + additional args."
  ([f & args]
   #(apply f args)))

(def add-5 (partial + 5))

(add-5 4) ; => 9
```
```javascript
partial = (f, ...some_args) => (...more_args) => f.apply(null, some_args.concat(more_args))

add = (a, b) => a + b // **

add5 = partial(add, 5)

add5(4) // => 9
```
\* I shortened the actual implementation of `partial` for brevity.

** Because in JavaScript + is not a function but an operator, it can't be used as a function.

However, as we will see later in this blog post, calling `partial` significantly differs from the using the function literals `(fn ...)` and `#(...)`.

```clojure
(defmacro -# [& fn-call] `(fn [] (~@fn-call)))
```
