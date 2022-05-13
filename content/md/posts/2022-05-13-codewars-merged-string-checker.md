{:title "Codewars: Merged String Checker - Clojure"
 :layout :post
 :tags ["clojure" "codewars"]}
 
Why nobody likes bananas from the Bahamas, and a rather pleasing recursive solution
to love them again.

### The problem

You are given three strings, a target string and two partial strings.  
Your task is to check if the target string can be merged from the partials. 
The only thing guaranteed is that the letters in the partial string will always 
be in the correct order.

While exploring the possible solutions you will find another restriction: all the letters
from the partial strings must be used.

### My solution

From reading the task description I decided to attack the kata with a recursive function
The strategy would be to pick one letter at a time from all three strings and either
fail fast or proceed with the rest of the strings, so I started like this:

```language-clojure
(defn is-merge ;; A predicate needs a question mark, this is Clojure!
  [[s1 & s']
   [a1 & a' :as a]
   [b1 & b' :as b]]
   (cond
     (nil? s1) true            ;; we built the whole string, success!
     (= s1 a1) (recur s' a' b) ;; partial a matched, so proceed with b and the rest of a
     (= s1 b1) (recur s' a b') ;; vice versa
     :else     false))         ;; no match - false
```

Here I immediately destructure the functions arguments, hoping that the inputs would always
be sequable. If you don't know, the `:as` keyword in destructuring parameters will give the
whole input parameter before destructuring a symbolic name, so you can both eat your cake
and keep it. I often wish I could do that in my TypeScript day job.

This will pass roughly half the test cases. *Wait. What about the other half?*

This is where I discovered the hidden rule that all the letters from the partials have to
be used. Easy to fix:

```language-clojure
(defn is-merge [] ;; A predicate needs a question mark, it's Clojure!
  [[s1 & s']
   [a1 & a' :as a]
   [b1 & b' :as b]]
   (cond
     (nil? s1) (every? nil? [a1 b1]) ;; only succeed if partials were consumed completely 
     (= s1 a1) (recur s' a' b)
     (= s1 b1) (recur s' a b')
     :else     false))
```

Ok, now only 7 of 74 fail. Yes, but why do they?

The first failing test is the infamous "Bananas from Bahamas" example:  
`(is (true? (is-merge "Bananas from Bahamas" "Bahas" "Bananas from am")))`

As you can see, both partials start with "Ba", but differ at the third letter. Also,
"Bananas" and "Bahamas" differ only by two letters.

It is actually not enough to greedily devour one partial or the other - we actually need
to check what would happen if we chose either to proceed.

To solve this we could use a trie or some backtracking algorithm, but exploring the
space of test cases I found the strings not to be overly long. So we should be fine
using recursion and losing tail call optimisation in some of the cases.

```language-clojure
(defn is-merge [] ;; A predicate needs a question mark, it's Clojure!
  [[s1 & s']
   [a1 & a' :as a]
   [b1 & b' :as b]]
   (cond
     (nil? s1)    (every? nil? [a1 b1])
     (= s1 a1 b1) (or (is-merge s' a' b)  ;; when we don't know which partial to use 
                      (is-merge s' a b')) ;; check both possibilities
     (= s1 a1)    (recur s' a' b)
     (= s1 b1)    (recur s' a b')
     :else        false))
```

This will consume partials if only one of them provides the next required letter and branches
when both partial do so.
