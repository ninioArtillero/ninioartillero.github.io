---
title: "Yet another take on lazy evaluation"
layout: post
author: Xavier Góngora
locale: en_US
tags:
- functional programming
- lazy evaluation
- haskell
---

[Why functional programming matters?]() sets a canon in the haskell and the
broader functional programming community on the benefits of funtional programming.
The takeaway is _composability_. A reified example of this in action is the
power of combinator libraries. The article also points to the fruitful
combination of functional programming with lazy evaluation.

Apfelmus also has a nice [post](), elaborating in how to think about lazy
evalutaion. I”ve seen also many discussion around it, following different threads
related to the issue:

- Performance
- Execution model
- Debugging
- Composability

My take here is for composability. It was when dealing with optimizing a function
that reused logic from elsewhere in the codebase whose role was mapping over a
record field onto one of its type parameters. This kind of function definition
implias an _ad hoc_ approach to each of the field. In particular, I needed to
transform and manipulate just one of such fields, but used the transformation of
the whole record to do so. Having though about lazy evaluation, I could figure
that in reality this optimization already took place thans to lazy evaluation!
As only one field was requested in the code, the transformation would only take
place for the values I was interested in. In actuallity, much of the fields where
_strict_, meaning that this was not really the case. But got me thinking about
the usefulness of non-strict variants of a data type.
