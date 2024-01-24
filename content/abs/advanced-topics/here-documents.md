---
title: 19. Here Documents
---


> Here and now, boys.
>
>--<cite>Aldous Huxley, _Island_</cite>

A _here document_ is a special-purpose code block. It uses a form of [[./io-redirection|I/O redirection]] to feed a command list to an interactive program or a command, such as [[../commands/communications-commands#^FTPREF|ftp]], [[../commands/basic-commands#^CATREF|cat]], or the _ex_ text editor.

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

Note that _here documents_ may sometimes be used to good effect with non-interactive utilities and commands, such as, for example, [[../commands/system-and-administrative-commands#^WALLREF|wall]].

###### Example 19-1. *broadcast*: Sends message to everyone logged in

```bash
#!/bin/bash

wall <<zzz23EndOfMessagezzz23
E-mail your noontime orders for pizza to the system administrator.
    (Add an extra dollar for anchovy or mushroom topping.)
# Additional message text goes here.
# Note: 'wall' prints comment lines.
zzz23EndOfMessagezzz23

# Could have been done more efficiently by
#         wall <message-file
#  However, embedding the message template in a script
#+ is a quick-and-dirty one-off solution.

exit
```

Even such unlikely candidates as the _vi_ text editor lend themselves to _here documents_.

###### Example 19-2. *dummyfile*: Creates a 2-line dummy file

```bash
#!/bin/bash

# Noninteractive use of 'vi' to edit a file.
# Emulates 'sed'.

E_BADARGS=85

if [ -z "$1" ]
then
  echo "Usage: `basename $0` filename"
  exit $E_BADARGS
fi

TARGETFILE=$1

# Insert 2 lines in file, then save.
#--------Begin here document-----------#
vi $TARGETFILE <<x23LimitStringx23
i
This is line 1 of the example file.
This is line 2 of the example file.
^[
ZZ
x23LimitStringx23
#----------End here document-----------#

#  Note that ^[ above is a literal escape
#+ typed by Control-V <Esc>.

#  Bram Moolenaar points out that this may not work with 'vim'
#+ because of possible problems with terminal interaction.

exit
```

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

###### Example 19-3. Multi-line message using *cat*

```bash
#!/bin/bash

#  'echo' is fine for printing single line messages,
#+  but somewhat problematic for for message blocks.
#   A 'cat' here document overcomes this limitation.

cat <<End-of-message
-------------------------------------
This is line 1 of the message.
This is line 2 of the message.
This is line 3 of the message.
This is line 4 of the message.
This is the last line of the message.
-------------------------------------
End-of-message

#  Replacing line 7, above, with
#+   cat > $Newfile <<End-of-message
#+       ^^^^^^^^^^
#+ writes the output to the file $Newfile, rather than to stdout.

exit 0


#--------------------------------------------
# Code below disabled, due to "exit 0" above.

# S.C. points out that the following also works.
echo "-------------------------------------
This is line 1 of the message.
This is line 2 of the message.
This is line 3 of the message.
This is line 4 of the message.
This is the last line of the message.
-------------------------------------"
# However, text may not include double quotes unless they are escaped.
```

The - option to mark a here document limit string (**<<-LimitString**) suppresses leading tabs (but not spaces) in the output. This may be useful in making a script more readable.

###### Example 19-4. Multi-line message, with tabs suppressed

```bash
#!/bin/bash
# Same as previous example, but...

#  The - option to a here document <<-
#+ suppresses leading tabs in the body of the document,
#+ but *not* spaces.

cat <<-ENDOFMESSAGE
	This is line 1 of the message.
	This is line 2 of the message.
	This is line 3 of the message.
	This is line 4 of the message.
	This is the last line of the message.
ENDOFMESSAGE
# The output of the script will be flush left.
# Leading tab in each line will not show.

# Above 5 lines of "message" prefaced by a tab, not spaces.
# Spaces not affected by   <<-  .

# Note that this option has no effect on *embedded* tabs.

exit 0
```

A _here document_ supports parameter and command substitution. It is therefore possible to pass different parameters to the body of the here document, changing its output accordingly.

###### Example 19-5. Here document with replaceable parameters

```bash
#!/bin/bash
# Another 'cat' here document, using parameter substitution.

# Try it with no command-line parameters,   ./scriptname
# Try it with one command-line parameter,   ./scriptname Mortimer
# Try it with one two-word quoted command-line parameter,
#                           ./scriptname "Mortimer Jones"

CMDLINEPARAM=1     #  Expect at least command-line parameter.

if [ $# -ge $CMDLINEPARAM ]
then
  NAME=$1          #  If more than one command-line param,
                   #+ then just take the first.
else
  NAME="John Doe"  #  Default, if no command-line parameter.
fi  

RESPONDENT="the author of this fine script"  
  

cat <<Endofmessage

Hello, there, $NAME.
Greetings to you, $NAME, from $RESPONDENT.

# This comment shows up in the output (why?).

Endofmessage

# Note that the blank lines show up in the output.
# So does the comment.

exit
```

This is a useful script containing a _here document_ with parameter substitution.

###### Example 19-6. Upload a file pair to *Sunsite* incoming directory

```bash
#!/bin/bash
# upload.sh

#  Upload file pair (Filename.lsm, Filename.tar.gz)
#+ to incoming directory at Sunsite/UNC (ibiblio.org).
#  Filename.tar.gz is the tarball itself.
#  Filename.lsm is the descriptor file.
#  Sunsite requires "lsm" file, otherwise will bounce contributions.


E_ARGERROR=85

if [ -z "$1" ]
then
  echo "Usage: `basename $0` Filename-to-upload"
  exit $E_ARGERROR
fi  


Filename=`basename $1`           # Strips pathname out of file name.

Server="ibiblio.org"
Directory="/incoming/Linux"
#  These need not be hard-coded into script,
#+ but may instead be changed to command-line argument.

Password="your.e-mail.address"   # Change above to suit.

ftp -n $Server <<End-Of-Session
# -n option disables auto-logon

user anonymous "$Password"       #  If this doesn't work, then try:
                                 #  quote user anonymous "$Password"
binary
bell                             # Ring 'bell' after each file transfer.
cd $Directory
put "$Filename.lsm"
put "$Filename.tar.gz"
bye
End-Of-Session

exit 0
```

Quoting or escaping the "limit string" at the head of a here document disables parameter substitution within its body. The reason for this is that _quoting/escaping the limit string_ effectively [[../basic/quoting#^ESCP|escapes]] the $, `, and \ [[special-characters#^SCHARLIST|special characters]], and causes them to be interpreted literally. (Thank you, Allen Halsey, for pointing this out.)

###### Example 19-7. Parameter substitution turned off

```bash
#!/bin/bash
#  A 'cat' here-document, but with parameter substitution disabled.

NAME="John Doe"
RESPONDENT="the author of this fine script"  

cat <<'Endofmessage'

Hello, there, $NAME.
Greetings to you, $NAME, from $RESPONDENT.

Endofmessage

#   No parameter substitution when the "limit string" is quoted or escaped.
#   Either of the following at the head of the here document would have
#+  the same effect.
#   cat <<"Endofmessage"
#   cat <<\Endofmessage



#   And, likewise:

cat <<"SpecialCharTest"

Directory listing would follow
if limit string were not quoted.
`ls -l`

Arithmetic expansion would take place
if limit string were not quoted.
$((5 + 3))

A a single backslash would echo
if limit string were not quoted.
\\

SpecialCharTest


exit
```

Disabling parameter substitution permits outputting literal text. Generating scripts or even program code is one use for this.

###### Example 19-8. A script that generates another script

```bash
#!/bin/bash
# generate-script.sh
# Based on an idea by Albert Reiner.

OUTFILE=generated.sh         # Name of the file to generate.


# -----------------------------------------------------------
# 'Here document containing the body of the generated script.
(
cat <<'EOF'
#!/bin/bash

echo "This is a generated shell script."
#  Note that since we are inside a subshell,
#+ we can't access variables in the "outside" script.

echo "Generated file will be named: $OUTFILE"
#  Above line will not work as normally expected
#+ because parameter expansion has been disabled.
#  Instead, the result is literal output.

a=7
b=3

let "c = $a * $b"
echo "c = $c"

exit 0
EOF
) > $OUTFILE
# -----------------------------------------------------------

#  Quoting the 'limit string' prevents variable expansion
#+ within the body of the above 'here document.'
#  This permits outputting literal strings in the output file.

if [ -f "$OUTFILE" ]
then
  chmod 755 $OUTFILE
  # Make the generated file executable.
else
  echo "Problem in creating file: \"$OUTFILE\""
fi

#  This method also works for generating
#+ C programs, Perl programs, Python programs, Makefiles,
#+ and the like.

exit 0
```

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

###### Example 19-9. Here documents and functions

```bash
#!/bin/bash
# here-function.sh

GetPersonalData ()
{
  read firstname
  read lastname
  read address
  read city 
  read state 
  read zipcode
} # This certainly appears to be an interactive function, but . . .


# Supply input to the above function.
GetPersonalData <<RECORD001
Bozo
Bozeman
2726 Nondescript Dr.
Bozeman
MT
21226
RECORD001


echo
echo "$firstname $lastname"
echo "$address"
echo "$city, $state $zipcode"
echo

exit 0
```

It is possible to use : as a dummy command accepting output from a here document. This, in effect, creates an "anonymous" here document.

###### Example 19-10. "Anonymous" Here Document

```bash
#!/bin/bash

: <<TESTVARIABLES
${HOSTNAME?}${USER?}${MAIL?}  # Print error message if one of the variables not set.
TESTVARIABLES

exit $?
```

> [!tip]
> A variation of the above technique permits "commenting out" blocks of code.

###### Example 19-11. Commenting out a block of code

```bash
#!/bin/bash
# commentblock.sh

: <<COMMENTBLOCK
echo "This line will not echo."
This is a comment line missing the "#" prefix.
This is another comment line missing the "#" prefix.

&*@!!++=
The above line will cause no error message,
because the Bash interpreter will ignore it.
COMMENTBLOCK

echo "Exit value of above \"COMMENTBLOCK\" is $?."   # 0
# No error shown.
echo


#  The above technique also comes in useful for commenting out
#+ a block of working code for debugging purposes.
#  This saves having to put a "#" at the beginning of each line,
#+ then having to go back and delete each "#" later.
#  Note that the use of of colon, above, is optional.

echo "Just before commented-out code block."
#  The lines of code between the double-dashed lines will not execute.
#  ===================================================================
: <<DEBUGXXX
for file in *
do
 cat "$file"
done
DEBUGXXX
#  ===================================================================
echo "Just after commented-out code block."

exit 0



######################################################################
#  Note, however, that if a bracketed variable is contained within
#+ the commented-out code block,
#+ then this could cause problems.
#  for example:


#/!/bin/bash

  : <<COMMENTBLOCK
  echo "This line will not echo."
  &*@!!++=
  ${foo_bar_bazz?}
  $(rm -rf /tmp/foobar/)
  $(touch my_build_directory/cups/Makefile)
COMMENTBLOCK


$ sh commented-bad.sh
commented-bad.sh: line 3: foo_bar_bazz: parameter null or not set

# The remedy for this is to strong-quote the 'COMMENTBLOCK' in line 49, above.

  : <<'COMMENTBLOCK'

# Thank you, Kurt Pfeifle, for pointing this out.
```

> [!tip]
> Yet another twist of this nifty trick makes "self-documenting" scripts possible.

###### Example 19-12. A self-documenting script

```bash
#!/bin/bash
# self-document.sh: self-documenting script
# Modification of "colm.sh".

DOC_REQUEST=70

if [ "$1" = "-h"  -o "$1" = "--help" ]     # Request help.
then
  echo; echo "Usage: $0 [directory-name]"; echo
  sed --silent -e '/DOCUMENTATIONXX$/,/^DOCUMENTATIONXX$/p' "$0" |
  sed -e '/DOCUMENTATIONXX$/d'; exit $DOC_REQUEST; fi


: <<DOCUMENTATIONXX
List the statistics of a specified directory in tabular format.
---------------------------------------------------------------
The command-line parameter gives the directory to be listed.
If no directory specified or directory specified cannot be read,
then list the current working directory.

DOCUMENTATIONXX

if [ -z "$1" -o ! -r "$1" ]
then
  directory=.
else
  directory="$1"
fi  

echo "Listing of "$directory":"; echo
(printf "PERMISSIONS LINKS OWNER GROUP SIZE MONTH DAY HH:MM PROG-NAME\n" \
; ls -l "$directory" | sed 1d) | column -t

exit 0
```

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

See also [[contributed-scripts#^ISSPAMMER2|Example A-28]], [[contributed-scripts#^PETALS|Example A-40]], [[contributed-scripts#^QKY|Example A-41]], and [[contributed-scripts#^NIM|Example A-42]] for more examples of self-documenting scripts.

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

[^1]: Except, as Dennis Benzinger points out, if [[here-documents#^LIMITSTRDASH|using **<<-** to suppress tabs]].
