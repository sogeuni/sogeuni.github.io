---
title: 32. Debugging
---


> Debugging is twice as hard as writing the code in the first place. Therefore, if you write the code as cleverly as possible, you are, by definition, not smart enough to debug it.
>
> --<cite>Brian Kernighan</cite>

The Bash shell contains no built-in debugger, and only bare-bones debugging-specific commands and constructs. Syntax errors or outright typos in the script generate cryptic error messages that are often of no help in debugging a non-functional script.

![[Example 32-1|Example 32-1]]

What's wrong with the above script? Hint: after the _if_.

![[Example 32-2|Example 32-2]]

Note that the error message does _not_ necessarily reference the line in which the error occurs, but the line where the Bash interpreter finally becomes aware of the error.

Error messages may disregard comment lines in a script when reporting the line number of a syntax error.

What if the script executes, but does not work as expected? This is the all too familiar logic error.

![[Example 32-3|Example 32-3]]

Try to find out what's wrong with [[Example 32-3|Example 32-3]] by uncommenting the **echo "$badname"** line. Echo statements are useful for seeing whether what you expect is actually what you get.

In this particular case, **rm "$badname"** will not give the desired results because $badname should not be quoted. Placing it in quotes ensures that **rm** has only one argument (it will match only one filename). A partial fix is to remove to quotes from $badname and to reset $IFS to contain only a newline, **IFS=$'\n'**. However, there are simpler ways of going about it.

```bash
# Correct methods of deleting filenames containing spaces.
rm *\ *
rm *" "*
rm *' '*
# Thank you. S.C.
```

Summarizing the symptoms of a buggy script,

1. It bombs with a "syntax error" message, or
2. It runs, but does not work as expected (logic error).
3. It runs, works as expected, but has nasty side effects (logic bomb).

Tools for debugging non-working scripts include

1. Inserting [[internal-commands-and-builtins#^ECHOREF|echo]] statements at critical points in the script to trace the variables, and otherwise give a snapshot of what is going on.

> [~tip]
> Even better is an **echo** that echoes only when _debug_ is on.
>
> ```bash
> ### debecho (debug-echo), by Stefano Falsetto ###
> ### Will echo passed parameters only if DEBUG is set to a value. ###
> debecho () {
>   if [ ! -z "$DEBUG" ]; then
>      echo "$1" >&2
>      #         ^^^ to stderr
>   fi
> }
> 
> DEBUG=on
> Whatever=whatnot
> debecho $Whatever   # whatnot
> 
> DEBUG=
> Whatever=notwhat
> debecho $Whatever   # (Will not echo.)
> ```
    
2. Using the [[miscellaneous-commands#^TEEREF|tee]] filter to check processes or data flows at critical points.
    
3. Setting option flags -n -v -x
    
    **sh -n scriptname** checks for syntax errors without actually running the script. This is the equivalent of inserting **set -n** or **set -o noexec** into the script. Note that certain types of syntax errors can slip past this check.
    
    **sh -v scriptname** echoes each command before executing it. This is the equivalent of inserting **set -v** or **set -o verbose** in the script.
    
    The -n and -v flags work well together. **sh -nv scriptname** gives a verbose syntax check.
    
    **sh -x scriptname** echoes the result each command, but in an abbreviated manner. This is the equivalent of inserting **set -x** or **set -o xtrace** in the script.
    
    Inserting **set -u** or **set -o nounset** in the script runs it, but gives an unbound variable error message and aborts the script.
    
```bash
set -u   # Or   set -o nounset

# Setting a variable to null will not trigger the error/abort.
# unset_var=

echo $unset_var   # Unset (and undeclared) variable.

echo "Should not echo!"

# sh t2.sh
# t2.sh: line 6: unset_var: unbound variable
```
    
4. Using an "assert" function to test a variable or condition at critical points in a script. (This is an idea borrowed from C.)
    
![[Example 32-4|Example 32-4]]
    
5. Using the [[another-look-at-variables#^LINENOREF|$LINENO]] variable and the [[internal-commands-and-builtins#^CALLERREF|caller]] builtin.
    
6. Trapping at exit.
    
    The [[internal-commands-and-builtins#^EXITREF|exit]] command in a script triggers a signal 0, terminating the process, that is, the script itself. [^1] It is often useful to trap the _exit_, forcing a "printout" of variables, for example. The _trap_ must be the first command in the script.
    

**Trapping signals**

**trap**

Specifies an action on receipt of a signal; also useful for debugging.

> A _signal_ is a message sent to a process, either by the kernel or another process, telling it to take some specified action (usually to terminate). For example, hitting a [[special-characters#^CTLCREF|Control-C]] sends a user interrupt, an INT signal, to a running program.

_A simple instance:_

```bash
trap '' 2
# Ignore interrupt 2 (Control-C), with no action specified. 

trap 'echo "Control-C disabled."' 2
# Message when Control-C pressed.
```

![[Example 32-5|Example 32-5]]

![[Example 32-6|Example 32-6]]

![[Example 32-7|Example 32-7]]

> [!note]
> The DEBUG argument to **trap** causes a specified action to execute after every command in a script. This permits tracing variables, for example.
> 
> **Example 32-8. Tracing a variable**
> 
> ```bash
> #!/bin/bash
> 
> trap 'echo "VARIABLE-TRACE> \$variable = \"$variable\""' DEBUG
> # Echoes the value of $variable after every command.
> 
> variable=29; line=$LINENO
> 
> echo "  Just initialized \$variable to $variable in line number $line."
> 
> let "variable *= 3"; line=$LINENO
> echo "  Just multiplied \$variable by 3 in line number $line."
> 
> exit 0
> 
> #  The "trap 'command1 . . . command2 . . .' DEBUG" construct is
> #+ more appropriate in the context of a complex script,
> #+ where inserting multiple "echo $variable" statements might be
> #+ awkward and time-consuming.
> 
> # Thanks, Stephane Chazelas for the pointer.
> 
> 
> Output of script:
> 
> VARIABLE-TRACE> $variable = ""
> VARIABLE-TRACE> $variable = "29"
>   Just initialized $variable to 29.
> VARIABLE-TRACE> $variable = "29"
> VARIABLE-TRACE> $variable = "87"
>   Just multiplied $variable by 3.
> VARIABLE-TRACE> $variable = "87"
> ```

Of course, the **trap** command has other uses aside from debugging, such as disabling certain keystrokes within a script (see [[Example A-43|Example A-43]]).

![[Example 32-9|Example 32-9]]

> [!note]
> **trap '' SIGNAL** (two adjacent apostrophes) disables SIGNAL for the remainder of the script. **trap SIGNAL** restores the functioning of SIGNAL once more. This is useful to protect a critical portion of a script from an undesirable interrupt.|

```bash
	trap '' 2  # Signal 2 is Control-C, now disabled.
	command
	command
	command
	trap 2     # Reenables Control-C
	
```

> [[bashver3#^BASH3REF|Version 3]] of Bash adds the following [[another-look-at-variables#^INTERNALVARIABLES1|internal variables]] for use by the debugger.
>
> 1. $BASH_ARGC
>     
>     Number of command-line arguments passed to script, similar to [[another-look-at-variables#^CLACOUNTREF|$#]].
>     
> 2. $BASH_ARGV
>     
>     Final command-line parameter passed to script, equivalent [[othertypesv#^LASTARGREF|${!#}]].
>     
> 3. $BASH_COMMAND
>     
>     Command currently executing.
>     
> 4. $BASH_EXECUTION_STRING
>     
>     The _option string_ following the -c [[bash-options#^CLOPTS|option]] to Bash.
>     
> 5. $BASH_LINENO
>     
>     In a [[functions|function]], indicates the line number of the function call.
>     
> 6. $BASH_REMATCH
>     
>     Array variable associated with **=~** [[bashver3#^REGEXMATCHREF|conditional regex matching]].
>     
> 7. $BASH_SOURCE
>     
>     This is the name of the script, usually the same as [[othertypesv#^ARG0|$0]].
>     
> 8. [[another-look-at-variables#^BASHSUBSHELLREF|$BASH_SUBSHELL]]|

[^1]: By convention, _signal 0_ is assigned to [[exit-and-exit-status|exit]].
