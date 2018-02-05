## Motivation for [fct](http://github.com/achatz64/fct)

Development of fct was done in essentailly 3 steps guided by the following questions:
1. How can we express and argue with a statement like "x is y" in clojure, which depends on  some variables x,y?
2. How can functions depend on variables?
3. How can we test efficiently?

> In this text, I will focus on 1. and a little on 2.

#### For Question 1.
Let the statement `f` depend on `x,y`, and the statement `g` depend on `y,z`. Maybe the first approach in clojure would be to consider
```clj
(def f (fn [x y] ...))
```
and similarly for `g`. The problem is that
`
(and f g)
`
doesn't make any sense. Our definition of `f` wasn't great anyway, because the arguments are positional, and associative arguments are certainly better suited for our problem. After defining
```clj
(def f (fn [{:keys [x y]}] ...))
(def g (fn [{:keys [y z]}] ...))
```
the expression `(and f g)` still doesn't make any sense, but with
```clj
(defn lift-and [f g] (fn [l] (and (f l) (g l))))
```
the expression `(lift-and f g)` can be evaluated at `{:x value-x :y value-y :z value-z}` to produce the desired result. Alright, let associative functions be the statements we ask for in Question 1, and let's work with them by lifting clojure functions. For this we need the lifting construction:
```clj
(defn lift [clj-fn]
  (fn [& args]
    (fn [l] (apply clj-fn (map (fn [f] (f l))
                                args)))))
```
so that `(lift clj-fn)` will do what we expect.
> Note: `(lift and)` won't work, because `and` is a macro

At this point we have associative functions as objects, lifted clojure functions in order to combine them to new associative functions, and we can substitute values for the variables by calling the objects on `{:x value-x ...}`.

The variables construction is given by:
```clj
(defn var [key-word] (fn [l] (key-word l)))
```
This provides an answer to Question 1.  

#### For Question 2.
One problem with the formalism explained so far is that it is asymmetric. The lifted clojure functions are totally different from the associative functions they combine. For example, evaluating a lifted clojure function on a lifted clojure function is just non-sense.

For another example, suppose the variable `(var :x)` will become a certain data structure one day, but we are not quite sure which specifications the data structure will have. However, we know that they will all admit an identity, and we need the `get-id` function to go on. It's natural to just create a new variable `(var :get-id)` which we will supply later (after we know the specs of `(var :x)`). Unfortunately,
```clj
((var :get-id) (var :x))
```
doesn't make sense. The goal of the fct framework is to make these kind of constructions possible.

The quasi-philosophy behind fct is to create an alternative for a work flow that starts with fixing the data structures and then goes on with building the functions for them. There is nothing wrong with this, but it's fun to try an alternative.

One immediate problem that arises: when there are variables all over the code, the compiler can't check the resulting gibberish. Therefore testing must be part of the framework.    
