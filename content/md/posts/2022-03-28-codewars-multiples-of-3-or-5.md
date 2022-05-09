{:title "Codewars: Multiples of 3 or 5 - Clojure"
 :layout :post
 :tags ["codewars" "clojure"]}
 
An easy problem, and like most data stream processing problems a
perfect fit for functional programming.

There are multiple possible solutions on how to implement the filtering. I
chose one that reads closely to the problem instruction.  
The question whether to `reduce` or to `apply` a variadic function is for
wiser people to discuss, both produce the same result.

### The problem

Given an exclusive upper limit, find all number between zero and that limit
that are divisible by either three or five and sum them up.

### My solution

```language-clojure
(defn solution [number]
  (->> (range number)
       (filter #(some zero? [(mod % 3) (mod % 5)]))
       (apply +)))
```

This one is straightforward. The collection-threading-macro `->>` makes the
code easy on the eye, and the code itself reads just like a description of
the problem.  
As we all once learned, a natural number `n` is divisible by `k` if the
remainder of `n / k` is equal to `0`, and the modulus operation calculates
that remainder.
