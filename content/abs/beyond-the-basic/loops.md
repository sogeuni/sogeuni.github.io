---
title: 11.1. Loops
---


A _loop_ is a block of code that _iterates_ [^1] a list of commands as long as the _loop control condition_ is true.

**for loops**

**for _arg_ in _[list]_**

This is the basic looping construct. It differs significantly from its _C_ counterpart.

**for** _arg_ in [_list_]  
do
 command(s)...
done

> [!note]
> During each pass through the loop, _arg_ takes on the value of each successive variable in the _list_.

```bash
for arg in "$var1" "$var2" "$var3" ... "$varN"  
# In pass 1 of the loop, arg = $var1	    
# In pass 2 of the loop, arg = $var2	    
# In pass 3 of the loop, arg = $var3	    
# ...
# In pass N of the loop, arg = $varN

# Arguments in [list] quoted to prevent possible word splitting.
```

The argument _list_ may contain [[special-chars#ASTERISKREF|wild cards]].

If _do_ is on same line as _for_, there needs to be a semicolon after list.

**for** _arg_ in [_list_] ; do  

###### Example 11-1. Simple *for* loops

```bash
#!/bin/bash
# Listing the planets.

for planet in Mercury Venus Earth Mars Jupiter Saturn Uranus Neptune Pluto
do
  echo $planet  # Each planet on a separate line.
done

echo; echo

for planet in "Mercury Venus Earth Mars Jupiter Saturn Uranus Neptune Pluto"
    # All planets on same line.
    # Entire 'list' enclosed in quotes creates a single variable.
    # Why? Whitespace incorporated into the variable.
do
  echo $planet
done

echo; echo "Whoops! Pluto is no longer a planet!"

exit 0
```

Each **[list]** element may contain multiple parameters. This is useful when processing parameters in groups. In such cases, use the [[../commands/internal-commands-and-builtins#SETREF|set]] command (see [[../commands/internal-commands-and-builtins#EX34|Example 15-16]]) to force parsing of each **[list]** element and assignment of each component to the positional parameters.

###### Example 11-2. *for* loop with two parameters in each [list] element

```bash
#!/bin/bash
# Planets revisited.

# Associate the name of each planet with its distance from the sun.

for planet in "Mercury 36" "Venus 67" "Earth 93"  "Mars 142" "Jupiter 483"
do
  set -- $planet  #  Parses variable "planet"
                  #+ and sets positional parameters.
  #  The "--" prevents nasty surprises if $planet is null or
  #+ begins with a dash.

  #  May need to save original positional parameters,
  #+ since they get overwritten.
  #  One way of doing this is to use an array,
  #         original_params=("$@")

  echo "$1		$2,000,000 miles from the sun"
  #-------two  tabs---concatenate zeroes onto parameter $2
done

# (Thanks, S.C., for additional clarification.)

exit 0
```

A variable may supply the **[list]** in a _for loop_.

###### Example 11-3. *Fileinfo:* operating on a file list contained in a variable

```bash
#!/bin/bash
# fileinfo.sh

FILES="/usr/sbin/accept
/usr/sbin/pwck
/usr/sbin/chroot
/usr/bin/fakefile
/sbin/badblocks
/sbin/ypbind"     # List of files you are curious about.
                  # Threw in a dummy file, /usr/bin/fakefile.

echo

for file in $FILES
do

  if [ ! -e "$file" ]       # Check if file exists.
  then
    echo "$file does not exist."; echo
    continue                # On to next.
   fi

  ls -l $file | awk '{ print $8 "         file size: " $5 }'  # Print 2 fields.
  whatis `basename $file`   # File info.
  # Note that the whatis database needs to have been set up for this to work.
  # To do this, as root run /usr/bin/makewhatis.
  echo
done  

exit 0
```

The **[list]** in a _for loop_ may be parameterized.

###### Example 11-4. Operating on a parameterized file list

```bash
#!/bin/bash

filename="*txt"

for file in $filename
do
 echo "Contents of $file"
 echo "---"
 cat "$file"
 echo
done
```

If the **[list]** in a _for loop_ contains wild cards (* and ?) used in filename expansion, then [[../advanced-topics/globbing|globbing]] takes place.

###### Example 11-5. Operating on files with a *for* loop

```bash
#!/bin/bash
# list-glob.sh: Generating [list] in a for-loop, using "globbing" ...
# Globbing = filename expansion.

echo

for file in *
#           ^  Bash performs filename expansion
#+             on expressions that globbing recognizes.
do
  ls -l "$file"  # Lists all files in $PWD (current directory).
  #  Recall that the wild card character "*" matches every filename,
  #+ however, in "globbing," it doesn't match dot-files.

  #  If the pattern matches no file, it is expanded to itself.
  #  To prevent this, set the nullglob option
  #+   (shopt -s nullglob).
  #  Thanks, S.C.
done

echo; echo

for file in [jx]*
do
  rm -f $file    # Removes only files beginning with "j" or "x" in $PWD.
  echo "Removed file \"$file\"".
done

echo

exit 0
```

Omitting the **in [list]** part of a _for loop_ causes the loop to operate on $@ -- the [[internalvariables#POSPARAMREF|positional parameters]]. A particularly clever illustration of this is [[../apendix/contributed-scripts#PRIMES|Example A-15]]. See also [[../commands/internal-commands-and-builtins#REVPOSPARAMS|Example 15-17]].

###### Example 11-6. Missing **in [list]** in a *for* loop

```bash
#!/bin/bash

#  Invoke this script both with and without arguments,
#+ and see what happens.

for a
do
 echo -n "$a "
done

#  The 'in list' missing, therefore the loop operates on '$@'
#+ (command-line argument list, including whitespace).

echo

exit 0
```

It is possible to use [[./command-substitution#COMMANDSUBREF|command substitution]] to generate the **[list]** in a _for loop_. See also [[../commands/miscellaneous-commands#EX53|Example 16-54]], [[loops1#SYMLINKS|Example 11-11]] and [[../commands/math-commands#BASE|Example 16-48]].

###### Example 11-7. Generating the **[list]** in a *for* loop with command substitution

```bash
#!/bin/bash
#  for-loopcmd.sh: for-loop with [list]
#+ generated by command substitution.

NUMBERS="9 7 3 8 37.53"

for number in `echo $NUMBERS`  # for number in 9 7 3 8 37.53
do
  echo -n "$number "
done

echo 
exit 0
```

Here is a somewhat more complex example of using command substitution to create the **[list]**.

###### Example 11-8. A *grep* replacement for binary files

```bash
#!/bin/bash
# bin-grep.sh: Locates matching strings in a binary file.

# A "grep" replacement for binary files.
# Similar effect to "grep -a"

E_BADARGS=65
E_NOFILE=66

if [ $# -ne 2 ]
then
  echo "Usage: `basename $0` search_string filename"
  exit $E_BADARGS
fi

if [ ! -f "$2" ]
then
  echo "File \"$2\" does not exist."
  exit $E_NOFILE
fi  


IFS=$'\012'       # Per suggestion of Anton Filippov.
                  # was:  IFS="\n"
for word in $( strings "$2" | grep "$1" )
# The "strings" command lists strings in binary files.
# Output then piped to "grep", which tests for desired string.
do
  echo $word
done

# As S.C. points out, lines 23 - 30 could be replaced with the simpler
#    strings "$2" | grep "$1" | tr -s "$IFS" '[\n*]'


#  Try something like  "./bin-grep.sh mem /bin/ls"
#+ to exercise this script.

exit 0
```

More of the same.

###### Example 11-9. Listing all users on the system

```bash
#!/bin/bash
# userlist.sh

PASSWORD_FILE=/etc/passwd
n=1           # User number

for name in $(awk 'BEGIN{FS=":"}{print $1}' < "$PASSWORD_FILE" )
# Field separator = :    ^^^^^^
# Print first field              ^^^^^^^^
# Get input from password file  /etc/passwd  ^^^^^^^^^^^^^^^^^
do
  echo "USER #$n = $name"
  let "n += 1"
done  


# USER #1 = root
# USER #2 = bin
# USER #3 = daemon
# ...
# USER #33 = bozo

exit $?

#  Discussion:
#  ----------
#  How is it that an ordinary user, or a script run by same,
#+ can read /etc/passwd? (Hint: Check the /etc/passwd file permissions.)
#  Is this a security hole? Why or why not?
```

Yet another example of the **[list]** resulting from command substitution.

###### Example 11-10. Checking all the binaries in a directory for authorship

```bash
#!/bin/bash
# findstring.sh:
# Find a particular string in the binaries in a specified directory.

directory=/usr/bin/
fstring="Free Software Foundation"  # See which files come from the FSF.

for file in $( find $directory -type f -name '*' | sort )
do
  strings -f $file | grep "$fstring" | sed -e "s%$directory%%"
  #  In the "sed" expression,
  #+ it is necessary to substitute for the normal "/" delimiter
  #+ because "/" happens to be one of the characters filtered out.
  #  Failure to do so gives an error message. (Try it.)
done  

exit $?

#  Exercise (easy):
#  ---------------
#  Convert this script to take command-line parameters
#+ for $directory and $fstring.
```

A final example of **[list]** / command substitution, but this time the "command" is a [[../advanced-topics/functions|function]].

```bash
generate_list ()
{
  echo "one two three"
}

for word in $(generate_list)  # Let "word" grab output of function.
do
  echo "$word"
done

# one
# two
# three
```

The output of a _for loop_ may be piped to a command or commands.

###### Example 11-11. Listing the *symbolic links* in a directory

```bash
#!/bin/bash
# symlinks.sh: Lists symbolic links in a directory.


directory=${1-`pwd`}
#  Defaults to current working directory,
#+ if not otherwise specified.
#  Equivalent to code block below.
# ----------------------------------------------------------
# ARGS=1                 # Expect one command-line argument.
#
# if [ $# -ne "$ARGS" ]  # If not 1 arg...
# then
#   directory=`pwd`      # current working directory
# else
#   directory=$1
# fi
# ----------------------------------------------------------

echo "symbolic links in directory \"$directory\""

for file in "$( find $directory -type l )"   # -type l = symbolic links
do
  echo "$file"
done | sort                                  # Otherwise file list is unsorted.
#  Strictly speaking, a loop isn't really necessary here,
#+ since the output of the "find" command is expanded into a single word.
#  However, it's easy to understand and illustrative this way.

#  As Dominik 'Aeneas' Schnitzer points out,
#+ failing to quote  $( find $directory -type l )
#+ will choke on filenames with embedded whitespace.
#  containing whitespace. 

exit 0


# --------------------------------------------------------
# Jean Helou proposes the following alternative:

echo "symbolic links in directory \"$directory\""
# Backup of the current IFS. One can never be too cautious.
OLDIFS=$IFS
IFS=:

for file in $(find $directory -type l -printf "%p$IFS")
do     #                              ^^^^^^^^^^^^^^^^
       echo "$file"
       done|sort

# And, James "Mike" Conley suggests modifying Helou's code thusly:

OLDIFS=$IFS
IFS='' # Null IFS means no word breaks
for file in $( find $directory -type l )
do
  echo $file
  done | sort

#  This works in the "pathological" case of a directory name having
#+ an embedded colon.
#  "This also fixes the pathological case of the directory name having
#+  a colon (or space in earlier example) as well."
```

The stdout of a loop may be [[../advanced-topics/io-redirection|redirected]] to a file, as this slight modification to the previous example shows.

###### Example 11-12. Symbolic links in a directory, saved to a file

```bash
#!/bin/bash
# symlinks.sh: Lists symbolic links in a directory.

OUTFILE=symlinks.list                         # save-file

directory=${1-`pwd`}
#  Defaults to current working directory,
#+ if not otherwise specified.


echo "symbolic links in directory \"$directory\"" > "$OUTFILE"
echo "---------------------------" >> "$OUTFILE"

for file in "$( find $directory -type l )"    # -type l = symbolic links
do
  echo "$file"
done | sort >> "$OUTFILE"                     # stdout of loop
#           ^^^^^^^^^^^^^                       redirected to save file.

# echo "Output file = $OUTFILE"

exit $?
```

There is an alternative syntax to a _for loop_ that will look very familiar to C programmers. This requires [[dblparens#DBLPARENSREF|double parentheses]].

###### Example 11-13. A C-style *for* loop

```bash
#!/bin/bash
# Multiple ways to count up to 10.

echo

# Standard syntax.
for a in 1 2 3 4 5 6 7 8 9 10
do
  echo -n "$a "
done  

echo; echo

# +==========================================+

# Using "seq" ...
for a in `seq 10`
do
  echo -n "$a "
done  

echo; echo

# +==========================================+

# Using brace expansion ...
# Bash, version 3+.
for a in {1..10}
do
  echo -n "$a "
done  

echo; echo

# +==========================================+

# Now, let's do the same, using C-like syntax.

LIMIT=10

for ((a=1; a <= LIMIT ; a++))  # Double parentheses, and naked "LIMIT"
do
  echo -n "$a "
done                           # A construct borrowed from ksh93.

echo; echo

# +=========================================================================+

# Let's use the C "comma operator" to increment two variables simultaneously.

for ((a=1, b=1; a <= LIMIT ; a++, b++))
do  # The comma concatenates operations.
  echo -n "$a-$b "
done

echo; echo

exit 0
```

See also [[../advanced-topics/arrays#^QFUNCTION|Example 27-16]], [[../advanced-topics/arrays#^TWODIM|Example 27-17]], and [[../apendix/contributed-scripts#^COLLATZ|Example A-6]].

---

Now, a _for loop_ used in a "real-life" context.

###### Example 11-14. Using *efax* in batch mode

```bash
#!/bin/bash
# Faxing (must have 'efax' package installed).

EXPECTED_ARGS=2
E_BADARGS=85
MODEM_PORT="/dev/ttyS2"   # May be different on your machine.
#                ^^^^^      PCMCIA modem card default port.

if [ $# -ne $EXPECTED_ARGS ]
# Check for proper number of command-line args.
then
   echo "Usage: `basename $0` phone# text-file"
   exit $E_BADARGS
fi


if [ ! -f "$2" ]
then
  echo "File $2 is not a text file."
  #     File is not a regular file, or does not exist.
  exit $E_BADARGS
fi
  

fax make $2              #  Create fax-formatted files from text files.

for file in $(ls $2.0*)  #  Concatenate the converted files.
                         #  Uses wild card (filename "globbing")
			 #+ in variable list.
do
  fil="$fil $file"
done  

efax -d "$MODEM_PORT"  -t "T$1" $fil   # Finally, do the work.
# Trying adding  -o1  if above line fails.


#  As S.C. points out, the for-loop can be eliminated with
#     efax -d /dev/ttyS2 -o1 -t "T$1" $2.0*
#+ but it's not quite as instructive [grin].

exit $?   # Also, efax sends diagnostic messages to stdout.
```

> [!note]
> The [[../commands/internal-commands-and-builtins#keywordref|keywords]] **do** and **done** delineate the _for-loop_ command block. However, these may, in certain contexts, be omitted by framing the command block within [[special-chars#CODEBLOCKREF|curly brackets]]
>
> ```bash
> for((n=1; n<=10; n++)) 
> # No do!
> {
>   echo -n "* $n *"
> }
> # No done!
> 
> 
> # Outputs:
> # * 1 ** 2 ** 3 ** 4 ** 5 ** 6 ** 7 ** 8 ** 9 ** 10 *
> # And, echo $? returns 0, so Bash does not register an error.
> 
> 
> echo
> 
> 
> #  But, note that in a classic for-loop:    for n in [list] ...
> #+ a terminal semicolon is required.
> 
> for n in 1 2 3
> {  echo -n "$n "; }
> #               ^
> 
> 
> # Thank you, YongYe, for pointing this out.
> ```

**while**

This construct tests for a condition at the top of a loop, and keeps looping as long as that condition is true (returns a 0 [[exit-and-exit-status#EXITSTATUSREF|exit status]]). In contrast to a [[loops1#FORLOOPREF1|for loop]], a _while loop_ finds use in situations where the number of loop repetitions is not known beforehand.

**while** [ _condition_ ]  
do
 command(s)...
done

The bracket construct in a _while loop_ is nothing more than our old friend, the [[testconstructs#TESTCONSTRUCTS1|test brackets]] used in an _if/then_ test. In fact, a _while loop_ can legally use the more versatile [[testconstructs#DBLBRACKETS|double-brackets construct]] (while [[ condition | condition ]]).

[[loops1#NEEDSEMICOLON|As is the case with _for loops_]], placing the _do_ on the same line as the condition test requires a semicolon.

**while** [ _condition_ ] ; do

Note that the _test brackets_ [[loops1#WHILENOBRACKETS|are _not_ mandatory]] in a _while_ loop. See, for example, the [[../commands/internal-commands-and-builtins#GETOPTSX|getopts construct]].

###### Example 11-15. Simple *while* loop

```bash
#!/bin/bash

var0=0
LIMIT=10

while [ "$var0" -lt "$LIMIT" ]
#      ^                    ^
# Spaces, because these are "test-brackets" . . .
do
  echo -n "$var0 "        # -n suppresses newline.
  #             ^           Space, to separate printed out numbers.

  var0=`expr $var0 + 1`   # var0=$(($var0+1))  also works.
                          # var0=$((var0 + 1)) also works.
                          # let "var0 += 1"    also works.
done                      # Various other methods also work.

echo

exit 0
```

###### Example 11-16. Another *while* loop

```bash
#!/bin/bash

echo
                               # Equivalent to:
while [ "$var1" != "end" ]     # while test "$var1" != "end"
do
  echo "Input variable #1 (end to exit) "
  read var1                    # Not 'read $var1' (why?).
  echo "variable #1 = $var1"   # Need quotes because of "#" . . .
  # If input is 'end', echoes it here.
  # Does not test for termination condition until top of loop.
  echo
done  

exit 0
```

A _while loop_ may have multiple conditions. Only the final condition determines when the loop terminates. This necessitates a slightly different loop syntax, however.

###### Example 11-17. *while* loop with multiple conditions

```bash
#!/bin/bash

var1=unset
previous=$var1

while echo "previous-variable = $previous"
      echo
      previous=$var1
      [ "$var1" != end ] # Keeps track of what $var1 was previously.
      # Four conditions on *while*, but only the final one controls loop.
      # The *last* exit status is the one that counts.
do
echo "Input variable #1 (end to exit) "
  read var1
  echo "variable #1 = $var1"
done  

# Try to figure out how this all works.
# It's a wee bit tricky.

exit 0
```

As with a _for loop_, a _while loop_ may employ C-style syntax by using the double-parentheses construct (see also [[dblparens#CVARS|Example 8-5]]).

###### Example 11-18. C-style syntax in a *while* loop

```bash
#!/bin/bash
# wh-loopc.sh: Count to 10 in a "while" loop.

LIMIT=10                 # 10 iterations.
a=1

while [ "$a" -le $LIMIT ]
do
  echo -n "$a "
  let "a+=1"
done                     # No surprises, so far.

echo; echo

# +=================================================================+

# Now, we'll repeat with C-like syntax.

((a = 1))      # a=1
# Double parentheses permit space when setting a variable, as in C.

while (( a <= LIMIT ))   #  Double parentheses,
do                       #+ and no "$" preceding variables.
  echo -n "$a "
  ((a += 1))             # let "a+=1"
  # Yes, indeed.
  # Double parentheses permit incrementing a variable with C-like syntax.
done

echo

# C and Java programmers can feel right at home in Bash.

exit 0
```

Inside its test brackets, a _while loop_ can call a [[../advanced-topics/functions|function]].

```bash
t=0

condition ()
{
  ((t++))

  if [ $t -lt 5 ]
  then
    return 0  # true
  else
    return 1  # false
  fi
}

while condition
#     ^^^^^^^^^
#     Function call -- four loop iterations.
do
  echo "Still going: t = $t"
done

# Still going: t = 1
# Still going: t = 2
# Still going: t = 3
# Still going: t = 4
```

> Similar to the [[testconstructs#IFGREPREF|if-test]] construct, a _while_ loop can omit the test brackets.
>
> ```bash
> while condition
> do
>    command(s) ...
> done
> ```

By coupling the power of the [[../commands/internal-commands-and-builtins#READREF|read]] command with a _while loop_, we get the handy [[../commands/internal-commands-and-builtins#WHILEREADREF|while read]] construct, useful for reading and parsing files.

```bash
cat $filename |   # Supply input from a file.
while read line   # As long as there is another line to read ...
do
  ...
done

# =========== Snippet from "sd.sh" example script ========== #

  while read value   # Read one data point at a time.
  do
    rt=$(echo "scale=$SC; $rt + $value" | bc)
    (( ct++ ))
  done

  am=$(echo "scale=$SC; $rt / $ct" | bc)

  echo $am; return $ct   # This function "returns" TWO values!
  #  Caution: This little trick will not work if $ct > 255!
  #  To handle a larger number of data points,
  #+ simply comment out the "return $ct" above.
} <"$datafile"   # Feed in data file.
```

> [!note]
> A _while loop_ may have its stdin [[../advanced-topics/redirecting-code-blocks#REDIRREF|redirected to a file]] by a < at its end.
>
> A _while loop_ may have its stdin [[../commands/internal-commands-and-builtins#READPIPEREF|supplied by a pipe]].

**until**

This construct tests for a condition at the top of a loop, and keeps looping as long as that condition is _false_ (opposite of _while loop_).

**until** [ _condition-is-true_ ]  
do
 command(s)...
done

Note that an _until loop_ tests for the terminating condition at the _top_ of the loop, differing from a similar construct in some programming languages.

As is the case with _for loops_, placing the _do_ on the same line as the condition test requires a semicolon.

**until** [ _condition-is-true_ ] ; do

###### Example 11-19. *until* loop

```bash
#!/bin/bash

END_CONDITION=end

until [ "$var1" = "$END_CONDITION" ]
# Tests condition here, at top of loop.
do
  echo "Input variable #1 "
  echo "($END_CONDITION to exit)"
  read var1
  echo "variable #1 = $var1"
  echo
done  

#                     ---                        #

#  As with "for" and "while" loops,
#+ an "until" loop permits C-like test constructs.

LIMIT=10
var=0

until (( var > LIMIT ))
do  # ^^ ^     ^     ^^   No brackets, no $ prefixing variables.
  echo -n "$var "
  (( var++ ))
done    # 0 1 2 3 4 5 6 7 8 9 10 


exit 0
```

How to choose between a _for_ loop or a _while_ loop or _until_ loop? In **C**, you would typically use a _for_ loop when the number of loop iterations is known beforehand. With _Bash_, however, the situation is fuzzier. The Bash _for_ loop is more loosely structured and more flexible than its equivalent in other languages. Therefore, feel free to use whatever type of loop gets the job done in the simplest way.

[^1]: _Iteration_: Repeated execution of a command or group of commands, usually -- but not always, _while_ a given condition holds, or _until_ a given condition is met.
