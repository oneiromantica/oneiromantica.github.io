{:title "Codewars: Pyramid Slide Down - Clojure"
 :layout :post
 :tags ["codewars" "clojure"]}

A nice little problem, reminding me of my studies in bioinformatics.

### The Problem

There is a pyramid of numbers, the goal is to descend from the top by
"sliding" over adjacent neighbours and finish with the greatest sum.  
The description explicitly announces that huge inputs are used to 
discourage any brute force attempts (test runs time out after 12s, btw). 

### My solution

I elaborated a little on the basics of sequence alignment scoring ideas
formalised by Smith & Waterman or Needleman & Wunsch. The simplicity
of the problem allows leaving out the more complex parts of their 
algorithms.

The basic idea is to walk through the pyramid and always find the maximum
possible distance for every single position by adding the current position's
value to the maximum value of both adjacent position in the row above us.

```
input   | row 1    | row 2
        |          |
   3    |     3    |             3
  7 4   |  7+3 4+3 |     10             7
 2 4 6  |   ...    | 10+2  max(10+4,7+4) 7+6
8 5 9 3 |          |                         
```

Thus, we can find the maximum in a single pass of `O(h * h/2)` with the
pyramid's height `h`.

```language-clojure
(defn get-upper-neighbours [dists column]
  [(get dists (dec column) 0)
   (get dists column 0)])

(defn longest-slide-down [pyramid]
  (let [height (count pyramid)
        dists (reduce (fn [ds row]
                        (mapv #(+ (get-in pyramid [row %] 0))
                                  (apply max (get-upper-neighbours ds %))
                              (range (inc row))))
                      []
                      (range (inc height)))]
    (apply max dists)))
```

As we're only interested in the largest number inside the last row, we do
not need to store the whole distance pyramid in memory, it suffices to 
only return the last row.

A more concise approach would be to invert the pyramid, partition each row
into tuples and add the bigger neighbour to the next row's value. 
