---
title: "36.4. Recursion: a script calling itself"
---


Can a script [[local-variables#^RECURSIONREF|recursively]] call itself? Indeed.

![[Example 36-10|Example 36-10]]

![[Example 36-11|Example 36-11]]

![[Example 36-12|Example 36-12]]

> [!caution]
> Too many levels of recursion can exhaust the script's stack space, causing a segfault.
