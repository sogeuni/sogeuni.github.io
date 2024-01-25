---
title: Appendix C. A Sed and Awk Micro-Primer
---


This is a very brief introduction to the **sed** and **awk** text processing utilities. We will deal with only a few basic commands here, but that will suffice for understanding simple sed and awk constructs within shell scripts.

**sed**: a non-interactive text file editor

**awk**: a field-oriented pattern processing language with a **C**-style syntax

For all their differences, the two utilities share a similar invocation syntax, use [[regexp#^REGEXREF|regular expressions]] , read input by default from stdin, and output to stdout. These are well-behaved UNIX tools, and they work together well. The output from one can be piped to the other, and their combined capabilities give shell scripts some of the power of [[shell-wrappers#^PERLREF|Perl]].

> [!Note]
> One important difference between the utilities is that while shell scripts can easily pass arguments to sed, it is more cumbersome for awk (see [[shell-wrappers#^COLTOTALER|Example 36-5]] and [[indirect-references#^COLTOTALER2|Example 28-2]]).
