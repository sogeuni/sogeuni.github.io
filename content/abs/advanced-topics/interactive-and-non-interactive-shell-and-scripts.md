---
title: 36.1. Interactive and non-interactive shells and scripts
---


An _interactive_ shell reads commands from user input on a tty. Among other things, such a shell reads startup files on activation, displays a prompt, and enables job control by default. The user can _interact_ with the shell.

A shell running a script is always a non-interactive shell. All the same, the script can still access its tty. It is even possible to emulate an interactive shell in a script.

```bash
#!/bin/bash
MY_PROMPT='$ '
while :
do
  echo -n "$MY_PROMPT"
  read line
  eval "$line"
  done

exit 0

# This example script, and much of the above explanation supplied by
# Stéphane Chazelas (thanks again).
```

Let us consider an _interactive_ script to be one that requires input from the user, usually with [[../commands/internal-commands-and-builtins#^READREF|read]] statements (see [[../commands/internal-commands-and-builtins#^EX36|Example 15-3]]). "Real life" is actually a bit messier than that. For now, assume an interactive script is bound to a tty, a script that a user has invoked from the console or an _xterm_.

Init and startup scripts are necessarily non-interactive, since they must run without human intervention. Many administrative and system maintenance scripts are likewise non-interactive. Unvarying repetitive tasks cry out for automation by non-interactive scripts.

Non-interactive scripts can run in the background, but interactive ones hang, waiting for input that never comes. Handle that difficulty by having an **expect** script or embedded [[./here-documents#^HEREDOCREF|here document]] feed input to an interactive script running as a background job. In the simplest case, redirect a file to supply input to a **read** statement (**read variable <file**). These particular workarounds make possible general purpose scripts that run in either interactive or non-interactive modes.

If a script needs to test whether it is running in an interactive shell, it is simply a matter of finding whether the _prompt_ variable, [[../beyond-the-basic/another-look-at-variables#^PS1REF|$PS1]] is set. (If the user is being prompted for input, then the script needs to display a prompt.)

```bash
if [ -z $PS1 ] # no prompt?
### if [ -v PS1 ]   # On Bash 4.2+ ...
then
  # non-interactive
  ...
else
  # interactive
  ...
fi
```

Alternatively, the script can test for the presence of option "i" in the [[../beyond-the-basic/another-look-at-variables#^FLPREF|$-]] flag.

```bash
case $- in
*i*)    # interactive shell
;;
*)      # non-interactive shell
;;
# (Courtesy of "UNIX F.A.Q.," 1993)
```

However, John Lange describes an alternative method, using the [[../basic/tests#^TERMTEST|-t _test_ operator]].

```bash
# Test for a terminal!

fd=0   # stdin

#  As we recall, the -t test option checks whether the stdin, [ -t 0 ],
#+ or stdout, [ -t 1 ], in a given script is running in a terminal.
if [ -t "$fd" ]
then
  echo interactive
else
  echo non-interactive
fi


#  But, as John points out:
#    if [ -t 0 ] works ... when you're logged in locally
#    but fails when you invoke the command remotely via ssh.
#    So for a true test you also have to test for a socket.

if [[ -t "$fd" | -p /dev/stdin ]]
then
  echo interactive
else
  echo non-interactive
fi
```

> [!note] Scripts may be forced to run in interactive mode with the -i option or with a **#!/bin/bash -i** header. Be aware that this can cause erratic script behavior or show error messages even when no error is present.
