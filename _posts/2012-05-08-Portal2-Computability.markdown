---
layout: post
title: Portal 2 Computability
categories:
  - Thoery
---

The Portal 2 map editor was released today and I was wondering how it could be used as a sandbox for computability like Minecraft is. I have an example map in the  [workshop](http://steamcommunity.com/sharedfiles/filedetails/?id=68513261) . I'm just going to copy-paste the explanation I gave there:

> When I opened the editor my first thought was to build something crazy with the gels. I prompty became bored with the idea. However innovation never rests! My second instinct was to see if Portal 2 could be used to build basic logic gates and to consider the game in terms of computability.

> When you enter, to the left is and AND gate followed by an OR gate (I cheated somewhat and used AND and NOT to create OR rather than synthesis a unique approach). To you right is a NOT gate followed by a clock.

> I've been able to make similar gates using mechanical components, but I prefer the "solid state" nature of the lasers.

> The nice part about constucting these is that (1) inputs and outputs can be connected in a many-to-many relationship and (2) outputs can be set to on or off initially (with a signal toggling them). So AND is simply two input piped into one output; it won't fire if they are not both on. NOT is piping an input into an output with the output on initially. OR I synthesised from the fact that a OR b <-> NOT(NOT(a) AND NOT(b)).

>I choose to use block buttons for inputs and lasters for outputs, but the choice is arbitrary. Any of the inputs and outputs should work.

Thoughts comments suggestions? I posted this same brain dump on [reddit](http://www.reddit.com/r/compsci/comments/tdul2/any_portal_2_fans_thoughts_on_computability_in/) if thats your cup of tea.

~Matt
