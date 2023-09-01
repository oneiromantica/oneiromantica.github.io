{:title "Codewars: Tribonacci - Clojure"
 :layout :post
 :tags ["codewars" "clojure"]}
 
Some finicky maths, expanding on the Fibonacci problem. I usually dislike maths
problems, as they do not really test one's computer skills, but this one was
somewhat intriguing.

### The problem

Instead of the common Fibonacci sequence, creating a sequence of numbers by
adding its two predecessors in the sequence, usually starting from `[0, 1]` or
`[1, 1]`, we are asked to generate the sequence by adding its three
predecessors.

### My solution

One could go a relative straightforward way by destructuring and starting with
the first values given. On the other hand, the problem description told that
both Fibonacci and "tribonacci" were instances of an n-bonacci problem, so why
not follow the course of "solve the class of problems and specialise for your
case" pattern we're advised to use.

```clojure
(defn n-bonacci [s n]
  (let [step
        (fn [summands]
          (let [sum (apply + summands)]
            (prn summands sum)
            (conj (vec (rest summands)) sum)))]
    (->> (iterate step (step s))
       (map last)
       (concat s)
       (take n))))
```

Here we define a step function which takes a vector of arbitrary length and
creates the input vector for the next step by shifting it to the left and
appending the current step's value (the sum of the input) to the result.

```clojure
[[0 0 1], [0 1 1], [1 1 2], ...]
```

To get the desired n-bonacci number for each position, we map `last` over the
result, as a result of how our step function works.

As we only calculate values following the initial n-tuple, we need to
concatenate them to the seeding n-tuple used to create the sequence to get the
complete sequence.

### Results

```clojure
;; classical fibonacci
(n-bonacci [1 1] 10)
; [1 1 2 3 5 8 13 21 34 55]

;; one of Codewars' test cases
(n-bonacci [0 0 1] 10)
; [0 0 1 1 2 4 7 13 24 44]
```
