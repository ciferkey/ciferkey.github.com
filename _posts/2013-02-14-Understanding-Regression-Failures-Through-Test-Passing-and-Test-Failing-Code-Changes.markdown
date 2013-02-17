---
layout: post
title: H320 Research Paper Response - Understanding Regression Failures Through Test-Passing and Test-Failing Code Changes
categories:
  - University
---
This semester I am currently enrolled in a one credit honors extension to my SE class at the University of Massachusetts Amherst. We recently read this paper [paper](http://people.cs.umass.edu/~brun/pubs/pubs/Sukkerd13icse-nier.pdf) which my Professor [Yuriy Brun](http://people.cs.umass.edu/~brun/) is working on.  He is my take on it:

## Problem
Isolating the source code changes that are responsible for test failures is time consuming and challenging.

## Solution
Source code change are broken down into subsets with the intent of isolating the failure cause.  Two subsets, the minimal set of changes that produce the failure, and the complement of the maximum set of changes that pass, are both helpful for finding failure causes. Neither always contains the correct section and the combination leads to greater coverage of failure causes.

## How well did it work?
In 78% of the 45 regression failures studied the complement of the maximum set of changes that pass contained relevant information not in the minimal set of changes that produce the failure.

## Related Work
M. Eaddy et al. considered changes related to the most cross-cutting concerns.  N. Nagappan and T. Ball  looked into modules with the highest churn. T. Zimmermann et al. investigated changes to most recent changes. and T. Zimmermann and N. Nagappan worked with changes to modules with the most dependencies. A. Zeller and R. Hildebrandt developed an approach to finding the smallest subset of the recent changes that exhibits test failure.

## Contribution
The main contribution of the paper is the observations that the maximum set of changes that pass and the minimum set of changes that fail are often not complementary (rather than any particular approach to finding the complement of the max set or how the two set can be used together). The goals was to point out the counterintuitive finding.

## Questions
Even though the goal is to bring attention to the difference between the two sets, what approach was used with the preliminary study?

~ Matt
