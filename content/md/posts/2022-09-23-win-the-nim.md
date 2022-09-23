{:title "Codewars - Winning a round of NIM the game"
 :layout :post
 :tags ["codewars" "clojure"]}
 
We are challenged to a game of [Nim](https://en.wikipedia.org/wiki/Nim), a
mathematical game about taking stuff from a pile. And we are expected to win, of
course!

### The problem

We have a number of stacks of... things. Matches, coins, pebbles, pearls...  
Two players take turns, each taking any positive number of things from a single pile.  
Whoever takes the last item is the winner.

### The solution

An algorithm to winning exists, and it requires the notion of a _Nim-sum_:

> The Nim-sum is the result of reducing the sizes of all stacks by bit-wise
> `xor`

So how to win?

- If the _Nim-sum_ at the beginning of your move is not zero, you are in a
  winning position. Especially, when there is only one stack left, you can take
  all of its items to win.
- If you are in a beginning position at the start of your turn, you can always
  choose a move to ensure that the _Nim-sum_ is zero after your move.
- If the _Nim-sum_ at the beginning of your move is zero, there is no way to
  take items from a single stack so it is still zero after your move. You are in
  a losing position until your opponent makes a mistake and leaves the board
  with a positive _Nim-sum_ after his move.

```clojure
(defn choose-move [stacks]
  (let [nim-sum (apply bit-xor stacks)
        winning-position? (pos? nim-sum)

        ai-options
        {:winning-move {:filter (fn [[_ straws]]
                                 (and (pos? straws) ;; don't take from empty stacks
                                      (< (bit-xor nim-sum straws) straws)))
                        :move   (fn [[idx straws]]
                                  [idx (- straws (bit-xor nim-sum straws))])}
         :lurking-move {:filter (comp pos? second) ;; any non-empty
                        :move   (fn [[idx straws]]
                                 [idx (inc (rand-int (dec straws)))])}}
        ai (if winning-position?
            ;; we can't lose now!
            (:winning-move ai-options)
            ;; oh blast... we need to wait patiently
            (:lurking-move ai-options))]

    (->> stacks
         (map-indexed (fn [idx straws] [idx straws]))
         (filter (:filter ai))
         first
         ((:move ai)))))
```

After submitting the solution, I saw that the game would always start us in a
winning configuration, so there really is no need to write the whole game logic
in the first place as we can always keep winning.

```clojure
(defn play-winning-nim [stacks]
  (let [nim-sum (apply bit-xor stacks)]
    (->> stacks
         (map-indexed (fn [idx straws] [idx straws]))
         (filter (fn [[_ straws]]
                     (and (pos? straws) (< (bit-xor nim-sum straws) straws))))
         first
         (fn [[idx straws]]
             [idx (- straws (bit-xor nim-sum straws))]))))
```
