{:title "Codewars: RGB to hex conversion - Clojure"
 :layout :post
 :tags ["codewars" "clojure"]}

A quickie to do while waiting for the company's daily meeting to start.

### The Problem

You are given three decimal integers `r`, `g`, and `b` and are to create an upper case
hexadeximal color string from them.

### My solution

There is a number of ways for converting decimals to hex. If you want to practise your
maths, you can do the base transformation by hand, you can use `Integer/toHexString`,
but if you're familiar with them (or can't be bothered), the most concise way is
to just use string formatting: `(format "%02X" n)`, where `%02` means "ensure two digits",
and the format `X` is for "uppercase hexadeximal", in contrast to `x`, its lowercase
sibling.

Bear in mind that the input may be outside the valid range `[0, 255]`.

```language-clojure
(defn clamp [lower upper n] (max lower (min upper n)))

(defn rgb [r g b]
  (apply format "%02X%02X%02X" (map (partial clamp 0 255) [r g b])))
```

I'm the first to admit that `clamp` is too general for that specific problem, as there
is only one set of upper and lower limits. Making it take parametric limits came out of habit.

```language-clojure
(defn clamp-hex [n] (max 0 (min 255 n)))

(defn rgb [r g b]
  (apply format "%02X%02X%02X" (map clamp-hex [r g b])))
```

Is a little less over-engineered.
