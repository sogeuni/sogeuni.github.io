---
title: 7. Tests
---


Every reasonably complete programming language can test for a condition, then act according to the result of the test. Bash has the [[tests#^TTESTREF|test]] command, various [[tests#^DBLBRACKETS|bracket]] and [[tests#^DBLPARENSTST|parenthesis]] operators, and the **if/then** construct.

## Test Constructs

- An **if/then** construct tests whether the [[exit-and-exit-status#^EXITSTATUSREF|exit status]] of a list of commands is 0 (since 0 means "success" by UNIX convention), and if so, executes one or more commands.
- There exists a dedicated command called **[[special-characters#^LEFTBRACKET|** ([left bracket]] special character). It is a synonym for **test**, and a [[internal-commands-and-builtins|builtin]] for efficiency reasons. This command considers its arguments as comparison expressions or file tests and returns an exit status corresponding to the result of the comparison (0 for true, 1 for false).
- With version 2.02, Bash introduced the [[tests#^DBLBRACKETS|[[ ... | ... ]]]] _extended test command_, which performs comparisons in a manner more familiar to programmers from other languages. Note that **[[internal-commands-and-builtins#^keywordref|[** is a [keyword]], not a command.
    Bash sees **[[ $a -lt $b | $a -lt $b ]]** as a single element, which returns an exit status.
- The [[operations-and-related-topics.html|(( ... ))]] and [[internal-commands-and-builtins#^LETREF|let ...]] constructs return an [[exit-and-exit-status#^EXITSTATUSREF|exit status]], _according to whether the arithmetic expressions they evaluate expand to a non-zero value_. These [[arithmetic-expansion#^ARITHEXPREF|arithmetic-expansion]] constructs may therefore be used to perform [[other-comparison-operators#^ICOMPARISON1|arithmetic comparisons]].

```bash
(( 0 && 1 ))                 # Logical AND
echo $?     # 1     ***
# And so ...
let "num = (( 0 && 1 ))"
echo $num   # 0
# But ...
let "num = (( 0 && 1 ))"
echo $?     # 1     ***


(( 200 || 11 ))              # Logical OR
echo $?     # 0     ***
# ...
let "num = (( 200 || 11 ))"
echo $num   # 1
let "num = (( 200 || 11 ))"
echo $?     # 0     ***


(( 200 | 11 ))               # Bitwise OR
echo $?                      # 0     ***
# ...
let "num = (( 200 | 11 ))"
echo $num                    # 203
let "num = (( 200 | 11 ))"
echo $?                      # 0     ***

# The "let" construct returns the same exit status
#+ as the double-parentheses arithmetic expansion.
```

> [!caution]
> Again, note that the _exit status_ of an arithmetic expression is _not_ an error value.
>
> ```bash
> var=-2 && (( var+=2 ))
> echo $?                   # 1
> 
> var=-2 && (( var+=2 )) && echo $var
>                           # Will not echo $var!
> ```

- An **if** can test any command, not just conditions enclosed within brackets.

```bash
if cmp a b &> /dev/null  # Suppress output.
then echo "Files a and b are identical."
else echo "Files a and b differ."
fi

# The very useful "if-grep" construct:
# ----------------------------------- 
if grep -q Bash file
  then echo "File contains at least one occurrence of Bash."
fi

word=Linux
letter_sequence=inu
if echo "$word" | grep -q "$letter_sequence"
# The "-q" option to grep suppresses output.
then
  echo "$letter_sequence found in $word"
else
  echo "$letter_sequence not found in $word"
fi


if COMMAND_WHOSE_EXIT_STATUS_IS_0_UNLESS_ERROR_OCCURRED
  then echo "Command succeeded."
  else echo "Command failed."
fi
```

- _These last two examples courtesy of Stéphane Chazelas._

![[Example 7-1|Example 7-1]]

**Exercise.** Explain the behavior of [[Example 7-1|Example 7-1]], above.

```bash
if [ condition-true ]
then
   command 1
   command 2
   ...
else  # Or else ...
      # Adds default code block executing if original condition tests false.
   command 3
   command 4
   ...
fi
```

> [!note] 
> When _if_ and _then_ are on same line in a condition test, a semicolon must terminate the _if_ statement. Both _if_ and _then_ are [[internal-commands-and-builtins#^keywordref|keywords]]. Keywords (or commands) begin statements, and before a new statement on the same line begins, the old one must terminate.
>
> ```bash
> if [ -x "$filename" ]; then
> ```

**Else if and elif**

elif

**elif** is a contraction for _else if_. The effect is to nest an inner if/then construct within an outer one.

```bash
if [ condition1 ]
then
   command1
   command2
   command3
elif [ condition2 ]
# Same as else if
then
   command4
   command5
else
   default-command
fi
```

The **if test condition-true** construct is the exact equivalent of **if [ condition-true ]**. As it happens, the left bracket, **[** , is a _token_ [^1] which invokes the **test** command. The closing right bracket, **]** , in an if/test should not therefore be strictly necessary, however newer versions of Bash require it.

> [!note] 
> The **test** command is a Bash [[internal-commands-and-builtins|builtin]] which tests file types and compares strings. Therefore, in a Bash script, **test** does _not_ call the external /usr/bin/test binary, which is part of the _sh-utils_ package. Likewise, **[** does not call /usr/bin/[, which is linked to /usr/bin/test.
>
> ```bash
> bash$ type test
> test is a shell builtin
> bash$ type '['
> [ is a shell builtin
> bash$ type '[['
> [[ is a shell keyword
> bash$ type ']]'
> ]] is a shell keyword
> bash$ type ']'
> bash: type: ]: not found
> 	      
> ```
>
> If, for some reason, you wish to use /usr/bin/test in a Bash script, then specify it by full pathname.

![[Example 7-2|Example 7-2]]

> The `[[]]` construct is the more versatile Bash version of `[ ]`. This is the _extended test command_, adopted from _ksh88_.
>
> No filename expansion or word splitting takes place between [[ and | and ]], but there is parameter expansion and command substitution.
>
> ```bash
> file=/etc/passwd
> 
> if [[ -e $file ]]
> then
>   echo "Password file exists."
> fi
> ```
>
> Using the **[[ ... | ... ]]** test construct, rather than **[ ... ]** can prevent many logic errors in scripts. For example, the &&, \|, <, and > operators work within a [[|7. Tests]] test, despite giving an error within a [ ] construct.
>
> _Arithmetic evaluation_ of octal / hexadecimal constants takes place automatically within a [[ ... | ... ]] construct.
>
> ```bash
> # [[ Octal and hexadecimal evaluation ]]
> # Thank you, Moritz Gronbach, for pointing this out.
> 
> 
> decimal=15
> octal=017   # = 15 (decimal)
> hex=0x0f    # = 15 (decimal)
> 
> if [ "$decimal" -eq "$octal" ]
> then
>   echo "$decimal equals $octal"
> else
>   echo "$decimal is not equal to $octal"       # 15 is not equal to 017
> fi      # Doesn't evaluate within [ single brackets ]!
> 
> 
> if [[ "$decimal" -eq "$octal" ]]
> then
>   echo "$decimal equals $octal"                # 15 equals 017
> else
>   echo "$decimal is not equal to $octal"
> fi      # Evaluates within [[ double brackets ]]!
> 
> if [[ "$decimal" -eq "$hex" ]]
> then
>   echo "$decimal equals $hex"                  # 15 equals 0x0f
> else
>   echo "$decimal is not equal to $hex"
> fi      # [[ $hexadecimal ]] also evaluates!
> ```

> [!note]
> Following an **if**, neither the **test** command nor the test brackets ( [ ] or [[|7. Tests]] ) are strictly necessary.
>
> ```bash
> dir=/home/bozo
> 
> if cd "$dir" 2>/dev/null; then   # "2>/dev/null" hides error message.
>   echo "Now in $dir."
> else
>   echo "Can't change to $dir."
> fi
> ```
>
> The "if COMMAND" construct returns the exit status of COMMAND.
>
> Similarly, a condition within test brackets may stand alone without an **if**, when used in combination with a [[list-constructs#^LISTCONSREF|list construct]].
>
> ```bash
> var1=20
> var2=22
> [ "$var1" -ne "$var2" ] && echo "$var1 is not equal to $var2"
> 
> home=/home/bozo
> [ -d "$home" ] || echo "$home directory does not exist."
> ```

The [[operations-and-related-topics|(( )) construct]] expands and evaluates an arithmetic expression. If the expression evaluates as zero, it returns an [[exit-and-exit-status#^EXITSTATUSREF|exit status]] of 1, or "false". A non-zero expression returns an exit status of 0, or "true". This is in marked contrast to using the **test** and [ ] constructs previously discussed.

![[Example 7-3|Example 7-3]]

## File test operators

**Returns true if...**

-e

file exists

-a

file exists

This is identical in effect to -e. It has been "deprecated," [^2] and its use is discouraged.

-f

file is a _regular_ file (not a directory or [[dev#^DEVFILEREF|device file]])

-s

file is not zero size

-d

file is a directory

-b

file is a [[dev#^BLOCKDEVREF|block device]]

-c

file is a [[dev#^CHARDEVREF|character device]]

```bash
device0="/dev/sda2"    # /   (root directory)
if [ -b "$device0" ]
then
  echo "$device0 is a block device."
fi

# /dev/sda2 is a block device.



device1="/dev/ttyS1"   # PCMCIA modem card.
if [ -c "$device1" ]
then
  echo "$device1 is a character device."
fi

# /dev/ttyS1 is a character device.
```

-p

file is a [[special-characters#^PIPEREF|pipe]]

```bash
function show_input_type()
{
   [ -p /dev/fd/0 ] && echo PIPE || echo STDIN
}

show_input_type "Input"                           # STDIN
echo "Input" | show_input_type                    # PIPE

# This example courtesy of Carl Anderson.
```

-h

file is a [[basic-commands#^SYMLINKREF|symbolic link]]

-L

file is a symbolic link

-S

file is a [[dev#^SOCKETREF|socket]]

-t

file ([[io-redirection#^FDREF|descriptor]]) is associated with a terminal device

This test option [[interactive-and-non-interactive-shell-and-scripts#^II2TEST|may be used to check]] whether the stdin **[ -t 0 ]** or stdout **[ -t 1 ]** in a given script is a terminal.

-r

file has read permission (_for the user running the test_)

-w

file has write permission (for the user running the test)

-x

file has execute permission (for the user running the test)

-g

set-group-id (sgid) flag set on file or directory

If a directory has the _sgid_ flag set, then a file created within that directory belongs to the group that owns the directory, not necessarily to the group of the user who created the file. This may be useful for a directory shared by a workgroup.

-u

set-user-id (suid) flag set on file

A binary owned by _root_ with _set-user-id_ flag set runs with _root_ privileges, even when an ordinary user invokes it. [^3] This is useful for executables (such as **pppd** and **cdrecord**) that need to access system hardware. Lacking the _suid_ flag, these binaries could not be invoked by a _non-root_ user.

```bash
	      -rwsr-xr-t    1 root       178236 Oct  2  2000 /usr/sbin/pppd
	      
```

A file with the _suid_ flag set shows an _s_ in its permissions.

-k

_sticky bit_ set

Commonly known as the _sticky bit,_ the _save-text-mode_ flag is a special type of file permission. If a file has this flag set, that file will be kept in cache memory, for quicker access. [^4] If set on a directory, it restricts write permission. Setting the sticky bit adds a _t_ to the permissions on the file or directory listing. This restricts altering or deleting specific files in that directory to the owner of those files.

```bash
	      drwxrwxrwt    7 root         1024 May 19 21:26 tmp/
	      
```

If a user does not own a directory that has the sticky bit set, but has write permission in that directory, she can only delete those files that she owns in it. This keeps users from inadvertently overwriting or deleting each other's files in a publicly accessible directory, such as /tmp. (The _owner_ of the directory or _root_ can, of course, delete or rename files there.)

-O

you are owner of file

-G

group-id of file same as yours

-N

file modified since it was last read

f1 -nt f2

file _f1_ is newer than _f2_

f1 -ot f2

file _f1_ is older than _f2_

f1 -ef f2

files _f1_ and _f2_ are hard links to the same file

!

"not" -- reverses the sense of the tests above (returns true if condition absent).

![[Example 7-4|Example 7-4]]

[[Example 31-1|Example 31-1]], [[Example 11-8|Example 11-8]], [[Example 11-3|Example 11-3]], [[Example 31-3|Example 31-3]], and [[Example A-1|Example A-1]] also illustrate uses of the file test operators.

## Other Comparison Operators

A _binary_ comparison operator compares two variables or quantities. _Note that integer and string comparison use a different set of operators._

**integer comparison**

-eq

is equal to

**if [ "$a" -eq "$b" ]**

-ne

is not equal to

**if [ "$a" -ne "$b" ]**

-gt

is greater than

**if [ "$a" -gt "$b" ]**

-ge

is greater than or equal to

**if [ "$a" -ge "$b" ]**

-lt

is less than

**if [ "$a" -lt "$b" ]**

-le

is less than or equal to

**if [ "$a" -le "$b" ]**

<

is less than (within [[operations-and-related-topics.html|double parentheses]])

**(("$a" < "$b"))**

<=

is less than or equal to (within double parentheses)

**(("$a" <= "$b"))**

>

is greater than (within double parentheses)

**(("$a" > "$b"))**

>=

is greater than or equal to (within double parentheses)

**(("$a" >= "$b"))**

**string comparison**

=

is equal to

**if [ "$a" = "$b" ]**

> [!caution]
> Note the [[special-characters#Whitespace|whitespace]] framing the **=**.
>
> **if [ "$a"="$b" ]** is _not_ equivalent to the above.

==

is equal to

**if [ "$a" == "$b" ]**

This is a synonym for =.

> [!note]
> The == comparison operator behaves differently within a [[tests#^DBLBRACKETS|double-brackets]] test than within single brackets.
>
> ```bash
> [[ $a == z* ]]   # True if $a starts with an "z" (pattern matching).
> [[ $a == "z*" ]] # True if $a is equal to z* (literal matching).
> 
> [ $a == z* ]     # File globbing and word splitting take place.
> [ "$a" == "z*" ] # True if $a is equal to z* (literal matching).
> 
> # Thanks, Stéphane Chazelas
> ```

!=

is not equal to

**if [ "$a" != "$b" ]**

This operator uses pattern matching within a [[tests#^DBLBRACKETS|[[ ... | ... ]]]] construct.

<

is less than, in [[special-characters#^ASCIIDEF|ASCII]] alphabetical order

**if [[ "$a" < "$b" | "$a" < "$b" ]]**

**if [ "$a" \< "$b" ]**

Note that the "<" needs to be [[quoting#^ESCP|escaped]] within a **[ ]** construct.

>

is greater than, in ASCII alphabetical order

**if [[ "$a" > "$b" | "$a" > "$b" ]]**

**if [ "$a" \> "$b" ]**

Note that the ">" needs to be escaped within a **[ ]** construct.

See [[Example 27-11|Example 27-11]] for an application of this comparison operator.

-z

string is _null_, that is, has zero length

```bash
 String=''   # Zero-length ("null") string variable.

if [ -z "$String" ]
then
  echo "\$String is null."
else
  echo "\$String is NOT null."
fi     # $String is null.
```

-n

string is not _null._

> [!caution] The **-n** test requires that the string be quoted within the test brackets. Using an unquoted string with _! -z_, or even just the unquoted string alone within test brackets (see [[Example 7-6|Example 7-6]]) normally works, however, this is an unsafe practice. _Always_ quote a tested string. [^5]

![[Example 7-5|Example 7-5]]

![[Example 7-6|Example 7-6]]

![[Example 7-7|Example 7-7]]

**compound comparison**

-a

logical and

_exp1 -a exp2_ returns true if _both_ exp1 and exp2 are true.

-o

logical or

_exp1 -o exp2_ returns true if either exp1 _or_ exp2 is true.

These are similar to the Bash comparison operators **&&** and **||**, used within [[test-constructs.md#^DBLBRACKETS|double brackets]].

```bash
[[ condition1 && condition2 ]]
```

The **-o** and **-a** operators work with the [[test-constructs.md#^TTESTREF|test]] command or occur within single test brackets.

```bash
if [ "$expr1" -a "$expr2" ]
then
  echo "Both expr1 and expr2 are true."
else
  echo "Either expr1 or expr2 is false."
fi
```

> [!caution]
> But, as _rihad_ points out:
>
> ```bash
> [ 1 -eq 1 ] && [ -n "`echo true 1>&2`" ]   # true
> [ 1 -eq 2 ] && [ -n "`echo true 1>&2`" ]   # (no output)
> # ^^^^^^^ False condition. So far, everything as expected.
> 
> # However ...
> [ 1 -eq 2 -a -n "`echo true 1>&2`" ]       # true
> # ^^^^^^^ False condition. So, why "true" output?
> 
> # Is it because both condition clauses within brackets evaluate?
> [[ 1 -eq 2 && -n "`echo true 1>&2`" ]]     # (no output)
> # No, that's not it.
> 
> # Apparently && and || "short-circuit" while -a and -o do not.
> ```

Refer to [[Example 8-3|Example 8-3]], [[Example 27-17|Example 27-17]], and [[Example A-29|Example A-29]] to see compound comparison operators in action.

## Nested _if/then_ Condition Tests

Condition tests using the _if/then_ construct may be nested. The net result is equivalent to using the [[operations-and-related-topics#LOGOPS1|_&&_]] compound comparison operator.

```bash
a=3

if [ "$a" -gt 0 ]
then
  if [ "$a" -lt 5 ]
  then
    echo "The value of \"a\" lies somewhere between 0 and 5."
  fi
fi

# Same result as:

if [ "$a" -gt 0 ] && [ "$a" -lt 5 ]
then
  echo "The value of \"a\" lies somewhere between 0 and 5."
fi
```

[[Example 37-4|Example 37-4]] and [[Example 17-11|Example 17-11]] demonstrate nested _if/then_ condition tests.

## Testing Your Knowledge of Tests

The systemwide xinitrc file can be used to launch the X server. This file contains quite a number of _if/then_ tests. The following is excerpted from an "ancient" version of xinitrc (_Red Hat 7.1_, or thereabouts).

```bash
if [ -f $HOME/.Xclients ]; then
  exec $HOME/.Xclients
elif [ -f /etc/X11/xinit/Xclients ]; then
  exec /etc/X11/xinit/Xclients
else
     # failsafe settings.  Although we should never get here
     # (we provide fallbacks in Xclients as well) it can't hurt.
     xclock -geometry 100x100-5+5 &
     xterm -geometry 80x50-50+150 &
     if [ -f /usr/bin/netscape -a -f /usr/share/doc/HTML/index.html ]; then
             netscape /usr/share/doc/HTML/index.html &
     fi
fi
```

Explain the _test_ constructs in the above snippet, then examine an updated version of the file, /etc/X11/xinit/xinitrc, and analyze the _if/then_ test constructs there. You may need to refer ahead to the discussions of [[text-processing-commands#^GREPREF|grep]], [[sedawk#^SEDREF|sed]], and [[regexp#^REGEXREF|regular expressions]].

[^1]: A _token_ is a symbol or short string with a special meaning attached to it (a [[brief-introduction-to-regular-expressions#^metameaningref|meta-meaning]]). In Bash, certain tokens, such as **[[special-characters#^DOTREF|** and [. (dot-command)]], may expand to _keywords_ and commands.

[^2]: Per the 1913 edition of _Webster's Dictionary_:

    ```bash
    Deprecate
    ...

    To pray against, as an evil;
    to seek to avert by prayer;
    to desire the removal of;
    to seek deliverance from;
    to express deep regret for;
    to disapprove of strongly.
    ```

[^3]: Be aware that _suid_ binaries may open security holes. The _suid_ flag has no effect on shell scripts.

[^4]: On Linux systems, the sticky bit is no longer used for files, only on directories.

[^5]: As S.C. points out, in a compound test, even quoting the string variable might not suffice. **[ -n "$string" -o "$a" = "$b" ]** may cause an error with some versions of Bash if $string is empty. The safe way is to append an extra character to possibly empty variables, **[ "x$string" != x -o "x$a" = "x$b" ]** (the "x's" cancel out).
