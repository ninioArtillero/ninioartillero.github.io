---
title: "A Liquid Haskell Kick-Start"
layout: post
author: Xavier Góngora
tags:
- programming languages
- type systems
- formal methods
- software verification
- tutorial
- haskell
---
In this tutorial post, I introduce Liquid Haskell (LH), present a step-by-step
installation procedure, and work through a simple example of its use.
As a bonus, we will see how to use a local LH build in our projects.

This is basically a rehash of some setup notes I made while preparing a proposal for the
[Google Summer of Code 2025](https://summerofcode.withgoogle.com/), based on the
[project idea](https://github.com/haskell-org/summer-of-haskell/blob/3c9efd21fc0022f7b9c21ac2001ca1049d888dc9/content/ideas/lh-aliases.md)
proposed by Facundo Domínguez.
More detailed information can be found at the LH
[documentation site](https://ucsd-progsys.github.io/liquidhaskell/) and
[source repository](https://github.com/ucsd-progsys/liquidhaskell).

LH is under active development, with a focus on quality-of-life features that make
it a breeze to use (or maybe not quite yet, but its getting there). I believe that
using _refinement types_ to specify programs unlocks expressive possibilities,
enabling stronger guarantees and clearer intent, that are worth exploring.

## About Liquid Haskell

LH is a tool that allows programmers to enforce contracts on their functions
inputs (_pre-conditions_) and outputs (_post-conditions_).
This is done through special comment annotations that extend a function type
signature with _refinement types_: types that include constraints defined by
logical predicates.

A typical example is that of a "successor" function, which we can specify to
_take only_ positive integers and _guarantee to produce only_ positive integers.

```haskell
{-@ succ :: {v : Int | v > 0} -> {v : Int | v > 0 } @-}
succ :: Int -> Int
succ n = n + 1
```

We can be more succinct by declaring a refinement type alias.

```haskell
{-@ type Pos = {v : Int | v > 0} @-}

{-@ succ :: Pos -> Pos @-}
succ :: Int -> Int
succ n = n + 1
```

Other arithmetic properties like "multiplication by a positive
integer preserves order" can be specified as well.[^real-world]

[^real-world]: Liquid Haskell has been used to specify and verify complex properties in significant portions of major Haskell libraries. See _LiquidHaskell: Experience with Refinement Types in the Real World_ @ <https://dl.acm.org/doi/10.1145/2633357.2633366>.

```haskell
{-@ type OrderedPair = { pair : (Int,Int) | fst pair < snd pair } @-}

{-@ escalarMultiplication :: Pos -> OrderedPair -> OrderedPair @-}
escalarMultiplication :: Int -> (Int,Int) -> (Int,Int)
escalarMultiplication n (x,y) = (n * x, n * y)
```

Note that we can use regular Haskell functions in the predicates, here `fst` and
`snd` to access the components of the pair. These are available, along with other
functions from the Haskell standard library (the "prelude"), because LH comes bundled
with their specifications (defined using _assumptions_).
In general, Haskell functions need to be lifted into the logic using `reflect`
and other constructs (which I don't cover here) to be used in specifications.
LH verifies that given an ordered pair, we get back an ordered pair, so that our
specification is _correct_. But also that this function is only called with
ordered pairs, so that the specification is _enforced_ at every call site.
To accomplish this LH renders the specifications into a collection of _constraints_
(think of a system of equations) that are passed to an external SMT solver that
verifies the specification.

I think of this as an alternative (or perhaps, complimentary) approach to
property based testing, in which properties are logically proven instead of being
checked against (cleverly) random generated input. In practice, what this means
is that we can express properties of our code and document its behaviour _in place_.

## Installation

First, we'll need the Haskell toolchain: `cabal` to manage the project
and the `ghc` Haskell compiler. The (current) recommended way to install and
manage both is  through [GHCup](https://www.haskell.org/ghcup/).[^nix]
After installing it with the default options, install a `ghc` version that corresponds
to a LH release (check the [docs](https://ucsd-progsys.github.io/liquidhaskell/install/)).
In this tutorial we'll use  `ghc-9.10.1` and `liquidhaskell-0.9.10.1.2`.[^ghc-policy]

[^nix]: Other common alternatives are using the [Stack build tool](https://www.haskellstack.org/) or the [Nix package manager](https://nixos.org/). Nix is a comprehensive tool (and language!) for reproducible package deployment, which can also be use to create declarative development environments.

```sh
ghcup install ghc 9.10.1
```

The following command bootstraps the creation of our project.
It creates a new cabal project for our tutorial, with the corresponding `base`
and `liquidhaskell` versions as dependencies,
and configures it to use the corresponding version of `ghc`.

[^ghc-policy]: As mentioned in the [LH source repository README](https://github.com/ucsd-progsys/liquidhaskell/), LH is developed against a specific version of `ghc` given its tight dependence on the `ghc` library which tends to break existing code without notice (in particular, because a distinction does not yet exists between public and internal API's). At the moment of writing, previous versions of `liquidhaskell` are not maintained, so that new features and bug fixes reach only the next major and minor releases monotonically.

```sh
mkdir lh-tutorial && cd lh-tutorial && \
cabal init --lib --dependency="base==4.20.0.0,liquidhaskell==0.9.10.1.2"  && \
cabal configure --with-compiler ghc-9.10.1
```

LH is implemented as a GHC plugin, which modifies the compiler pipeline to deliver
the extracted constrains to an SMT solver for verification.
For it to work, you must have any of the following installed in your system and
accessible from your `$PATH` environment variable:
[Z3](https://github.com/Z3Prover/z3), [CVC4](https://cvc4.github.io/) or [MathSat](https://mathsat.fbk.eu/)
The LH docs recommend Z3, which should be available from your Linux distribution package
manager. On MacOs, it can be installed using [Homebrew](https://brew.sh/): `brew install z3`.

To use LH, we need to indicate GHC to use the plugin. We can do so by modifying
the library stanza of the `lh-tutorial.cabal`.

```cabal
library
    import:           warnings
    exposed-modules:  MyLib
    -- other-modules:
    -- other-extensions:
    build-depends:
        base ==4.20.0.0,
        liquidhaskell ==0.9.10.1.2
    hs-source-dirs:   src
    default-language: Haskell2010
    ghc-options: -fplugin=LiquidHaskell -- ADD THIS LINE!
```

This enables LH verification across all modules in the project.
Finally, build the project with `cabal build`. If the build succeeds, now LH has
been properly installed within the project. For the time being, you should be
notified that no constraints where checked.

## Dummy Library

Modify the contents of `src/MyLib.hs` to include the examples from before.

```haskell
module MyLib where

{-@ type Pos = {v : Int | v > 0} @-}

{-@ succ :: Pos -> Pos @-}
succ :: Int -> Int
succ n = n + 1

{-@ type OrderedPair = { pair : (Int,Int) | fst pair <= snd pair } @-}

{-@ escalarMultiplication :: Pos -> OrderedPair -> OrderedPair @-}
escalarMultiplication :: Int -> (Int,Int) -> (Int,Int)
escalarMultiplication n (x,y) = (n * x, n * y)
```

If you `cabal build` now, you'll see that some constraints have actually been
checked. Now you're ready to work on your own project, using these
steps as a template.

## Using a local build of Liquid Haskell

Since I plan to work on the LH codebase and test my changes across various projects,
I need a way to use my local build. We now examine a couple of approaches, broadly
described
[here](https://stackoverflow.com/questions/69773999/how-do-i-get-cabal-to-use-a-local-version-of-a-package-as-a-dependency-for-a-hac),
to accomplish this.

We start by cloning LH source with the `--recurse-submodules` flag so that
the `liquid-fixpoint` package is also cloned (needed for the build).

```sh
git clone --recurse-submodules https://github.com/ucsd-progsys/liquidhaskell.git
```

The upstream source is developed against the `ghc` version corresponding to its
latest realease, so make sure to install it (at the time of writing its `9.12.2`).
The build instructions can be found at the source repository
[README](https://github.com/ucsd-progsys/liquidhaskell/blob/192b8766a521c6bef8be2b61c6fda3a1b53783fb/README.md),
suggest using the following command:

```sh
cabal build liquidhaskell
```

This _should_ build, but cabal is known to have its quirks. If it does, proceed
to the next section. If it doesn't, carefully check the installation
documentation at the source repository and the documentation site.
It is possible that your current cabal library version does not
support the needed `ghc` version (check the error message), so you might need to
change (_set_) it. Installation and _set_ of different versions of Haskell tools
can be done within `ghcup tui`. If the problem persists, consider rising an
[issue](https://github.com/ucsd-progsys/liquidhaskell/issues).

### Symlink

The simplest way to use our build in a project is
to create a symlink to our `liquidhaskell` local repository in our project.

```sh
ln -s /path/to/liquidhaskell/ /path/to/lh-tutorial/
```

For `cabal build` to pick it, we point to it in a
`cabal.project` file within the `lh-tutorial` project.

```sh
echo "package: . liquidhaskell" > cabal.project
```

### Local Repository

As an alternative, we can create a local package repository for cabal to
fetch the dependency from instead of [Hackage](https://hackage.haskell.org/).
The advantage of this approach is that we can easily use our build across projects.

First, build the `liquidhaskell` package tarball and place it in the directory
intended for the local repository. You can do this by running this command at the
source repository root.

```sh
cabal sdist -o /absolute/path/to/local/repository
```

Now declare a
[local no-index repository](https://cabal.readthedocs.io/en/3.14/config.html#local-no-index-repositories)
in the cabal configuration file (typically found at `$HOME/.cabal/config` or
`$HOME/.config/cabal/config`) by adding this line:

```cabal
repository local-repo
  url: file+noindex:///absolute/path/to/local/repository
```

We also need to set our local repository to have a
higher priority than Hackage. This is done by editing the
`active-repositories` [field](https://cabal.readthedocs.io/en/3.14/cabal-project-description-file.html#cfg-field-active-repositories)
in the cabal config file, putting the local repository at the end of the list.

```cabal
active-repositories: hackage.haskell.org, local-repo
```

Running `cabal update` makes cabal discover and merge our repository into its index.
In this way, every local project having `liquidhakell == 0.9.10.1.2` as a dependency
will fetch our tarball build instead of Hackage's.
A downside of this approach is that we will need to erase the `.cache` file
(that cabal puts in our local repository directory) whenever we update our build
tarball.

## Wrap Up

In this tutorial, we covered how to enable and use Liquid Haskell in a project,
as well as how to apply your local build across multiple projects. This should
give you everything needed to start building with LH or contribute to its source
code. Now go break some (verified) code!
