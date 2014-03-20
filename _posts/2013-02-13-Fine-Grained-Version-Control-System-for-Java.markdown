---
layout: post
title: H320 Research Paper Response - Fine-Grained Version Control System for Java
categories:
  - University
---
This semester I am currently enrolled in a one credit honors extension to my SE class at the University of Massachusetts Amherst. We recently read this paper[paper](https://dl.acm.org/citation.cfm?id=2024463) and were asked to response.  He is my take on it:

## Problem
Tracking the evolution of software systems can provide valuable insight. However few solutions exist to track changes on a fine grained (classes, methods, etc) level.

## Solution
Historage builds on top of the coarse grained (line-by-line file diff) level version control of git to provide the ability to trace changes of entities (moving, remaining, etc).

## How well did it work?
Historage was applied to five open source Java software projects: WTP incubator, Hadoop, Subversion, jEdit, and the Android Framework.  It provides fine-grained entity remaining at a precision of 97% (when the matching threshold is 30%).

## Related Work
Kim an Notkin provided a method for one-to-one matching techniques (based off syntax and the code graph) as well as origin analysis (renaming and moving) and achieved a function body diff detection rate of 90.2%.  Godfrey and Zou proposed a technique for inferring events based on splitting and merging.  RefactingCrawler applied the idea of detecting common code refactorings in code changes.  Xing and Stroulia proposed an approach for monitoring API evolution.

## Contribution
Fine-grained VCS can provide valuable information about software systems, such as historical changes to the code base and the evolution of the system. The fact that it can be implemented on top of existing VCS using pre existing changesets means it can be applied to previously existing projects.

## Questions
Do certain languages lend themselves better for fine grained tracking? (For example Haskell is quite concise and fits quite a lot of information into only a few lines).

~ Matt
