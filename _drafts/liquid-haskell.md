---
title: "A LiquidHaskell tutorial"
layout: post
author: Xavier Góngora
categories:
- programming languages
- program verification
- haskell
---
I'm currently doing some research to elaborate a proposal for this year's
[Google Summer of Code]() as a Haskell contributor.
For this I've chosen the project idea put forward by [Facundo Dominguez](), 
which involves fixing [LiquidHaskell](https://ucsd-progsys.github.io/liquidhaskell/) (LH)
implementation of name resolution for type and predicate aliases.

In this post we'll introduce the basics of LH installation a usage as a warm up.

Most relevant information can be found in the
[LiquidHaskell documentation](https://ucsd-progsys.github.io/liquidhaskell/),
but here I'll following an specific path to build a dummy library from scratch
for demonstration purposes.
As a bonus, we'll review how to set the project to test changes during LH development.

## Installation

Of course we'll need the Haskell toolchain first: `cabal` to manage the project and the `ghc` Haskell compiler.
The (current) recommended way to install and manage both is through [GHCup](https://www.haskell.org/ghcup/).[^nix]
After installing it with the default options, install the `ghc` version corresponding to the latest LH version (check the docs, at the time of writing this is 9.10.1).[^ghc-policy]

[^nix]: Other common alternatives are using the [Stack build tool]() or the [Nix package manager](https://nixos.org/). Nix is a comprehensive tool (and language!) for reproducible package deployment, which can also be use to create declarative development environments, that might be the topic of a future post.

```sh
ghcup install ghc 9.10.1
```

The following command bootstraps the creation of our project.
It creates a new `cabal` project for our library with the corresponding `base` and `liquidhaskell` versions as dependencies
and configures it to use the proper version of `ghc`. 

[^ghc-policy]: As mentioned in the [LH source repository README](https://github.com/ucsd-progsys/liquidhaskell/), LH is developed against a specific version of `ghc` given its tight dependence on the `ghc` library which tends to break existing code without notice (in particular, because a distinction does not yet exists between public and internal API's).

```sh
mkdir lh-tutorial && cd lh-tutorial && cabal init --lib --dependency="base==4.20.0.0,liquidhaskell==0.9.10.1 && cabal configure --with-compiler ghc-9.10.1
```

Finally, build the project with `cabal build`. If the build suceeds, now LH has been properly installed within the project.

LH is implemented as a GHC plugin, which modifies the compiler pipeline to deliver the extracted properties to a SMT solver for verification.
For it to work, you must have any of the following installed in your system:
[Z3](https://github.com/Z3Prover/z3), [CVC4](https://cvc4.github.io/) or [MathSat](https://mathsat.fbk.eu/).
The LH docs recommend Z3, which is available in the Arch Linux repositories (`sudo pacman -S z3`), and probably on other Linux distributions as well. 
It can be installed on MacOs using [Homebrew](https://brew.sh/) (`brew install z3`).

## Building LiquidHaskell

Clone LH with the `--recurse-submodules` flag so that `liquid-fixpoint` is also cloned.
This is needed for the build.

```sh
git clone --recurse-submodules https://github.com/ucsd-progsys/liquidhaskell.git
```
 The build instructions at the main
[source repository](https://github.com/ucsd-progsys/liquidhaskell/blob/192b8766a521c6bef8be2b61c6fda3a1b53783fb/README.md).
README sugggest the following sequence of commands to build and test the build on a single `FILE.hs`:

```sh
cabal build liquidhaskell
cabal exec ghc -- -fplugin=LiquidHaskell FILE.hs
```

However, this is not enough as we intend to use our build to test name resolution across modules in a project. 
To accomplish this, we can create a symlink to our `liquidhaskell` local repository in our project.

```sh
ln -s /path/to/liquidhaskell/ /path/to/lh-tutorial/
```

`cabal` will prefer our local package over a remote one, so its enough to specify it in a `cabal.project` file
within the `lh-tutorial` project.

```sh
echo "package: . liquidhaskell" > cabal.project
```


## Use local build

For other alternatives, see this [answer at Stackoverflow](https://stackoverflow.com/questions/69773999/how-do-i-get-cabal-to-use-a-local-version-of-a-package-as-a-dependency-for-a-hac).

## Basic refinement of types

Let's start by modifying `src/MyLib.hs` to have the following contents.



