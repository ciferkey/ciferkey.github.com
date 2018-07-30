---
layout: single
classes: wide
title: "Kotlin Bencode Decoding and Encoding [Weekend Project]"
categories:
  - Projects
---

As a fun weekend project I tried my hand at rolling an encoder/decoder for [Bencode](https://en.wikipedia.org/wiki/Bencode) (the encoding format used in [torrent files](https://en.wikipedia.org/wiki/Torrent_file)).

The specification defining how Bencoding works is defined in [The BitTorrent Protocol Specification v2](http://www.bittorrent.org/beps/bep_0052.html) (which supersedes the [v1 specification](http://www.bittorrent.org/beps/bep_0003.html)). The section defining bencoding states:

> Strings are length-prefixed base ten followed by a colon and the string. For example 4:spam corresponds to 'spam'.
> Integers are represented by an 'i' followed by the number in base 10 followed by an 'e'. For example i3e corresponds to 3 and i-3e corresponds to -3. Integers have no size limitation. i-0e is invalid. All encodings with a leading zero, such as i03e, are invalid, other than i0e, which of course corresponds to 0.
> Lists are encoded as an 'l' followed by their elements (also bencoded) followed by an 'e'. For example l4:spam4:eggse corresponds to ['spam', 'eggs'].
> Dictionaries are encoded as a 'd' followed by a list of alternating keys and their corresponding values followed by an 'e'. For example, d3:cow3:moo4:spam4:eggse corresponds to {'cow': 'moo', 'spam': 'eggs'} and d4:spaml1:a1:bee corresponds to {'spam': ['a', 'b']}. Keys must be strings and appear in sorted order (sorted as raw strings, not alphanumerics).

This relatively straight forward specification lends itself to a simple decoding process:
* read the next character to parse and see if it matches one of the markers in the specification:
** The digits 1 through nine mark a bencoded string
** i marks a bencoded integer
** l marks a bencoded listed
** d marks a bencoded dictionary
* try to decode the next part of the sequence based on the marker, recursing for the lists and dictionaries

Internally I chose to represent the becoded information as an ADT using a [sealed class](https://kotlinlang.org/docs/reference/sealed-classes.html). Here is a simplified version of it (omitting methods) :
<script src="https://gitlab.com/ciferkey/kotlin-bencode/snippets/1738327.js"></script>

Additionally I wanted to try and produce meaningful error information when decoding failed. I enjoyed using [Results in Rust](https://doc.rust-lang.org/book/first-edition/error-handling.html#the-result-type) to do this before and wanted to see how it could work in Kotlin. I found a [Result library](https://github.com/kittinunf/Result) and refactored to use it. On the whole I found it easier to propagate error messages but I'm not sure if it created idiomatic Kotlin code.

The encoding process is even simpler:
* for a string prepend the length on the string with a ':' separating them
* for a integer wrap the base 10 string representation of the number in 'i' and 'e'
* for a list recursively encode the list elements, concatenate them and wrap them in an 'l' and 'e'
* for a dictionary recursively encode the key value pairs, concatenate them and wrap them in an 'd' and 'e'

As far as testing the decoding and encoding I found example torrent files from the Webtorrent [Testing Fixtures](https://github.com/webtorrent/webtorrent-fixtures) project which are available for use under Creative Commons. This was a great way to test real world example of bencoded data. On large file in particular even revealed a performance problem around bencode string parsing.

I am hoping to build a series off of this where I will continue implementing the different parts of the bittorrent specification.
