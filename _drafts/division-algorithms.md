---
title: "Division Algorithms in Haskell"
layout: post
author: Xavier Góngora
locale: en_US
tags:
- haskell
- euclidean domain
- euclidean division
- integer division
---

Haskell has two pairs of functions related to
[euclidean division](https://en.wikipedia.org/wiki/Euclidean_division): `div` and `mod` | `quot` and `rem`.

At the [source documentation](https://hackage.haskell.org/package/base-4.21.0.0/docs/GHC-Real.html)
we find that `ðiv` is specified as "Integer division truncated toward negative infinity"
and `quot` as "Integer division truncated toward zero". But what does this _mean_?

The requirement of being an euclidean division means that the following properties most hold:

```haskell
(x `div` y)*y + (x `mod` y) == x

(x `quot` y)*y + (x `rem` y) == x
```

In mathematical notation, this is:

$$ x = qy + r | 0 <= r < |y| $$

Why are there different variants of the euclidean division?

What are they useful for?

What other varianst exist?

The division theorem states that both quotient and remainder most be unique, why is this not so?

```haskell
-- | A function to look for value differences in the applicacion of divMod and quotRem.
quotRemDivModDiff :: (Integral a) => [(a, a)] -> [Maybe ((a, a), (a, a))]
quotRemDivModDiff list = zipWith maybeDifferent (map (uncurry quotRem) list) (map (uncurry divMod) list)
  where
    maybeDifferent (d, m) (q, r) = if (d, m) == (q, r) then Nothing else Just ((d, m), (q, r))

-- | An implementation of 'quotRem', which is a primitive in the standard library.
quotRem' :: (Integral a) => a -> a -> (a, a)
quotRem' x y = case (signum x, signum y) of
  (-1, -1) -> second negate $ quotRemIter (-x) (-y) 0
  (1, -1) -> first negate $ quotRemIter x (-y) 0
  (-1, 1) -> both negate $ quotRemIter (-x) y 0
  (1, 1) -> quotRemIter x y 0
  (_, 0) -> error "quotRem': divide by zero"
  (0, _) -> (0, 0)

-- | A variant of 'quotRem' implemented using 'divMod'
quotRem'' :: (Integral a) => a -> a -> (a, a)
quotRem'' x y = case (signum x, signum y) of
  (1, -1) -> first negate $ divMod x (-y)
  (-1, 1) -> both negate $ divMod (-x) y
  (_, _) -> divMod x y

-- | Haskell implementation of SMT div and mod
divMod' :: (Integral a) => a -> a -> (a, a)
divMod' x y = case (signum x, signum y) of
  (_, 0) -> error "quotRem': divide by zero"
  (0, _) -> (0, 0)
  (1, 1) -> quotRemIter x y 0
  (-1, -1) -> second negate $ quotRemIter (-x) (-y) 0
  (1, -1) -> first negate $ quotRemIter x (-y) 0
  (-1, 1) -> both negate $ quotRemIter (-x) y 0

first :: (a -> a') -> (a, b) -> (a', b)
first f (x, y) = (f x, y)

second :: (b -> b') -> (a, b) -> (a, b')
second f (x, y) = (x, f y)

both :: (a -> b) -> (a, a) -> (b, b)
both f (x, y) = (f x, f y)

{-@ quotRemIter :: (Integral a) => {x:a | a >= 0}
                                -> {y:a | y > 0}
                                -> {q:a | q >= 0}
                                -> {z:(a,a) | fst z = div x y && snd = mod x y} 0}@-}

-- | A non-total and straight-forward implementation of the division algorithm on
-- positive values. This function behaves as 'divMod' and 'quotRem' if the first
-- and third arguments are natural and the second positive.
quotRemIter :: (Integral a) => a -> a -> a -> (a, a)
quotRemIter a b q =
  if a - b * q < b
    then (q, a - b * q)
    else quotRemIter a b (q + 1)

divModSMT :: (Integral a) => a -> a -> (a, a)
divModSMT = divModIter 0

divModIter q a b =
  case signum (a * b) of
    1 ->
      if abs (a - b * q) < abs b && (b * q <= a)
        then (q, a - b * q)
        else divModIter (q + 1) a b
    -1 ->
      if abs (a - b * q) < abs b && (b * q <= a)
        then (q, a - b * q)
        else divModIter (q - 1) a b
    0 -> if b == 0 then error "divide by zero" else (0, 0)

divModIter' q a b
  | b == 0 = error "divide by zero"
  | a == 0 = (0, 0)
  | otherwise =
      if abs a - abs (b * q) < abs b && (b * q < a)
        then (q, a - b * q)
        else divModIter' (q + 1) a b
```
