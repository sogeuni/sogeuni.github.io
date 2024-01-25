---
title: 4. Introduction to Variables and Parameters
---

*Variables* are how programming and scripting languages represent data. A variable is nothing more than a *label*, a name assigned to a location or set of locations in computer memory holding an item of data.

Variables appear in arithmetic operations and manipulation of quantities, and in string parsing.

## Variable Substitution

The *name* of a variable is a placeholder for its *value*, the data it holds. Referencing (retrieving) its value is called *variable substitution*.

### \$

Let us carefully distinguish between the *name* of a variable and its *value*. If **variable1** is the name of a variable, then **$variable1** is a reference to its *value*, the data item it contains. [^1]

```bash
bash$ variable1=23


bash$ echo variable1
variable1

bash$ echo $variable1
23
```

The only times a variable appears "naked" -- without the $ prefix -- is when declared or assigned, when *unset*, when [[internal-commands-and-builtins#^EXPORTREF|exported]], in an arithmetic expression within [[operations-and-related-topics.html|double parentheses (( ... ))]], or in the special case of a variable representing a [[debugging#^SIGNALD|signal]] (see [[debugging#^EX76|Example 32-5]]). Assignment may be with an = (as in *var1=27*), in a [[internal-commands-and-builtins#^READREF|read]] statement, and at the head of a loop (*for var2 in 1 2 3*).

Enclosing a referenced value in *double quotes* (" ... ") does not interfere with variable substitution. This is called *partial quoting*, sometimes referred to as "weak quoting." Using single quotes (' ... ') causes the variable name to be used literally, and no substitution will take place. This is *full quoting*, sometimes referred to as 'strong quoting.' See [[Chapter 5. Quoting|Chapter 5]] for a detailed discussion.

Note that **$variable** is actually a simplified form of **${variable}**. In contexts where the **$variable** syntax causes an error, the longer form may work (see [[parameter-substitution.html|Section 10.2]], below).

![[example 4-1|example 4-1]]

> [!caution]
> An uninitialized variable has a "null" value -- no assigned value at all (*not* zero!).
>
> ```bash
> if [ -z "$unassigned" ]
> then
>   echo "\$unassigned is NULL."
> fi     # $unassigned is NULL.
> ```
>
> Using a variable before assigning a value to it may cause problems. It is nevertheless possible to perform arithmetic operations on an uninitialized variable.
>
> ```bash
> echo "$uninitialized"                                # (blank line)
> let "uninitialized += 5"                             # Add 5 to it.
> echo "$uninitialized"                                # 5
> 
> #  Conclusion:
> #  An uninitialized variable has no value,
> #+ however it evaluates as 0 in an arithmetic operation.
> ```
>
> See also [[internal-commands-and-builtins#^SELFSOURCE|Example 15-23]].

## Variable Assignment

### =

the assignment operator (*no space before and after*)

> [!caution]
> Do not confuse this with [[other-comparison-operators#^EQUALSIGNREF|=]] and [[other-comparison-operators#^EQUALREF|-eq]], which [[tests#^IFTHEN|test]], rather than assign!
>
> Note that = can be either an *assignment* or a *test* operator, depending on context.

![[example 4-2|example 4-2]]

![[example 4-3|example 4-3]]

Variable assignment using the *$(...)* mechanism (a newer method than [[command-substitution#^BACKQUOTESREF|backquotes]]). This is likewise a form of [[command-substitution#^COMMANDSUBREF|command substitution]].

```bash
# From /etc/rc.d/rc.local
R=$(cat /etc/redhat-release)
arch=$(uname -m)
```

## Bash Variables Are Untyped

Unlike many other programming languages, Bash does not segregate its variables by "type." Essentially, *Bash variables are character strings*, but, depending on context, Bash permits arithmetic operations and comparisons on variables. The determining factor is whether the value of a variable contains only digits.

![[example 4-4|example 4-4]]

Untyped variables are both a blessing and a curse. They permit more flexibility in scripting and make it easier to grind out lines of code (and give you enough rope to hang yourself!). However, they likewise permit subtle errors to creep in and encourage sloppy programming habits.

To lighten the burden of keeping track of variable types in a script, Bash *does* permit [[typing-variables.html|declaring]] variables.

## Special Variable Types

### *Local variables*

Variables [[subshells#^SCOPEREF|visible]] only within a [[special-characters#^CODEBLOCKREF|code block]] or function (see also [[local-variables#^LOCALREF|local variables]] in [[functions|functions]])

### *Environmental variables*

Variables that affect the behavior of the shell and user interface

> [!note]
> In a more general context, each [[special-characters#^PROCESSREF|process]] has an "environment", that is, a group of variables that the process may reference. In this sense, the shell behaves like any other process.
> 
> Every time a shell starts, it creates shell variables that correspond to its own environmental variables. Updating or adding new environmental variables causes the shell to update its environment, and all the shell's *child processes* (the commands it executes) inherit this environment.

> [!caution]
> The space allotted to the environment is limited. Creating too many environmental variables or ones that use up excessive space may cause problems.
>
> ```bash
> bash$ eval "`seq 10000 | sed -e 's/.*/export var&=ZZZZZZZZZZZZZZ/'`"
> 
> bash$ du
> bash: /usr/bin/du: Argument list too long
> ```
>
> Note: this "error" has been fixed, as of kernel version 2.6.23.
>
> (Thank you, Stéphane Chazelas for the clarification, and for providing the above example.)

If a script sets environmental variables, they need to be "exported," that is, reported to the *environment* local to the script. This is the function of the [[internal-commands-and-builtins#^EXPORTREF|export]] command.

> [!note]
> A script can **export** variables only to child [[special-characters#^PROCESSREF|processes]], that is, only to commands or processes which that particular script initiates. A script invoked from the command-line *cannot* export variables back to the command-line environment. *[[internal-commands-and-builtins#^FORKREF|Child processes]] cannot export variables back to the parent processes that spawned them.*
>
> **Definition:** A *child process* is a subprocess launched by another process, its [[internal-commands-and-builtins#^PARENTREF|parent]].|

### *Positional parameters*

Arguments passed to the script from the command line [^2] : $0, $1, $2, $3 . . .

$0 is the name of the script itself, $1 is the first argument, $2 the second, $3 the third, and so forth. [^3] After $9, the arguments must be enclosed in brackets, for example, ${10}, ${11}, ${12}.

The special variables [[another-look-at-variables#^APPREF|$* and $@]] denote *all* the positional parameters.

![[example 4-5|example 4-5]]

*Bracket notation* for positional parameters leads to a fairly simple way of referencing the *last* argument passed to a script on the command-line. This also requires [[bashver2#^VARREFNEW|indirect referencing]].

```bash
args=$#           # Number of args passed.
lastarg=${!args}
# Note: This is an *indirect reference* to $args ...


# Or:       lastarg=${!#}             (Thanks, Chris Monson.)
# This is an *indirect reference* to the $# variable.
# Note that lastarg=${!$#} doesn't work.
```

Some scripts can perform different operations, depending on which name they are invoked with. For this to work, the script needs to check $0, the name it was invoked by. [^4] There must also exist symbolic links to all the alternate names of the script. See [[basic-commands#^HELLOL|Example 16-2]].

> [!tip]
> If a script expects a command-line parameter but is invoked without one, this may cause a *null variable assignment*, generally an undesirable result. One way to prevent this is to append an extra character to both sides of the assignment statement using the expected positional parameter.

```bash
variable1_=$1_  # Rather than variable1=$1
# This will prevent an error, even if positional parameter is absent.

critical_argument01=$variable1_

# The extra character can be stripped off later, like so.
variable1=${variable1_/_/}
# Side effects only if $variable1_ begins with an underscore.
# This uses one of the parameter substitution templates discussed later.
# (Leaving out the replacement pattern results in a deletion.)

#  A more straightforward way of dealing with this is
#+ to simply test whether expected positional parameters have been passed.
if [ -z $1 ]
then
  exit $E_MISSING_POS_PARAM
fi


#  However, as Fabian Kreutz points out,
#+ the above method may have unexpected side-effects.
#  A better method is parameter substitution:
#         ${1:-$DefaultVal}
#  See the "Parameter Substition" section
#+ in the "Variables Revisited" chapter.
```

---

![[example 4-6|example 4-6]]

---

The **shift** command reassigns the positional parameters, in effect shifting them to the left one notch.

$1 <--- $2, $2 <--- $3, $3 <--- $4, etc.

The old $1 disappears, but *$0 (the script name) does not change*. If you use a large number of positional parameters to a script, **shift** lets you access those past 10, although [[othertypesv#^BRACKETNOTATION|{bracket} notation]] also permits this.

![[example 4-7|example 4-7]]

The **shift** command can take a numerical parameter indicating how many positions to shift.

```bash
#!/bin/bash
# shift-past.sh

shift 3    # Shift 3 positions.
#  n=3; shift $n
#  Has the same effect.

echo "$1"

exit 0

# ======================== #


$ sh shift-past.sh 1 2 3 4 5
4

#  However, as Eleni Fragkiadaki, points out,
#+ attempting a 'shift' past the number of
#+ positional parameters ($#) returns an exit status of 1,
#+ and the positional parameters themselves do not change.
#  This means possibly getting stuck in an endless loop. . . .
#  For example:
#      until [ -z "$1" ]
#      do
#         echo -n "$1 "
#         shift 20    #  If less than 20 pos params,
#      done           #+ then loop never ends!
#
# When in doubt, add a sanity check. . . .
#           shift 20 || break
#                    ^^^^^^^^
```

> [!note] The **shift** command works in a similar fashion on parameters passed to a [[functions|function]]. See [[assortedtips#^MULTIPLICATION|Example 36-18]].

[^1]: Technically, the *name* of a variable is called an *lvalue*, meaning that it appears on the *left* side of an assignment statment, as in **VARIABLE=23**. A variable's *value* is an *rvalue*, meaning that it appears on the *right* side of an assignment statement, as in **VAR2=$VARIABLE**.

    A variable's *name* is, in fact, a *reference*, a *pointer* to the memory location(s) where the actual data associated with that variable is kept.

[^2]: Note that [[complex-functions-and-function-complexities#^PASSEDARGS|*functions* also take positional parameters]].

[^3]: The process calling the script sets the $0 parameter. By convention, this parameter is the name of the script. See the [[basic-commands#^MANREF|manpage]] (manual page) for **execv**.

    From the *command-line*, however, $0 is the name of the shell.

    ```bash
    bash$ echo $0
    bash

    tcsh% echo $0
    tcsh
    ```

[^4]: If the the script is [[internal-commands-and-builtins#^SOURCEREF|sourced]] or [[basic-commands#^SYMLINKREF|symlinked]], then this will not work. It is safer to check [[debugging#^BASHSOURCEREF|$BASH_Source]].
