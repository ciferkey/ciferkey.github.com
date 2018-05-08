---
layout: single
title: H320 Research Paper Response - Simplifying and Isolating Failure-Inducing Input
categories:
  - University
---
This semester I am currently enrolled in a one credit honors extension to my SE class at the University of Massachusetts Amherst. We recently read this paper[paper](http://pag-www.gtisc.gatech.edu/courses/common/zeller-tse02.pdf) and were asked to response.  He is my take on it:


## Problem
“Given some test case, a program fails. Which circumstances of the test case are responsible for the particular failure?”

## Solution
“...generalizes and simplifies the failing test case to a minimal test case that still produces the failure” by repeatedly dividing the test case to isolate a locally minimal (1-minimal) input.

## How well did it work?
Three test cases were presented:  In the GCC case study the improved version DD required only 59 tests and points to a relative difference of only two characters.  The fuzzed unix input case study saw a similar size reduction as well.  The firefox case study was able to reduce a complex description and large (40K character) html document down to a handful of x events and selects.

## Related Work
While it is common to manually apply the divide and conquer approach to isolate failures, at the time of writing there were no other automated approaches to simplifying real test cases.  However Slutz had described and automated approach for artificially produced test cases in his work on stress testing databases.

The DD algorithm is also a successor to DD+ in that DD+ is not well suited for failures caused by large combinations of changes and dd+ assumes monotonicity.

## Contribution
The work is helpful for simplifying bug reports down to the minimal amount of information necessary to reproduce a bug.  Minimal information makes reproducing bugs easier which is crucial to quickly resolving bugs.

## Questions
Where is this work today are there automated frameworks available that implement something similar to the approach?

~ Matt
