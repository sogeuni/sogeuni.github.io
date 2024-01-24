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

The only times a variable appears "naked" -- without the $ prefix -- is when declared or assigned, when *unset*, when [[../commands/internal-commands-and-builtins#^EXPORTREF|exported]], in an arithmetic expression within [[operations-and-related-topics.html|double parentheses (( ... ))]], or in the special case of a variable representing a [[../advanced-topics/debugging#^SIGNALD|signal]] (see [[../advanced-topics/debugging#^EX76|Example 32-5]]). Assignment may be with an = (as in *var1=27*), in a [[../commands/internal-commands-and-builtins#^READREF|read]] statement, and at the head of a loop (*for var2 in 1 2 3*).

Enclosing a referenced value in *double quotes* (" ... ") does not interfere with variable substitution. This is called *partial quoting*, sometimes referred to as "weak quoting." Using single quotes (' ... ') causes the variable name to be used literally, and no substitution will take place. This is *full quoting*, sometimes referred to as 'strong quoting.' See [[Chapter 5. Quoting|Chapter 5]] for a detailed discussion.

Note that **$variable** is actually a simplified form of **${variable}**. In contexts where the **$variable** syntax causes an error, the longer form may work (see [[parameter-substitution.html|Section 10.2]], below).

###### Example 4-1. Variable assignment and substitution

```bash
#!/bin/bash
# ex9.sh

# Variables: assignment and substitution

a=375
hello=$a
#   ^ ^

#-------------------------------------------------------------------------
# No space permitted on either side of = sign when initializing variables.
# What happens if there is a space?

#  "VARIABLE =value"
#           ^
#% Script tries to run "VARIABLE" command with one argument, "=value".

#  "VARIABLE= value"
#            ^
#% Script tries to run "value" command with
#+ the environmental variable "VARIABLE" set to "".
#-------------------------------------------------------------------------


echo hello    # hello
# Not a variable reference, just the string "hello" ...

echo $hello   # 375
#    ^          This *is* a variable reference.
echo ${hello} # 375
#               Likewise a variable reference, as above.

# Quoting . . .
echo "$hello"    # 375
echo "${hello}"  # 375

echo

hello="A B  C   D"
echo $hello   # A B C D
echo "$hello" # A B  C   D
# As we see, echo $hello   and   echo "$hello"   give different results.
# =======================================
# Quoting a variable preserves whitespace.
# =======================================

echo

echo '$hello'  # $hello
#    ^      ^
#  Variable referencing disabled (escaped) by single quotes,
#+ which causes the "$" to be interpreted literally.

# Notice the effect of different types of quoting.


hello=    # Setting it to a null value.
echo "\$hello (null value) = $hello"      # $hello (null value) =
#  Note that setting a variable to a null value is not the same as
#+ unsetting it, although the end result is the same (see below).

# --------------------------------------------------------------

#  It is permissible to set multiple variables on the same line,
#+ if separated by white space.
#  Caution, this may reduce legibility, and may not be portable.

var1=21  var2=22  var3=$V3
echo
echo "var1=$var1   var2=$var2   var3=$var3"

# May cause problems with legacy versions of "sh" . . .

# --------------------------------------------------------------

echo; echo

numbers="one two three"
#           ^   ^
other_numbers="1 2 3"
#               ^ ^
#  If there is whitespace embedded within a variable,
#+ then quotes are necessary.
#  other_numbers=1 2 3                  # Gives an error message.
echo "numbers = $numbers"
echo "other_numbers = $other_numbers"   # other_numbers = 1 2 3
#  Escaping the whitespace also works.
mixed_bag=2\ ---\ Whatever
#           ^    ^ Space after escape (\).

echo "$mixed_bag"         # 2 --- Whatever

echo; echo

echo "uninitialized_variable = $uninitialized_variable"
# Uninitialized variable has null value (no value at all!).
uninitialized_variable=   #  Declaring, but not initializing it --
                          #+ same as setting it to a null value, as above.
echo "uninitialized_variable = $uninitialized_variable"
                          # It still has a null value.

uninitialized_variable=23       # Set it.
unset uninitialized_variable    # Unset it.
echo "uninitialized_variable = $uninitialized_variable"
                                # uninitialized_variable =
                                # It still has a null value.
echo

exit 0
```

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
> See also [[../commands/internal-commands-and-builtins#^SELFSOURCE|Example 15-23]].

## Variable Assignment

### =

the assignment operator (*no space before and after*)

> [!caution]
> Do not confuse this with [[other-comparison-operators#^EQUALSIGNREF|=]] and [[other-comparison-operators#^EQUALREF|-eq]], which [[./tests#^IFTHEN|test]], rather than assign!
>
> Note that = can be either an *assignment* or a *test* operator, depending on context.

###### Example 4-2. Plain Variable Assignment

```bash
#!/bin/bash
# Naked variables

echo

# When is a variable "naked", i.e., lacking the '$' in front?
# When it is being assigned, rather than referenced.

# Assignment
a=879
echo "The value of \"a\" is $a."

# Assignment using 'let'
let a=16+5
echo "The value of \"a\" is now $a."

echo

# In a 'for' loop (really, a type of disguised assignment):
echo -n "Values of \"a\" in the loop are: "
for a in 7 8 9 11
do
  echo -n "$a "
done

echo
echo

# In a 'read' statement (also a type of assignment):
echo -n "Enter \"a\" "
read a
echo "The value of \"a\" is now $a."

echo

exit 0
```

###### Example 4-3. Variable Assignment, plain and fancy

```bash
#!/bin/bash

a=23              # Simple case
echo $a
b=$a
echo $b

# Now, getting a little bit fancier (command substitution).

a=`echo Hello!`   # Assigns result of 'echo' command to 'a' ...
echo $a
#  Note that including an exclamation mark (!) within a
#+ command substitution construct will not work from the command-line,
#+ since this triggers the Bash "history mechanism."
#  Inside a script, however, the history functions are disabled by default.

a=`ls -l`         # Assigns result of 'ls -l' command to 'a'
echo $a           # Unquoted, however, it removes tabs and newlines.
echo
echo "$a"         # The quoted variable preserves whitespace.
                  # (See the chapter on "Quoting.")

exit 0
```

Variable assignment using the *$(...)* mechanism (a newer method than [[../beyond-the-basic/command-substitution#^BACKQUOTESREF|backquotes]]). This is likewise a form of [[../beyond-the-basic/command-substitution#^COMMANDSUBREF|command substitution]].

```bash
# From /etc/rc.d/rc.local
R=$(cat /etc/redhat-release)
arch=$(uname -m)
```

## Bash Variables Are Untyped

Unlike many other programming languages, Bash does not segregate its variables by "type." Essentially, *Bash variables are character strings*, but, depending on context, Bash permits arithmetic operations and comparisons on variables. The determining factor is whether the value of a variable contains only digits.

###### Example 4-4. Integer or string?

```bash

#!/bin/bash
# int-or-string.sh

a=2334                   # Integer.
let "a += 1"
echo "a = $a "           # a = 2335
echo                     # Integer, still.


b=${a/23/BB}             # Substitute "BB" for "23".
                         # This transforms $b into a string.
echo "b = $b"            # b = BB35
declare -i b             # Declaring it an integer doesn't help.
echo "b = $b"            # b = BB35

let "b += 1"             # BB35 + 1
echo "b = $b"            # b = 1
echo                     # Bash sets the "integer value" of a string to 0.

c=BB34
echo "c = $c"            # c = BB34
d=${c/BB/23}             # Substitute "23" for "BB".
                         # This makes $d an integer.
echo "d = $d"            # d = 2334
let "d += 1"             # 2334 + 1
echo "d = $d"            # d = 2335
echo


# What about null variables?
e=''                     # ... Or e="" ... Or e=
echo "e = $e"            # e =
let "e += 1"             # Arithmetic operations allowed on a null variable?
echo "e = $e"            # e = 1
echo                     # Null variable transformed into an integer.

# What about undeclared variables?
echo "f = $f"            # f =
let "f += 1"             # Arithmetic operations allowed?
echo "f = $f"            # f = 1
echo                     # Undeclared variable transformed into an integer.
#
# However ...
let "f /= $undecl_var"   # Divide by zero?
#   let: f /= : syntax error: operand expected (error token is " ")
# Syntax error! Variable $undecl_var is not set to zero here!
#
# But still ...
let "f /= 0"
#   let: f /= 0: division by 0 (error token is "0")
# Expected behavior.


#  Bash (usually) sets the "integer value" of null to zero
#+ when performing an arithmetic operation.
#  But, don't try this at home, folks!
#  It's undocumented and probably non-portable behavior.


# Conclusion: Variables in Bash are untyped,
#+ with all attendant consequences.

exit $?
```

Untyped variables are both a blessing and a curse. They permit more flexibility in scripting and make it easier to grind out lines of code (and give you enough rope to hang yourself!). However, they likewise permit subtle errors to creep in and encourage sloppy programming habits.

To lighten the burden of keeping track of variable types in a script, Bash *does* permit [[typing-variables.html|declaring]] variables.

## Special Variable Types

### *Local variables*

Variables [[../advanced-topics/subshells#^SCOPEREF|visible]] only within a [[./special-characters#^CODEBLOCKREF|code block]] or function (see also [[../advanced-topics/local-variables#^LOCALREF|local variables]] in [[../advanced-topics/functions|functions]])

### *Environmental variables*

Variables that affect the behavior of the shell and user interface

> [!note]
> In a more general context, each [[./special-characters#^PROCESSREF|process]] has an "environment", that is, a group of variables that the process may reference. In this sense, the shell behaves like any other process.
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

If a script sets environmental variables, they need to be "exported," that is, reported to the *environment* local to the script. This is the function of the [[../commands/internal-commands-and-builtins#^EXPORTREF|export]] command.

> [!note]
> A script can **export** variables only to child [[./special-characters#^PROCESSREF|processes]], that is, only to commands or processes which that particular script initiates. A script invoked from the command-line *cannot* export variables back to the command-line environment. *[[../commands/internal-commands-and-builtins#^FORKREF|Child processes]] cannot export variables back to the parent processes that spawned them.*
>
> **Definition:** A *child process* is a subprocess launched by another process, its [[../commands/internal-commands-and-builtins#^PARENTREF|parent]].|

### *Positional parameters*

Arguments passed to the script from the command line [^2] : $0, $1, $2, $3 . . .

$0 is the name of the script itself, $1 is the first argument, $2 the second, $3 the third, and so forth. [^3] After $9, the arguments must be enclosed in brackets, for example, ${10}, ${11}, ${12}.

The special variables [[../beyond-the-basic/another-look-at-variables#^APPREF|$* and $@]] denote *all* the positional parameters.

###### Example 4-5. Positional Parameters

```bash
#!/bin/bash

# Call this script with at least 10 parameters, for example
# ./scriptname 1 2 3 4 5 6 7 8 9 10
MINPARAMS=10

echo

echo "The name of this script is \"$0\"."
# Adds ./ for current directory
echo "The name of this script is \"`basename $0`\"."
# Strips out path name info (see 'basename')

echo

if [ -n "$1" ]              # Tested variable is quoted.
then
 echo "Parameter #1 is $1"  # Need quotes to escape #
fi 

if [ -n "$2" ]
then
 echo "Parameter #2 is $2"
fi 

if [ -n "$3" ]
then
 echo "Parameter #3 is $3"
fi 

# ...


if [ -n "${10}" ]  # Parameters > $9 must be enclosed in {brackets}.
then
 echo "Parameter #10 is ${10}"
fi 

echo "-----------------------------------"
echo "All the command-line parameters are: "$*""

if [ $# -lt "$MINPARAMS" ]
then
  echo
  echo "This script needs at least $MINPARAMS command-line arguments!"
fi  

echo

exit 0
```

*Bracket notation* for positional parameters leads to a fairly simple way of referencing the *last* argument passed to a script on the command-line. This also requires [[bashver2#^VARREFNEW|indirect referencing]].

```bash
args=$#           # Number of args passed.
lastarg=${!args}
# Note: This is an *indirect reference* to $args ...


# Or:       lastarg=${!#}             (Thanks, Chris Monson.)
# This is an *indirect reference* to the $# variable.
# Note that lastarg=${!$#} doesn't work.
```

Some scripts can perform different operations, depending on which name they are invoked with. For this to work, the script needs to check $0, the name it was invoked by. [^4] There must also exist symbolic links to all the alternate names of the script. See [[../commands/basic-commands#^HELLOL|Example 16-2]].

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

###### Example 4-6. *wh*, *whois* domain name lookup

```bash
#!/bin/bash
# ex18.sh

# Does a 'whois domain-name' lookup on any of 3 alternate servers:
#                    ripe.net, cw.net, radb.net

# Place this script -- renamed 'wh' -- in /usr/local/bin

# Requires symbolic links:
# ln -s /usr/local/bin/wh /usr/local/bin/wh-ripe
# ln -s /usr/local/bin/wh /usr/local/bin/wh-apnic
# ln -s /usr/local/bin/wh /usr/local/bin/wh-tucows

E_NOARGS=75


if [ -z "$1" ]
then
  echo "Usage: `basename $0` [domain-name]"
  exit $E_NOARGS
fi

# Check script name and call proper server.
case `basename $0` in    # Or:    case ${0##*/} in
    "wh"       ) whois $1@whois.tucows.com;;
    "wh-ripe"  ) whois $1@whois.ripe.net;;
    "wh-apnic" ) whois $1@whois.apnic.net;;
    "wh-cw"    ) whois $1@whois.cw.net;;
    *          ) echo "Usage: `basename $0` [domain-name]";;
esac 

exit $?
```

---

The **shift** command reassigns the positional parameters, in effect shifting them to the left one notch.

$1 <--- $2, $2 <--- $3, $3 <--- $4, etc.

The old $1 disappears, but *$0 (the script name) does not change*. If you use a large number of positional parameters to a script, **shift** lets you access those past 10, although [[othertypesv#^BRACKETNOTATION|{bracket} notation]] also permits this.

###### Example 4-7. Using *shift*

```bash
#!/bin/bash
# shft.sh: Using 'shift' to step through all the positional parameters.

#  Name this script something like shft.sh,
#+ and invoke it with some parameters.
#+ For example:
#             sh shft.sh a b c def 83 barndoor

until [ -z "$1" ]  # Until all parameters used up . . .
do
  echo -n "$1 "
  shift
done

echo               # Extra linefeed.

# But, what happens to the "used-up" parameters?
echo "$2"
#  Nothing echoes!
#  When $2 shifts into $1 (and there is no $3 to shift into $2)
#+ then $2 remains empty.
#  So, it is not a parameter *copy*, but a *move*.

exit

#  See also the echo-params.sh script for a "shiftless"
#+ alternative method of stepping through the positional params.
```

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

> [!note] The **shift** command works in a similar fashion on parameters passed to a [[../advanced-topics/functions|function]]. See [[assortedtips#^MULTIPLICATION|Example 36-18]].

[^1]: Technically, the *name* of a variable is called an *lvalue*, meaning that it appears on the *left* side of an assignment statment, as in **VARIABLE=23**. A variable's *value* is an *rvalue*, meaning that it appears on the *right* side of an assignment statement, as in **VAR2=$VARIABLE**.

    A variable's *name* is, in fact, a *reference*, a *pointer* to the memory location(s) where the actual data associated with that variable is kept.

[^2]: Note that [[../advanced-topics/complex-functions-and-function-complexities#^PASSEDARGS|*functions* also take positional parameters]].

[^3]: The process calling the script sets the $0 parameter. By convention, this parameter is the name of the script. See the [[../commands/basic-commands#^MANREF|manpage]] (manual page) for **execv**.

    From the *command-line*, however, $0 is the name of the shell.

    ```bash
    bash$ echo $0
    bash

    tcsh% echo $0
    tcsh
    ```

[^4]: If the the script is [[../commands/internal-commands-and-builtins#^SOURCEREF|sourced]] or [[../commands/basic-commands#^SYMLINKREF|symlinked]], then this will not work. It is safer to check [[../advanced-topics/debugging#^BASHSOURCEREF|$BASH_Source]].
