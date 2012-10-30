---
layout: post
title: Monads, Not a Thing I Made Up
categories:
  - Thoery
---

I was reading about [Curry–Howard Correspondence](http://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence) after encountering it again in an [Article](http://pragprog.com/magazines/2012-09/thinking-functionally-with-haskell) by Paul Callaghan (I believe I first encounter it in a talk by Simon Peyton Jones). When I reached the final paragraph of the first second I had a real a-ha moment and just had to read the paragraph a second time.  The part the interested me was:

> “Because of the possibility of writing non-terminating programs, Turing-complete models of computation (such as languages with arbitrary recursive functions) must be interpreted with care, as naive application of the correspondence leads to an inconsistent logic. The best way of dealing with arbitrary computation from a logical point of view is still an actively debated research question, but one popular approach is based on using monads to segregate provably terminating from potentially non-terminating code.”

Its little connection like this that help me with understanding topics such as monads.

P.S. the title comes from the fact that my friends claim the word monad is something I made up (and certainly not a real thing).

~ Matt
