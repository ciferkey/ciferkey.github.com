---
layout: single
classes: wide
title: "Kotlin Bencode Decoding and Encoding"
categories:
  - Projects
---

As a fun weekend project I tried my hand at rolling an encoder/decoder for [Bencode](https://en.wikipedia.org/wiki/Bencode) (the encoding format used in [torrent files](https://en.wikipedia.org/wiki/Torrent_file)). Code for the project is [available](https://github.com/ciferkey/kotlin-bencode).

The specification for how Bencoding works is defined in [The BitTorrent Protocol Specification v2](http://www.bittorrent.org/beps/bep_0052.html) (which supersedes the [v1 specification](http://www.bittorrent.org/beps/bep_0003.html)). The section defining bencoding states:

*Decoding*

> * Strings are length-prefixed base ten followed by a colon and the string. For example 4:spam corresponds to 'spam'.
> * Integers are represented by an 'i' followed by the number in base 10 followed by an 'e'. For example i3e corresponds to 3 and i-3e corresponds to -3. Integers have no size limitation. i-0e is invalid. All encodings with a leading zero, such as i03e, are invalid, other than i0e, which of course corresponds to 0.
> * Lists are encoded as an 'l' followed by their elements (also bencoded) followed by an 'e'. For example l4:spam4:eggse corresponds to ['spam', 'eggs'].
> * Dictionaries are encoded as a 'd' followed by a list of alternating keys and their corresponding values followed by an 'e'. For example, d3:cow3:moo4:spam4:eggse corresponds to {'cow': 'moo', 'spam': 'eggs'} and d4:spaml1:a1:bee corresponds to {'spam': ['a', 'b']}. Keys must be strings and appear in sorted order (sorted as raw strings, not alphanumerics).

This relatively straight forward specification lends itself to a simple decoding process:
* read the next character to parse and see if it matches one of the markers in the specification:
 * The digits 0 through nine mark a bencoded string
 * i marks a bencoded integer
 * l marks a bencoded listed
 * d marks a bencoded dictionary
* try to decode the next part of the sequence based on the rules for that particular marker, recursing for lists and dictionaries

<script src="https://gist.github.com/ciferkey/bcfb3659286467c68a40198017ffafc1.js"></script>

It is a sequence of transformations over the Result type. Each of these steps could possibly fail:
 * read digits off the iterator. This could fail if the iterator does not have a next value.
 * convert them to an integer (representing the string's length). This could result in a parse failure.
 * try to consume the ':' separator. 
 * try to read the correct number of characters from the iterator

The upside of this approach is it requires no backtracking and only needs to look ahead a single character at a time. I chose to use [Guava's PeekingIterator](https://github.com/google/guava/wiki/CollectionHelpersExplained#peekingiterator) to assist this. It allows you to peek at the next value in an iterator without actually removing it (the same way one might peek at the top value in a stack). Additionally I implemented some helper [extension methods](https://kotlinlang.org/docs/reference/extensions.html) over PeekingIterator<Char> for common operations I was using to read characters off the iterator. For example reading characters while a condition is true:

<script src="https://gist.github.com/ciferkey/6782be7119a92f74feba1bebeba3218a.js"></script>

*Internal Representation*

Internally I chose to represent the becoded information as an ADT using a [sealed class](https://kotlinlang.org/docs/reference/sealed-classes.html). Here is a simplified version of it (omitting methods):

<script src="https://gist.github.com/ciferkey/94be77bac13b9f948342d6ee116e2c95.js"></script>

Additionally I wanted to try and produce meaningful error information when decoding failed. I enjoyed using [Results in Rust](https://doc.rust-lang.org/book/first-edition/error-handling.html#the-result-type) to do this before and wanted to see how it could work in Kotlin. I found a [Result library](https://github.com/kittinunf/Result) and refactored the decoder to use it. On the whole I found it easier to propagate error messages but I'm not sure if it created idiomatic Kotlin code. For example the Bencoded String parsing looks like:

<script src="https://gist.github.com/ciferkey/5fa44c91ff395666d440fff9e3ac3e30.js"></script>

*Encoding*

In comparison to decoding the encoding process is even simpler:
* for a string prepend the length on the string with a ':' separating them
* for a integer wrap the base 10 string representation of the number in 'i' and 'e'
* for a list recursively encode the list elements, concatenate them and wrap them in an 'l' and 'e'
* for a dictionary recursively encode the key value pairs, concatenate them and wrap them in an 'd' and 'e'

*Testing*

As far as testing the decoding and encoding I found example torrent files from the Webtorrent [Testing Fixtures](https://github.com/webtorrent/webtorrent-fixtures) project which are available for use under Creative Commons. This was a great way to test real world example of bencoded data. On large file in particular even revealed a performance problem around bencode string parsing.