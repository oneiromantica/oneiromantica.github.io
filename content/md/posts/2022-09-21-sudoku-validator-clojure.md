{:title "Codewars: Validate sudoku solution"
 :layout :post
 :tags ["codewars" "clojure"]}

I started wondering where all the array- or sudoku puzzles had hidden, and lo
here's one of them.

### The problem

We should check if given 2D array (passed as vector of vectors) is a valid
solution for a [sudoku](https://en.wikipedia.org/wiki/Sudoku) puzzle.

This means that every row, every column and every of the nine 3x3 sections of
the grid contains all numbers from 1 to 9 exactly once.

### My solution

This one is a bit less succinct than it could be because I decided to structure
the solution to some helper functions.  
I once more started with translating the rules to code:

```clojure
(defn valid-block? [block]
  (= (set (range 1 10)) (set block)))
  
(defn valid-solution [board]
  (and (every? valid-block? board)
       (every? valid-block? (columns board))
       (every? valid-block? (squres board))))
```

Getting the columns is not too complicated, either:

```clojure
(defn columns [m]
  (apply map vector m))
```

Here we use `apply` to pass each row of the board as argument to `map vector`
and take advantage of the multi-arity nature of `map`; that is when map is given
a number of sequences, it uses the n-th elements of every sequence as arguments
in the n-th step. In this context, we first apply `vector` to build a sequence
of the first elements of all rows, then a vector of the second, and so on.  
Incidentally, that is how to idiomatically transpose a matrix in Clojure.

Now what's left is getting the individual 3x3 squares:

```clojure
(defn squares [m]
   (->> (map (partial partition 3) m)
        columns
        flatten
        (partition 9))
```

This roughly translates to:

- split all rows to three-tuples 
  ```
  [[[11 12 13] [14 15 16] [17 18 18]]
   [[21 22 23] [24 25 25] ..]
   ..]```
- transpose the matrix of three tuples
  ```
  [[[11 12 13] [21 23 24] [31 32 34]]
   [[14 15 16] [24 ...]]
   ..]
  ```
- then merge the tuples of all rows to a sequence of sequences with nine numbers
  each

And that's all there is to validate a given sudoku solution.
