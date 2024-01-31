---
title: 25. Aliases
---


A Bash *alias* is essentially nothing more than a keyboard shortcut, an abbreviation, a means of avoiding typing a long command sequence. If, for example, we include **alias lm="ls -l | more"** in the [[sample-bashrc-and-bash-profile-files|~/.bashrc file]], then each **lm** [^1] typed at the command-line will automatically be replaced by a **ls -l | more**. This can save a great deal of typing at the command-line and avoid having to remember complex combinations of commands and options. Setting **alias rm="rm -i"** (interactive mode delete) may save a good deal of grief, since it can prevent inadvertently deleting important files. ^ALIASREF

In a script, aliases have very limited usefulness. It would be nice if aliases could assume some of the functionality of the **C** preprocessor, such as macro expansion, but unfortunately Bash does not expand arguments within the alias body. [^2] Moreover, a script fails to expand an alias itself within "compound constructs," such as [[tests#^IFTHEN|if/then]] statements, loops, and functions. An added limitation is that an alias will not expand recursively. Almost invariably, whatever we would like an alias to do could be accomplished much more effectively with a [[functions|function]].

![[Example 25-1|Example 25-1]]

The **unalias** command removes a previously set *alias*. ^UNALIASREF

![[Example 25-2|Example 25-2]]

```bash
bash$ ./unalias.sh
total 6
drwxrwxr-x    2 bozo     bozo         3072 Feb  6 14:04 .
drwxr-xr-x   40 bozo     bozo         2048 Feb  6 14:04 ..
-rwxr-xr-x    1 bozo     bozo          199 Feb  6 14:04 unalias.sh

./unalias.sh: llm: command not found
```

[^1]: ... as the first word of a command string. Obviously, an alias is only meaningful at the *beginning* of a command.

[^2]: However, aliases do seem to expand positional parameters.
