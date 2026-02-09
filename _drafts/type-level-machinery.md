---
title: "Motivating the use of type level machinery"
layout: post
author: Xavier Góngora
locale: en_US
tags:
- functional programming
- haskell
---

Type level features in Haskell, such those enabled by as `GADTs`
and `DataKinds` language extensions, can be hard to reason about and much
harder to develop a working intuition of. In this post we go over some
examples that both motivate the need and illustrate the advantages of type
level programming meant to encode statically verified correctness properties:

- Creating type safe themes in `xmonad`, as developed by [Rebecca Skinner](https://www.youtube.com/watch?v=fQggLFtdTTo).
- Define safe communication protocols by means of [`typed-protocols`](https://input-output-hk.github.io/typed-protocols/)

Some other nice post about type level programming in haskell:

- [An introduction to typeclass metaprogramming](https://lexi-lambda.github.io/blog/2021/03/25/an-introduction-to-typeclass-metaprogramming/)
  by Alexis King.
- [Abusing Constraints for Automatic Documentation](https://reasonablypolymorphic.com/blog/abusing-constraints/)
  by Sandy Maguire.
