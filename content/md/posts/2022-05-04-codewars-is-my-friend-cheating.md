{:title "Codewars: Is my friend cheating? - Clojure"
 :layout :post
 :tags ["codewars" "clojure"]}
 
One of those that makes you think "this is far too easy to be true". And you're right!

### The problem

Given an upper limit `n`, you are to give a list of tuples of numbers `(a, b)` for which `a * b` equals the sum of `[1, n]` minus the sum of `a` and `b` (or the empty list if there are none).

### My solution

At first glance this is a simple task, so I write it the most naive way possible, despite knowing that the quadratic execution time will get the tests to time out later on. Clojure's `for` construct makes the solution look quite elegant.

```language-clojure
(defn solve [n]
  (let [ns (range 1 (inc n))
        s (apply + ns)]
    (for [a ns, b ns
          :when (= (* a b) (- s a b))]
      [a b])))
```

This is basically a literal translation of the task - get all tuples where `a * b = (sum 1, n) - (a + b)`, and as it iterates all values `a, b` to build the tuples, it takes `O(n * n)` to evaluate.

As predicted, the full test suite contains `n = 1000003`, which tests `1e12` possible tuples and times out (being curious I tried to time it on my local REPL, but killed the process after some minutes).

So we need to get faster.

### Improving performance

#### "Typical" CS solutions

We could try optimising the loop. When we start the range for `b` at `a + 1`, we can get the number of tests down by half. Then we need to add all tuples `[(a,b), (b,a)]` to the result set. But now we need to flatten the results (pairs of tuples) and order them by increasing `a`, then increasing `b`. This is a lot of overhead, and additional we will only gain `n * n / 2 = O(n * n)` execution time, so there is barely anything to win.

We could go around, filtering the set of possible values for `b` by removing known hits/misses from running through `a`, but this will not get better than the first idea, plus adding a lot of work to constantly modify lists. And the code will get ugly.

So we need to do something scary: we need to use maths.

#### Thinking less like a CS person

What we really want is to solve the equation `a * b = s - a - b` for a given `s` and `a,b ∈ [1..n]`
We get out pen and paper and start:

- `a * b     = s - a - b` with `a,b ∈ [1..n]`
- `a * b + b = s - a`
- `b (a + 1) = s - a`
- `b         = (s - a) / (a + 1)` for `b ∈ [1..n]`

Now, that's something we can work with! We can derive a candidate value for `b` from `a`, so we do not need to run any loops for `b` at all, improving our execution time to `O(n)`. In the case of the `n = 1000003`, that's an improvement by six orders of magnitude. This got to be at least a little faster ;)

Now, let's change the Clojure `for` to reflect our insights:

```language-clojure
(defn solve-linear [n]
  (let [ns (range 1 (inc n))
        s (apply + ns)]
    (for [a ns
          :let [b (/ (- s a) (inc a))]
          :when (and (integer? b) (<= 1 b n) (not= a b))]
      [a b])))
```

Which requires only 5ms for `n = 1000003` on my machine.
