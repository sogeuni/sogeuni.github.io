---
title: 36.9. Portability Issues
---


> It is easier to port a shell than a shell script.
>
> --<cite>Larry Wall</cite>

This book deals specifically with Bash scripting on a GNU/Linux system. All the same, users of **sh** and **ksh** will find much of value here.

As it happens, many of the various shells and scripting languages seem to be converging toward the [[sha-bang#^POSIX2REF|POSIX]] 1003.2 standard. Invoking Bash with the --posix option or inserting a **set -o posix** at the head of a script causes Bash to conform very closely to this standard. Another alternative is to use a _#!/bin/sh_ [[sha-bang#^SHABANGREF|sha-bang header]] in the script, rather than _#!/bin/bash_. [^1] Note that /bin/sh is a [[external-filters-programs-and-commands#^LINKREF|link]] to /bin/bash in Linux and certain other flavors of UNIX, and a script invoked this way disables extended Bash functionality.

Most Bash scripts will run as-is under **ksh**, and vice-versa, since Chet Ramey has been busily porting **ksh** features to the latest versions of Bash.

On a commercial UNIX machine, scripts using GNU-specific features of standard commands may not work. This has become less of a problem in the last few years, as the GNU utilities have pretty much displaced their proprietary counterparts even on "big-iron" UNIX. [Caldera's release of the source](http://linux.oreillynet.com/pub/a/linux/2002/02/28/caldera.html) to many of the original UNIX utilities has accelerated the trend.

Bash has certain features that the traditional [[shell-programming#^BASHDEF|Bourne shell]] lacks. Among these are:

- Certain extended [[options#^INVOCATIONOPTIONSREF|invocation options]]
- [[command-substitution#^COMMANDSUBREF|Command substitution]] using **$( )** notation
- [[bash-version-2-3-and-4#^BRACEEXPREF3|Brace expansion]]
- Certain [[arrays#^ARRAYREF|array]] operations, and [[bash-version-2-3-and-4#^ASSOCARR|associative arrays]]
- The [[tests#^DBLBRACKETS|double brackets]] extended test construct
- The [[operations-and-related-topics#^DBLPARENSREF|double-parentheses]] arithmetic-evaluation construct
- Certain [[manipulating-variables#^STRINGMANIP|string manipulation]] operations
- [[process-substitution#^PROCESSSUBREF|Process substitution]]
- A Regular Expression [[bash-version-2-3-and-4#^REGEXMATCHREF|matching operator]]
- Bash-specific [[internal-commands-and-builtins|builtins]]
- [[bash-version-2-3-and-4#^COPROCREF|Coprocesses]]

See the [Bash F.A.Q.](ftp://ftp.cwru.edu/pub/bash/FAQ) for a complete listing.

## 36.9.1. A Test Suite

Let us illustrate some of the incompatibilities between Bash and the classic Bourne shell. Download and install the ["Heirloom Bourne Shell"](http://freshmeat.net/projects/bournesh) and run the following script, first using Bash, then the classic _sh_.

![[Example 36-23|Example 36-23]]

[^1]: Or, better yet, [[system-and-administrative-commands#^ENVV2REF|#!/bin/env sh]].
