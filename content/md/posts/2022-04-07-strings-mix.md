{:title "Codewars: Strings Mix - Clojure"
 :layout :post
 :tags ["codewars" "clojure"]}

A very strange sorting task, indeed. Once again, it's very suited for
functional programming, and the capabilities of Clojure make short work
for it.

### The Problem

Given two strings, you have to do the following:

- Compare their similarity by counting the maximum number each letter appears in either string
- Return a string describing the similarity

Rules for similarity:

- Only consider lower case letters `a-z`
- Only consider letters which appear more than once on either string

Rules to build the report string

- Sort fragments for each letter descending by the maximum
- On equal length, sort the fragments ascending (string-comparison)
- Output format:
  - `1`, `2`, or `=` to describe if the maximum was found in string 1, string 2 or if it was a tie
  - A colon
  - The letter, repeated as often as its maximum
  
### My solution

I had to read the instructions multiple times, but once I got the gist, implementation was a lot
less daunting.

The basic idea is to present the intermediate values as tuples `[original string, count, char]`, so
we can use destructuring and threading.

```language-clojure
(defn freqs [s]
  (frequencies (re-seq #"[a-z]" s))) ;; count appearances of all valid chars

(defn mix [s1 s2]
  (let [f1 (freqs s1)
        f2 (freqs s2)
        ks (into (set (keys f1)) (set (keys f2)))]    ;; get all keys in any word
    (->> ks
         (map #(let [n1 (get f1 % 0)
                     n2 (get f2 % 0)]
                 (cond (> n1 n2) [1 n1 %]
                       (> n2 n1) [2 n2 %]
                       :else     ["=" n1 %])))        ;; create our tuples
         (filter (fn [[_ n]] (> n 1)))                ;; only keep valid ones
         (sort-by (fn [[s n v]] [(- n) (str s v)]))   ;; sort by length and string value
         (map (fn [[sgn n v]]                         ;; build result fragments
                (str sgn ":" (clojure.string/join (repeat n v)))))
         (clojure.string/join "/"))))
```
