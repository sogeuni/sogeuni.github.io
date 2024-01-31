---
title: 21. Subshells
---


Running a shell script launches a new process, a _subshell_.

> **Definition:** A _subshell_ is a [[othertypesv#^CHILDREF2|child process]] launched by a shell (or _shell script_).

A subshell is a separate instance of the command processor -- the _shell_ that gives you the prompt at the console or in an _xterm_ window. Just as your commands are interpreted at the command-line prompt, similarly does a script [[external-filters-programs-and-commands#^BATCHPROCREF|batch-process]] a list of commands. Each shell script running is, in effect, a subprocess (_child process_) of the [[internal-commands-and-builtins#^FORKREF|parent]] shell.

A shell script can itself launch subprocesses. These _subshells_ let the script do parallel processing, in effect executing multiple subtasks simultaneously.

```bash
#!/bin/bash
# subshell-test.sh

(
# Inside parentheses, and therefore a subshell . . .
while [ 1 ]   # Endless loop.
do
  echo "Subshell running . . ."
done
)

#  Script will run forever,
#+ or at least until terminated by a Ctl-C.

exit $?  # End of script (but will never get here).



Now, run the script:
sh subshell-test.sh

And, while the script is running, from a different xterm:
ps -ef | grep subshell-test.sh

UID       PID   PPID  C STIME TTY      TIME     CMD
500       2698  2502  0 14:26 pts/4    00:00:00 sh subshell-test.sh
500       2699  2698 21 14:26 pts/4    00:00:24 sh subshell-test.sh

          ^^^^

Analysis:
PID 2698, the script, launched PID 2699, the subshell.

Note: The "UID ..." line would be filtered out by the "grep" command,
but is shown here for illustrative purposes.
```

In general, an [[external-filters-programs-and-commands#^EXTERNALREF|external command]] in a script [[internal-commands-and-builtins#^FORKREF|forks off]] a subprocess, [^1] whereas a Bash [[internal-commands-and-builtins|builtin]] does not. For this reason, builtins execute more quickly and use fewer system resources than their external command equivalents.

**Command List within Parentheses**

( command1; command2; command3; ... )

A command list embedded between _parentheses_ runs as a subshell.

Variables in a subshell are _not_ visible outside the block of code in the subshell. They are not accessible to the [[internal-commands-and-builtins#^FORKREF|parent process]], to the shell that launched the subshell. These are, in effect, variables [[local-variables#^LOCALREF|local]] to the _child process_.

###### Example 21-1. Variable scope in a subshell

```bash
#!/bin/bash
# subshell.sh

echo

echo "We are outside the subshell."
echo "Subshell level OUTSIDE subshell = $BASH_SUBSHELL"
# Bash, version 3, adds the new         $BASH_SUBSHELL variable.
echo; echo

outer_variable=Outer
global_variable=
#  Define global variable for "storage" of
#+ value of subshell variable.

(
echo "We are inside the subshell."
echo "Subshell level INSIDE subshell = $BASH_SUBSHELL"
inner_variable=Inner

echo "From inside subshell, \"inner_variable\" = $inner_variable"
echo "From inside subshell, \"outer\" = $outer_variable"

global_variable="$inner_variable"   #  Will this allow "exporting"
                                    #+ a subshell variable?
)

echo; echo
echo "We are outside the subshell."
echo "Subshell level OUTSIDE subshell = $BASH_SUBSHELL"
echo

if [ -z "$inner_variable" ]
then
  echo "inner_variable undefined in main body of shell"
else
  echo "inner_variable defined in main body of shell"
fi

echo "From main body of shell, \"inner_variable\" = $inner_variable"
#  $inner_variable will show as blank (uninitialized)
#+ because variables defined in a subshell are "local variables".
#  Is there a remedy for this?
echo "global_variable = "$global_variable""  # Why doesn't this work?

echo

# =======================================================================

# Additionally ...

echo "-----------------"; echo

var=41                                                 # Global variable.

( let "var+=1"; echo "\$var INSIDE subshell = $var" )  # 42

echo "\$var OUTSIDE subshell = $var"                   # 41
#  Variable operations inside a subshell, even to a GLOBAL variable
#+ do not affect the value of the variable outside the subshell!


exit 0

#  Question:
#  --------
#  Once having exited a subshell,
#+ is there any way to reenter that very same subshell
#+ to modify or access the subshell variables?
```

See also [[another-look-at-variables#^BASHPIDREF|$BASHPID]] and [[Example 34-2|Example 34-2]].

> **Definition:** The _scope_ of a variable is the context in which it has meaning, in which it has a _value_ that can be referenced. For example, the scope of a [[local-variables#^LOCALREF1|local variable]] lies only within the function, block of code, or subshell within which it is defined, while the scope of a _global_ variable is the entire script in which it appears.

> [!note]
> While the [[another-look-at-variables#^BASHSUBSHELLREF|$BASH_SUBSHELL]] internal variable indicates the nesting level of a subshell, the [[another-look-at-variables#^SHLVLREF|$SHLVL]] variable _shows no change_ within a subshell.
>
> ```bash
> echo " \$BASH_SUBSHELL outside subshell       = $BASH_SUBSHELL"           # 0
>   ( echo " \$BASH_SUBSHELL inside subshell        = $BASH_SUBSHELL" )     # 1
>   ( ( echo " \$BASH_SUBSHELL inside nested subshell = $BASH_SUBSHELL" ) ) # 2
> # ^ ^                           *** nested ***                        ^ ^
> 
> echo
> 
> echo " \$SHLVL outside subshell = $SHLVL"       # 3
> ( echo " \$SHLVL inside subshell  = $SHLVL" )   # 3 (No change!)
> ```

Directory changes made in a subshell do not carry over to the parent shell.

###### Example 21-2. List User Profiles

```bash
#!/bin/bash
# allprofs.sh: Print all user profiles.

# This script written by Heiner Steven, and modified by the document author.

FILE=.bashrc  #  File containing user profile,
              #+ was ".profile" in original script.

for home in `awk -F: '{print $6}' /etc/passwd`
do
  [ -d "$home" ] | continue    # If no home directory, go to next.
  [ -r "$home" ] | continue    # If not readable, go to next.
  (cd $home; [ -e $FILE ] && less $FILE)
done

#  When script terminates, there is no need to 'cd' back to original directory,
#+ because 'cd $home' takes place in a subshell.

exit 0
```

A subshell may be used to set up a "dedicated environment" for a command group.

```bash
COMMAND1
COMMAND2
COMMAND3
(
  IFS=:
  PATH=/bin
  unset TERMINFO
  set -C
  shift 5
  COMMAND4
  COMMAND5
  exit 3 # Only exits the subshell!
)
# The parent shell has not been affected, and the environment is preserved.
COMMAND6
COMMAND7
```

As seen here, the [[internal-commands-and-builtins#^EXITREF|exit]] command only terminates the subshell in which it is running, _not_ the parent shell or script.

One application of such a "dedicated environment" is testing whether a variable is defined.

```bash
if (set -u; : $variable) 2> /dev/null
then
  echo "Variable is set."
fi     #  Variable has been set in current script,
       #+ or is an an internal Bash variable,
       #+ or is present in environment (has been exported).

# Could also be written [[ ${variable-x} != x | ${variable-y} != y ]]
# or                    [[ ${variable-x} != x$variable ]]
# or                    [[ ${variable+x} = x ]]
# or                    [[ ${variable-x} != x ]]
```

Another application is checking for a lock file:

```bash
if (set -C; : > lock_file) 2> /dev/null
then
  :   # lock_file didn't exist: no user running the script
else
  echo "Another user is already running that script."
exit 65
fi

#  Code snippet by St�phane Chazelas,
#+ with modifications by Paulo Marcel Coelho Aragao.
```

+

Processes may execute in parallel within different subshells. This permits breaking a complex task into subcomponents processed concurrently.

###### Example 21-3. Running parallel processes in subshells

```bash
(cat list1 list2 list3 | sort | uniq > list123) &
	(cat list4 list5 list6 | sort | uniq > list456) &
	# Merges and sorts both sets of lists simultaneously.
	# Running in background ensures parallel execution.
	#
	# Same effect as
	#   cat list1 list2 list3 | sort | uniq > list123 &
	#   cat list4 list5 list6 | sort | uniq > list456 &
	
	wait   # Don't execute the next command until subshells finish.
	
	diff list123 list456
```

Redirecting I/O to a subshell uses the "|" pipe operator, as in **ls -al | (command)**.

> [!note]
> A code block between [[special-characters#^CODEBLOCKREF|curly brackets]] does _not_ launch a subshell.
>
> { command1; command2; command3; . . . commandN; }
>
> ```bash
> var1=23
> echo "$var1"   # 23
>
> { var1=76; }
> echo "$var1"   # 76
> ```

[[internal-commands-and-builtins#^EXECREF|^1]: An external command invoked with an [exec]] does _not_ (usually) fork off a subprocess / subshell.
