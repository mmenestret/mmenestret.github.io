---
layout: post
comments: true
title: "A minimal Haskell development environment for beginners with VS Code"
author: "Myself"
date: 2018-09-28
tags: Haskell
---

---

Recently, after several failed attemps, I decided to commit to learning [Haskell](https://www.haskell.org/).

For those who don't know what _Haskell_ is (run), here's the _Wikipedia_ definition:

> Haskell /ˈhæskəl/[27] is a standardized, general-purpose compiled purely functional programming language, with non-strict semantics and strong static typing.

_Haskell_ is seen by many functional programmers as THE functional programming language, and for now (maybe I'll be disillusioned someday) I'm one of them.

This short blog post presents the small work environment I'm using and which fits my current needs (which are mostly doing exercises and toy projects).

I know that it probably won't be suitable for big projects, but if that is your need, you're not the target here ;).

# Haskell

Let's start [here](https://www.haskell.org/downloads).

Download the and install the _Haskell Platform_ (bottom of the page) for your OS, which is an all in one bundle.

You will get:
  - __GHC__ : The _Haskell_ compiler
  - __Stack__: A cross-platform build tool for Haskell that handles management of the toolchain
  - __Cabal__: A build system which can install new packages, and by default fetches from _Hackage_, the central _Haskell_ package repository (the distinction with Stack is not clear for me right now)
  - The main and most useful _Haskell_ packages

## GHCi

It comes with the _Haskell Platform_ you downloaded.

- _GHCi_ is the _GHC_ interactive environment (a context sensitive [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) with _auto completion_)
- You can start it in the context of your project (just run `ghci` at your project's root) and run the functions you have in scope
- It is a great development tool since it allows you to interact with the functions you write as you write them just by reloading your sources in _GHCi_ (`:r` in the _REPL_) and calling them
- You can also use
  - `:t` and the name of a function (`:t map`) to get back the type of it (`map :: (a -> b) -> List a -> List b`)
  - `:i` and the name of a function or data type to get infos on it

# GHCid

Get it [here](https://github.com/ndmitchell/ghcid).

- _GHCid_ is a _GHCi_ daemon that will automatically reload your sources in a _GHCi_ and tell you about the potential compilation errors it encountered  

# VS Code

Get it [here](https://code.visualstudio.com/).

I chose to work on _VS Code_ because it seems to have pretty good _Haskell_ extensions and it is fast and light (especially coming from _IntelliJ_...)

## Extentions

- _Haskell Language Server_ is an interface with a the _Haskell IDE engine_ that comes with _Haskell_. It gives you:
  - Errors and warning as you type
  - Information on hover / highlight
  - Jump to definitions (_F12_)
  - Completion
  - Fixes suggestions
  - Renaming
  - And a lot more

- _Haskell Syntax Highlight_ (self explanatory)

- _hoogle-vscode_: lets you make a [Hoogle](https://www.haskell.org/hoogle/) search from _VS Code_.
  - _Hoogle_ is like a function dictionary website for Haskell
  - All you have to do is type a function type like: `(a -> b) -> [a] -> [b]` ([here](https://www.haskell.org/hoogle/?hoogle=%28a+-%3E+b%29+-%3E+%5Ba%5D+-%3E+%5Bb%5D)) and _Hoogle_ gives you back the list of functions that match or almost match your request

# Wrapping everything together

After having installed everything:

- Open your _VS Code_ at the root of your project
- Open two terminals side by side at the bottom of your screen and get to the root of your project
  - On the first one run the following: `ghcid "--command=ghci"` which starts _GHCid_
  - On the second one run `ghci` which will open a REPL in the context of your project (in that one you'll have to `:r` to try the functions that you write)
- Now you're all set, no more excuses :D

[![Everything's OK](/ressources/haskell-setup/OK.png)](/ressources/haskell-setup/OK.png)
_Click image to enlarge_
[![Something's wrong](/ressources/haskell-setup/KO.png)](/ressources/haskell-setup/KO.png)
_Click image to enlarge_

# Conclusion

That's a small but good enough setup for me now.

I'll try to update my post if I find new interesting stuff to add to that setup.
Feel free to contact me if anything needs to be corrected or if you feel so :).

Happy Haskelling !

---

___Edit__: I felt like Haskell Language Server was taking a lot of memory and... [indeed it does](https://github.com/haskell/haskell-ide-engine/issues/665).
I hope it's gonna be fixed soon, in the meantime, you can desactivate and reactivate it in your VS Code extensions when it gets too greedy..._