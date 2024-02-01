---
title: 19. Here Documents
---


> Here and now, boys.
>
>--<cite>Aldous Huxley, _Island_</cite>

A _here document_ is a special-purpose code block. It uses a form of [[io-redirection|I/O redirection]] to feed a command list to an interactive program or a command, such as [[communications-commands#^FTPREF|ftp]], [[external-filters-programs-and-commands#^CATREF|cat]], or the _ex_ text editor.

```bash
COMMAND <<InputComesFromHERE
...
...
...
InputComesFromHERE
```

A _limit string_ delineates (frames) the command list. The special symbol << precedes the limit string. This has the effect of redirecting the output of a command block into the stdin of the program or command. It is similar to **interactive-program < command-file**, where command-file contains

```bash
command #1
command #2
...
```

The _here document_ equivalent looks like this:

```bash
interactive-program <<LimitString
command #1
command #2
...
LimitString
```

Choose a _limit string_ sufficiently unusual that it will not occur anywhere in the command list and confuse matters.

Note that _here documents_ may sometimes be used to good effect with non-interactive utilities and commands, such as, for example, [[system-and-administrative-commands#^WALLREF|wall]].

![[Example 19-1|Example 19-1]]

Even such unlikely candidates as the _vi_ text editor lend themselves to _here documents_.

![[Example 19-2|Example 19-2]]

The above script could just as effectively have been implemented with **ex**, rather than **vi**. _Here documents_ containing a list of **ex** commands are common enough to form their own category, known as _ex scripts_.

```bash
#!/bin/bash
#  Replace all instances of "Smith" with "Jones"
#+ in files with a ".txt" filename suffix. 

ORIGINAL=Smith
REPLACEMENT=Jones

for word in $(fgrep -l $ORIGINAL *.txt)
do
  # -------------------------------------
  ex $word <<EOF
  :%s/$ORIGINAL/$REPLACEMENT/g
  :wq
EOF
  # :%s is the "ex" substitution command.
  # :wq is write-and-quit.
  # -------------------------------------
done
```

Analogous to "ex scripts" are _cat scripts_.

![[Example 19-3|Example 19-3]]

The - option to mark a here document limit string (**<<-LimitString**) suppresses leading tabs (but not spaces) in the output. This may be useful in making a script more readable.

![[Example 19-4|Example 19-4]]

A _here document_ supports parameter and command substitution. It is therefore possible to pass different parameters to the body of the here document, changing its output accordingly.

![[Example 19-5|Example 19-5]]

This is a useful script containing a _here document_ with parameter substitution.

![[Example 19-6|Example 19-6]]

Quoting or escaping the "limit string" at the head of a here document disables parameter substitution within its body. The reason for this is that _quoting/escaping the limit string_ effectively [[quoting#^ESCP|escapes]] the $, `, and \ [[special-characters#^SCHARLIST|special characters]], and causes them to be interpreted literally. (Thank you, Allen Halsey, for pointing this out.)

![[Example 19-7]]

Disabling parameter substitution permits outputting literal text. Generating scripts or even program code is one use for this.

![[Example 19-8]]

It is possible to set a variable from the output of a here document. This is actually a devious form of [[command-substitution#^COMMANDSUBREF|command substitution]].

```bash
variable=$(cat <<SETVAR
This variable
runs over multiple lines.
SETVAR
)

echo "$variable"
```

A here document can supply input to a function in the same script.

![[Example 19-9]]

It is possible to use : as a dummy command accepting output from a here document. This, in effect, creates an "anonymous" here document.

![[Example 19-10]]

> [!tip]
> A variation of the above technique permits "commenting out" blocks of code.

![[Example 19-11]]

> [!tip]
> Yet another twist of this nifty trick makes "self-documenting" scripts possible.

![[Example 19-12]]

Using a [[here-documents#^CATSCRIPTREF|cat script]] is an alternate way of accomplishing this.

```bash
DOC_REQUEST=70

if [ "$1" = "-h"  -o "$1" = "--help" ]     # Request help.
then                                       # Use a "cat script" . . .
  cat <<DOCUMENTATIONXX
List the statistics of a specified directory in tabular format.
---------------------------------------------------------------
The command-line parameter gives the directory to be listed.
If no directory specified or directory specified cannot be read,
then list the current working directory.

DOCUMENTATIONXX
exit $DOC_REQUEST
fi
```

See also [[Example A-28]], [[Example A-40]], [[Example A-41]], and [[Example A-42]] for more examples of self-documenting scripts.

> [!note]
> Here documents create temporary files, but these files are deleted after opening and are not accessible to any other process.
>
> ```bash
> bash$ bash -c 'lsof -a -p $$ -d0' << EOF
> EOF
> lsof    1213 bozo    0r   REG    3,5    0 30386 /tmp/t1213-0-sh (deleted)
> ```

> [!caution]
> Some utilities will not work inside a _here document_.

> [!warning]
> The closing _limit string_, on the final line of a here document, must start in the _first_ character position. There can be _no leading whitespace_. Trailing whitespace after the limit string likewise causes unexpected behavior. The whitespace prevents the limit string from being recognized. [^1]
>
> ```bash
> #!/bin/bash
> 
> echo "----------------------------------------------------------------------"
> 
> cat <<LimitString
> echo "This is line 1 of the message inside the here document."
> echo "This is line 2 of the message inside the here document."
> echo "This is the final line of the message inside the here document."
>      LimitString
> #^^^^Indented limit string. Error! This script will not behave as expected.
> 
> echo "----------------------------------------------------------------------"
> 
> #  These comments are outside the 'here document',
> #+ and should not echo.
> 
> echo "Outside the here document."
> 
> exit 0
> 
> echo "This line had better not echo."  # Follows an 'exit' command.
> ```

> [!caution]
> Some people very cleverly use a single ! as a limit string. But, that's not necessarily a good idea.
>
> ```bash
> # This works.
> cat <<!
> Hello!
> ! Three more exclamations !!!
> !
> 
> 
> # But . . .
> cat <<!
> Hello!
> Single exclamation point follows!
> !
> !
> # Crashes with an error message.
> 
> 
> # However, the following will work.
> cat <<EOF
> Hello!
> Single exclamation point follows!
> !
> EOF
> # It's safer to use a multi-character limit string.
> ```

For those tasks too complex for a _here document_, consider using the _expect_ scripting language, which was specifically designed for feeding input into interactive programs.

## Here Strings

> A _here string_ can be considered as a stripped-down form of a _here document_.  
> It consists of nothing more than **COMMAND <<< $WORD**,  
> where $WORD is expanded and fed to the stdin of **COMMAND**.

As a simple example, consider this alternative to the [[internal-commands-and-builtins#^ECHOGREPREF|echo-grep]] construction.

```bash
# Instead of:
if echo "$VAR" | grep -q txt   # if [[ $VAR = *txt* | $VAR = *txt* ]]
# etc.

# Try:
if grep -q "txt" <<< "$VAR"
then   #         ^^^
   echo "$VAR contains the substring sequence \"txt\""
fi
# Thank you, Sebastian Kaminski, for the suggestion.
```

Or, in combination with [[internal-commands-and-builtins#^READREF|read]]:

```bash
String="This is a string of words."

read -r -a Words <<< "$String"
#  The -a option to "read"
#+ assigns the resulting values to successive members of an array.

echo "First word in String is:    ${Words[0]}"   # This
echo "Second word in String is:   ${Words[1]}"   # is
echo "Third word in String is:    ${Words[2]}"   # a
echo "Fourth word in String is:   ${Words[3]}"   # string
echo "Fifth word in String is:    ${Words[4]}"   # of
echo "Sixth word in String is:    ${Words[5]}"   # words.
echo "Seventh word in String is:  ${Words[6]}"   # (null)
                                                 # Past end of $String.

# Thank you, Francisco Lobo, for the suggestion.
```

It is, of course, possible to feed the output of a _here string_ into the stdin of a [[loops-and-branches#^LOOPREF00|loop]].

```bash
# As Seamus points out . . .

ArrayVar=( element0 element1 element2 {A..D} )

while read element ; do
  echo "$element" 1>&2
done <<< $(echo ${ArrayVar[*]})

# element0 element1 element2 A B C D
```

![[Example 19-13|Example 19-13]]

![[Example 19-14|Example 19-14]]

Exercise: Find other uses for _here strings_, such as, for example, [[math-commands#^GOLDENRATIO|feeding input to _dc_]].

[^1]: Except, as Dennis Benzinger points out, if [[here-documents#^LIMITSTRDASH|using **<<-** to suppress tabs]].
