---
title: 20.1. Using _exec_
---


An **exec <filename** command redirects stdin to a file. From that point on, all stdin comes from that file, rather than its normal source (usually keyboard input). This provides a method of reading a file line by line and possibly parsing each line of input using [[a-sed-and-awk-micro-primer#^SEDREF|sed]] and/or [[awk#^AWKREF|awk]].

![[Example 20-1|Example 20-1]]

Similarly, an **exec >filename** command redirects stdout to a designated file. This sends all command output that would normally go to stdout to that file.

> [!important]
> **exec N > filename** affects the entire script or _current shell_. Redirection in the [[special-characters#^PROCESSIDREF|PID]] of the script or shell from that point on has changed. However . . .
>
> **N > filename** affects only the newly-forked process, not the entire script or shell.
>
> Thank you, Ahmed Darwish, for pointing this out.

![[Example 20-2|Example 20-2]]

![[Example 20-3|Example 20-3]]

I/O redirection is a clever way of avoiding the dreaded [[subshells#^PARVIS|inaccessible variables within a subshell]] problem.

![[Example 20-4|Example 20-4]]
