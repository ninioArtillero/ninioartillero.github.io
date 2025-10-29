---
title: "Using Doom Emacs as Haskell IDE"
layout: post
author: Xavier Góngora
locale: en_US
tags:
- workflow
- editor
- configuration
---

Setting up Doom Emacs as a Haskell IDE is rather straightforward.
The goal of this little post is to document the knowledge I've gathered regarding
the components that provide the related functionality, provide a basic setup
tutorial, and show some configuration options relevant for a Nix centered
development workflow. Hopefully this will be of use for other folks to get
up and running as quick as possible.

## The one step tutorial

In essence, all you need to do is to uncomment the `haskell`
module under the `:lang` section of the `$DOOMDIR/init.el`, and make sure
to enable some useful flags.

```elisp

(doom! ...

       :lang

       ...

       (haskell +lsp +tree-sitter)

       ...)
       
```

Relevant key-bindings.

Configuration options.

## Nix workflow

Install nix-direnv: install using nix + hook into shell

Configure executable. See this [issue](https://github.com/input-output-hk/haskell.nix/issues/1776).

```elisp
(setq lsp-haskell-server-path "haskell-language-server")
```

## Hard won knowledge

At the moment of writing, the `+tree-sitter` flag
changes the automatically enabled major mode for Haskell files from the
traditional `haskell-mode` to the uprising `haskell-ts-mode`. The latter is
newer and piggy-backs on tree-sitter and native emacs functionality.
One minor caveat though, is that lsp won't start automatically, but it is
needed to be run manually `M-x lsp`.

`lsp-mode` vs `eglot`.
