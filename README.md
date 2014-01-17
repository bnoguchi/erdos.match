# erdos.match

A simple clojure library for pattern matching.

Patterns will be simplified and compiled to clojure code.

## Usage

Copy the match.clj file to your src/erdos folder.
Include the following in your code:
```clojure
(use '[erdos.match :only (match)])
```

## Syntax

Usage of the `(match)` macro. The first argument is the value to be matched. The following arguments are patterns and bodies. The body for the first matching pattern will be executed.

The `_` symbol can be used to match any object. Symbols starting with `?` sign will be used as names for capturing objects. (They can also be used for type matching, see examples.) The first occurence of the symbol is used for binding and all other occurences are for equality checking. Thus, you can check for repeating parts in your pattern.

All other symbols will be handled as concrete objects for matching.

The `[]` notation is for matching vectors and `()` is for matching lists. They are not interchangeable. The `&` sign is used for matching the remaining part of the sequence.

Values in maps can be matched by keys. For example, pattern {:a ?a} matches {:a 1} object by binding 1 to a.

## Examples

### simple value matching

When matching for simple values, the pattern will be compiled for a `(case)` expression.
```clojure
(match a
   1       :one,
   2       :two,
   "three" :three,
   ?a      (str "value: " ?a))
;; a=1 => :one, a=2 => :two, a="three" => :three, etc..
```

### vectors

It is possible to match vectors and lists by length.
```clojure
(match a
   []      0,
   [_]     1,
   [_ _]   2,
   [_ _ _] 3,
   [& _]   :many)
;; when a=[] => 0, a=[5] => 1, a=[4 5] => 2, etc..
```

Or combining the previous two:
```clojure
(match a
   [:t _] :t,
   [_ :t] :t,
   [_ _]  :f)
;; when a=[:t :t] => :t, a=[:f :t] => :t, etc..
```

You can also refer to previously matched values.
```clojure
(match a
   [?a ?a] (str ?a "=" ?a)
   [?a ?b] (str ?a "/=" ?b))
;; when a=[1 1] => "1=1", a=[1 2]=>"1/=2", etc..
```

### maps
```clojure
(match {:a [1 2 3] :b 2 :c 3}
   {:a [?a1 ?a2 ?a3]} (+ ?a1 ?a2 ?a3)
   _                  :f)
;; => 6
```

### for analyzing clojure code
Checking for symbols and lists is possible. Please note the different syntax for lists and vectors.
```clojure
(match a
   (if ?cond ?then ?else) (str "if-expression")
   (when ?cond & ?then)   (str "when-expr")
   _                      :unexpected)
```

### Type matching

```clojure
(match "what is this?"
       ^Integer ?a :int
       ^String  ?a :string
       ^Long    ?a :long
       _ :unknown)
;; => :string
```
Cool eeh?

### Guard functions

Match even numbers.

```clojure
(match 24
   ^{:guard even?} ?e :even
   _                  :odd)
;; => :even
```

## License

Copyright © 2014 Janos Erdos
