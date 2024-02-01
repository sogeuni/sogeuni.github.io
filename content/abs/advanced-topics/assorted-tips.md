---
title: 36.7. Assorted Tips
---


## 36.7.1. Ideas for more powerful scripts

- You have a problem that you want to solve by writing a Bash script. Unfortunately, you don't know quite where to start. One method is to plunge right in and code those parts of the script that come easily, and write the hard parts as *pseudo-code*.

```bash
#!/bin/bash

ARGCOUNT=1                     # Need name as argument.
E_WRONGARGS=65

if [ number-of-arguments is-not-equal-to "$ARGCOUNT" ]
#    ^^^^^^^^^^^^^^^^^^^ ^^^^^^^^^^^^^^^
#  Can't figure out how to code this . . .
#+ . . . so write it in pseudo-code.

then
  echo "Usage: name-of-script name"
  #            ^^^^^^^^^^^^^^     More pseudo-code.
  exit $E_WRONGARGS
fi 

. . .

exit 0


# Later on, substitute working code for the pseudo-code.

# Line 6 becomes:
if [ $# -ne "$ARGCOUNT" ]

# Line 12 becomes:
  echo "Usage: `basename $0` name"
```

For an example of using pseudo-code, see the [[writing-scripts#^NEWTONSQRT|Square Root]] exercise.

- To keep a record of which user scripts have run during a particular session or over a number of sessions, add the following lines to each script you want to keep track of. This will keep a continuing file record of the script names and invocation times.

```bash
# Append (>>) following to end of each script tracked.

whoami>> $SAVE_FILE    # User invoking the script.
echo $0>> $SAVE_FILE   # Script name.
date>> $SAVE_FILE      # Date and time.
echo>> $SAVE_FILE      # Blank line as separator.

#  Of course, SAVE_FILE defined and exported as environmental variable in ~/.bashrc
#+ (something like ~/.scripts-run)
```

- The >> operator *appends* lines to a file. What if you wish to *prepend* a line to an existing file, that is, to paste it in at the beginning?

```bash
file=data.txt
title="***This is the title line of data text file***"

echo $title | cat - $file >$file.new
# "cat -" concatenates stdout to $file.
#  End result is
#+ to write a new file with $title appended at *beginning*.
```

This is a simplified variant of the [[Example 19-13|Example 19-13]] script given earlier. And, of course, [[sedawk#^SEDREF|sed]] can also do this.

- A shell script may act as an embedded command inside another shell script, a *Tcl* or *wish* script, or even a [[file-and-archiving-commands#^MAKEFILEREF|Makefile]]. It can be invoked as an external shell command in a C program using the *system()* call, i.e., *system("script_name");*.

- Setting a variable to the contents of an embedded _sed_ or _awk_ script increases the readability of the surrounding [[shell-wrappers#^SHWRAPPER|shell wrapper]]. See [[Example A-1|Example A-1]] and [[Example 15-20|Example 15-20]].

- Put together files containing your favorite and most useful definitions and functions. As necessary, "include" one or more of these "library files" in scripts with either the [[special-characters#^DOTREF|dot]] (**.**) or [[internal-commands-and-builtins#^SOURCEREF|source]] command.

```bash
# SCRIPT LIBRARY
# ------ -------

# Note:
# No "#!" here.
# No "live code" either.


# Useful variable definitions

ROOT_UID=0             # Root has $UID 0.
E_NOTROOT=101          # Not root user error. 
MAXRETVAL=255          # Maximum (positive) return value of a function.
SUCCESS=0
FAILURE=-1



# Functions

Usage ()               # "Usage:" message.
{
  if [ -z "$1" ]       # No arg passed.
  then
    msg=filename
  else
    msg=$@
  fi

  echo "Usage: `basename $0` "$msg""
}  


Check_if_root ()       # Check if root running script.
{                      # From "ex39.sh" example.
  if [ "$UID" -ne "$ROOT_UID" ]
  then
    echo "Must be root to run this script."
    exit $E_NOTROOT
  fi
}  


CreateTempfileName ()  # Creates a "unique" temp filename.
{                      # From "ex51.sh" example.
  prefix=temp
  suffix=`eval date +%s`
  Tempfilename=$prefix.$suffix
}


isalpha2 ()            # Tests whether *entire string* is alphabetic.
{                      # From "isalpha.sh" example.
  [ $# -eq 1 ] || return $FAILURE

  case $1 in
  *[!a-zA-Z]*|"") return $FAILURE;;
  *) return $SUCCESS;;
  esac                 # Thanks, S.C.
}


abs ()                           # Absolute value.
{                                # Caution: Max return value = 255.
  E_ARGERR=-999999

  if [ -z "$1" ]                 # Need arg passed.
  then
    return $E_ARGERR             # Obvious error value returned.
  fi

  if [ "$1" -ge 0 ]              # If non-negative,
  then                           #
    absval=$1                    # stays as-is.
  else                           # Otherwise,
    let "absval = (( 0 - $1 ))"  # change sign.
  fi  

  return $absval
}


tolower ()             #  Converts string(s) passed as argument(s)
{                      #+ to lowercase.

  if [ -z "$1" ]       #  If no argument(s) passed,
  then                 #+ send error message
    echo "(null)"      #+ (C-style void-pointer error message)
    return             #+ and return from function.
  fi  

  echo "$@" | tr A-Z a-z
  # Translate all passed arguments ($@).

  return

# Use command substitution to set a variable to function output.
# For example:
#    oldvar="A seT of miXed-caSe LEtTerS"
#    newvar=`tolower "$oldvar"`
#    echo "$newvar"    # a set of mixed-case letters
#
# Exercise: Rewrite this function to change lowercase passed argument(s)
#           to uppercase ... toupper()  [easy].
}
```

- Use special-purpose comment headers to increase clarity and legibility in scripts.

```bash
## Caution.
rm -rf *.zzy   ##  The "-rf" options to "rm" are very dangerous,
               ##+ especially with wild cards.

#+ Line continuation.
#  This is line 1
#+ of a multi-line comment,
#+ and this is the final line.

#* Note.

#o List item.

#> Another point of view.
while [ "$var1" != "end" ]    #> while test "$var1" != "end"
```

- Dotan Barak contributes template code for a _progress bar_ in a script.

![[Example 36-17|Example 36-17]]

- A particularly clever use of [[tests#^TESTCONSTRUCTS1|if-test]] constructs is for comment blocks.

```bash
#!/bin/bash

COMMENT_BLOCK=
#  Try setting the above variable to some value
#+ for an unpleasant surprise.

if [ $COMMENT_BLOCK ]; then

Comment block --
=================================
This is a comment line.
This is another comment line.
This is yet another comment line.
=================================

echo "This will not echo."

Comment blocks are error-free! Whee!

fi

echo "No more comments, please."

exit 0
```

Compare this with [[here-documents#^CBLOCK1|using here documents to comment out code blocks]].

- Using the [[another-look-at-variables#^XSTATVARREF|$? exit status variable]], a script may test if a parameter contains only digits, so it can be treated as an integer.

```bash
#!/bin/bash

SUCCESS=0
E_BADINPUT=85

test "$1" -ne 0 -o "$1" -eq 0 2>/dev/null
# An integer is either equal to 0 or not equal to 0.
# 2>/dev/null suppresses error message.

if [ $? -ne "$SUCCESS" ]
then
  echo "Usage: `basename $0` integer-input"
  exit $E_BADINPUT
fi

let "sum = $1 + 25"             # Would give error if $1 not integer.
echo "Sum = $sum"

# Any variable, not just a command-line parameter, can be tested this way.

exit 0
```

- The 0 - 255 range for function return values is a severe limitation. Global variables and other workarounds are often problematic. An alternative method for a function to communicate a value back to the main body of the script is to have the function write to stdout (usually with [[internal-commands-and-builtins#^ECHOREF|echo]]) the "return value," and assign this to a variable. This is actually a variant of [[command-substitution#^COMMANDSUBREF|command substitution.]]

![[Example 36-18|Example 36-18]]

The same technique also works for alphanumeric strings. This means that a function can "return" a non-numeric value.

```bash
capitalize_ichar ()          #  Capitalizes initial character
{                            #+ of argument string(s) passed.

  string0="$@"               # Accepts multiple arguments.

  firstchar=${string0:0:1}   # First character.
  string1=${string0:1}       # Rest of string(s).

  FirstChar=`echo "$firstchar" | tr a-z A-Z`
                             # Capitalize first character.

  echo "$FirstChar$string1"  # Output to stdout.

}  

newstring=`capitalize_ichar "every sentence should start with a capital letter."`
echo "$newstring"          # Every sentence should start with a capital letter.
```

It is even possible for a function to "return" multiple values with this method.

![[Example 36-19|Example 36-19]]

> [!caution]
> There can be only **one** _echo_ statement in the function for this to work. If you alter the previous example:
> 
> ```bash
> sum_and_product ()
> {
>   echo "This is the sum_and_product function." # This messes things up!
>   echo $(( $1 + $2 )) $(( $1 * $2 ))
> }
> ...
> retval=`sum_and_product $first $second`      # Assigns output of function.
> # Now, this will not work correctly.
> ```

- Next in our bag of tricks are techniques for passing an [[arrays#^ARRAYREF|array]] to a [[functions|function]], then "returning" an array back to the main body of the script.

    Passing an array involves loading the space-separated elements of the array into a variable with [[command-substitution#^COMMANDSUBREF|command substitution]]. Getting an array back as the "return value" from a function uses the previously mentioned strategem of [[internal-commands-and-builtins#^ECHOREF|echoing]] the array in the function, then invoking command substitution and the **( ... )** operator to assign it to an array.

![[Example 36-20|Example 36-20]]

For a more elaborate example of passing arrays to functions, see [[Example A-10|Example A-10]].

- Using the [[operations-and-related-topics|double-parentheses construct]], it is possible to use C-style syntax for setting and incrementing/decrementing variables and in [[loops-and-branches#^FORLOOPREF1|for]] and [[loops-and-branches#^WHILELOOPREF|while]] loops. See [[Example 11-13|Example 11-13]] and [[Example 11-18|Example 11-18]].

- Setting the [[another-look-at-variables#^PATHREF|path]] and [[system-and-administrative-commands#^UMASKREF|umask]] at the beginning of a script makes it more [[portability-issues|portable]] -- more likely to run on a "foreign" machine whose user may have bollixed up the $PATH and **umask**.

```bash
#!/bin/bash
PATH=/bin:/usr/bin:/usr/local/bin ; export PATH
umask 022   # Files that the script creates will have 755 permission.

# Thanks to Ian D. Allen, for this tip.
```

- A useful scripting technique is to _repeatedly_ feed the output of a filter (by piping) back to the _same filter_, but with a different set of arguments and/or options. Especially suitable for this are [[external-filters-programs-and-commands#^TRREF|tr]] and [[external-filters-programs-and-commands#^GREPREF|grep]].

```bash
# From "wstrings.sh" example.

wlist=`strings "$1" | tr A-Z a-z | tr '[:space:]' Z | \
tr -cs '[:alpha:]' Z | tr -s '\173-\377' Z | tr Z ' '`
```

![[Example 36-21|Example 36-21]]
    
See also [[Example 29-4|Example 29-4]], [[Example 16-25|Example 16-25]], and [[Example A-9|Example A-9]].

- Use "[[here-documents#^ANONHEREDOC0|anonymous here documents]]" to comment out blocks of code, to save having to individually comment out each line with a #. See [[Example 19-11|Example 19-11]].

- Running a script on a machine that relies on a command that might not be installed is dangerous. Use [[file-and-archiving-commands#^WHATISREF|whatis]] to avoid potential problems with this.

```bash
CMD=command1                 # First choice.
PlanB=command2               # Fallback option.

command_test=$(whatis "$CMD" | grep 'nothing appropriate')
#  If 'command1' not found on system , 'whatis' will return
#+ "command1: nothing appropriate."
#
#  A safer alternative is:
#     command_test=$(whereis "$CMD" | grep \/)
#  But then the sense of the following test would have to be reversed,
#+ since the $command_test variable holds content only if
#+ the $CMD exists on the system.
#     (Thanks, bojster.)


if [[ -z "$command_test" ]]  # Check whether command present.
then
  $CMD option1 option2       #  Run command1 with options.
else                         #  Otherwise,
  $PlanB                     #+ run command2. 
fi
```

- An [[tests#^IFGREPREF|if-grep test]] may not return expected results in an error case, when text is output to stderr, rather that stdout.

```bash
if ls -l nonexistent_filename | grep -q 'No such file or directory'
  then echo "File \"nonexistent_filename\" does not exist."
fi
```

[[io-redirection|Redirecting]] stderr to stdout fixes this.

```bash
if ls -l nonexistent_filename 2>&1 | grep -q 'No such file or directory'
#                             ^^^^
  then echo "File \"nonexistent_filename\" does not exist."
fi

# Thanks, Chris Martin, for pointing this out.
```

- If you absolutely must access a subshell variable outside the subshell, here's a way to do it.

```bash
TMPFILE=tmpfile                  # Create a temp file to store the variable.

(   # Inside the subshell ...
inner_variable=Inner
echo $inner_variable
echo $inner_variable >>$TMPFILE  # Append to temp file.
)

    # Outside the subshell ...

echo; echo "-----"; echo
echo $inner_variable             # Null, as expected.
echo "-----"; echo

# Now ...
read inner_variable <$TMPFILE    # Read back shell variable.
rm -f "$TMPFILE"                 # Get rid of temp file.
echo "$inner_variable"           # It's an ugly kludge, but it works.
```

- The [[miscellaneous-commands#^RUNPARTSREF|run-parts]] command is handy for running a set of command scripts in a particular sequence, especially in combination with [[system-and-administrative-commands#^CRONREF|cron]] or [[external-filters-programs-and-commands#^ATREF|at]].

- For doing multiple revisions on a complex script, use the _rcs_ Revision Control System package.

    Among other benefits of this is automatically updated ID header tags. The **co** command in _rcs_ does a parameter replacement of certain reserved key words, for example, replacing _# $Id$_ in a script with something like:

```bash
# $Id: hello-world.sh,v 1.1 2004/10/16 02:43:05 bozo Exp $
```

## 36.7.2. Widgets

It would be nice to be able to invoke X-Windows widgets from a shell script. There happen to exist several packages that purport to do so, namely _Xscript_, _Xmenu_, and _widtools_. The first two of these no longer seem to be maintained. Fortunately, it is still possible to obtain _widtools_ [here](http://www.batse.msfc.nasa.gov/~mallozzi/home/software/xforms/src/widtools-2.0.tgz).

> [!caution]
> The _widtools_ (widget tools) package requires the _XForms_ library to be installed. Additionally, the [[file-and-archiving-commands#^MAKEFILEREF|Makefile]] needs some judicious editing before the package will build on a typical Linux system. Finally, three of the six widgets offered do not work (and, in fact, segfault).

The _dialog_ family of tools offers a method of calling "dialog" widgets from a shell script. The original _dialog_ utility works in a text console, but its successors, _gdialog_, _Xdialog_, and _kdialog_ use X-Windows-based widget sets.

![[Example 36-22|Example 36-22]]

The [[miscellaneous-commands#^XMESSAGEREF|xmessage]] command is a simple method of popping up a message/query window. For example:

```bash
xmessage Fatal error in script! -button exit
```

The latest entry in the widget sweepstakes is [[miscellaneous-commands#^ZENITYREF|zenity]]. This utility pops up _GTK+_ dialog widgets-and-windows, and it works very nicely within a script.

```bash
get_info ()
{
  zenity --entry       #  Pops up query window . . .
                       #+ and prints user entry to stdout.

                       #  Also try the --calendar and --scale options.
}

answer=$( get_info )   #  Capture stdout in $answer variable.

echo "User entered: "$answer""
```

For other methods of scripting with widgets, try _Tk_ or _wish_ (_Tcl_ derivatives), _PerlTk_ (_Perl_ with _Tk_ extensions), _tksh_ (_ksh_ with _Tk_ extensions), _XForms4Perl_ (_Perl_ with _XForms_ extensions), _Gtk-Perl_ (_Perl_ with _Gtk_ extensions), or _PyQt_ (_Python_ with _Qt_ extensions).
