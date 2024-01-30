---
title: 3. Special Characters
---

What makes a character *special*? If it has a meaning beyond its *literal meaning*, a [[brief-introduction-to-regular-expressions#^metameaningref|meta-meaning]], then we refer to it as a *special character*. Along with commands and [[internal-commands-and-builtins#^keywordref|keywords]], *special characters* are building blocks of Bash scripts.

## Special Characters Found In Scripts and Elsewhere

### \# (hash)

**Comments.** Lines beginning with a # (with the exception of [[sha-bang#^magnumref|\#!]]) are comments and will *not* be executed.

```bash
# This line is a comment.
```

Comments may also occur following the end of a command.

```bash
echo "A comment will follow." # Comment here.
#                            ^ Note whitespace before #
```

Comments may also follow [[#Whitespace|whitespace]] at the beginning of a line

```bash
    # A tab precedes this comment.
```

Comments may even be embedded within a [[special-characters#^PIPEREF|pipe]].

```bash
initial=( `cat "$startfile" | sed -e '/#/d' | tr -d '\n' |\
# Delete lines containing '#' comment character.
           sed -e 's/\./\. /g' -e 's/*/* /g'` )
# Excerpted from life.sh script
```

> [!caution]
> A command may not follow a comment on the same line. There is no method of terminating the comment, in order for "live code" to begin on the same line. Use a new line for the next command.

> [!note]
> Of course, a [[quoting#^QUOTINGREF|quoted]] or an [[quoting#^ESCP|escaped]] # in an [[internal-commands-and-builtins#^ECHOREF|echo]] statement does *not* begin a comment. Likewise, a # appears in [[parameter-substitution#^PSUB2|certain parameter-substitution constructs]] and in [[operations-and-related-topics#^NUMCONSTANTS|numerical constant expressions]].
>
> ```bash
> echo "The # here does not begin a comment."
> echo 'The # here does not begin a comment.'
> echo The \# here does not begin a comment.
> echo The # here begins a comment.
> 
> echo ${PATH#*:}       # Parameter substitution, not a comment.
> echo $(( 2#101011 ))  # Base conversion, not a comment.
> 
> # Thanks, S.C.
> ```
>
> The standard [[Chapter 5. Quoting#^QUOTINGREF|quoting and escape]] characters (" ' \) escape the #.

Certain [[parameter-substitution#^PSOREX1|pattern matching operations]] also use the #.

### ; (semicolon)

**Command separator \[semicolon].** Permits putting two or more commands on the same line.

```bash
echo hello; echo there


if [ -x "$filename" ]; then    #  Note the space after the semicolon.
#+                   ^^
  echo "File $filename exists."; cp $filename $filename.bak
else   #                       ^^
  echo "File $filename not found."; touch $filename
fi; echo "File test complete."
```

Note that the ";" [[complex-commands#^FINDREF0|sometimes needs to be *escaped*]].

### ;; (double semicolon)

**Terminator in a [[loops-and-branches#^CASEESAC1|case]] option \[double semicolon].**

```bash
case "$variable" in
  abc)  echo "\$variable = abc" ;;
  xyz)  echo "\$variable = xyz" ;;
esac
```

### ;;&, ;&

**[[bash-version-4#^NCTERM|Terminators]] in a *case* option ([[bash-version-4#^BASH4REF|version 4+]] of Bash).**

### . (dot)

. 

**"dot" command [[internal-commands-and-builtins#^SOURCEREF|period].** Equivalent to [source]] (see [[Example 15-22|Example 15-22]]). This is a bash [[internal-commands-and-builtins|builtin]].

.

**"dot", as a component of a filename.** When working with filenames, a leading dot is the prefix of a "hidden" file, a file that an [[basic-commands#^LSREF|ls]] will not normally show.

```bash
bash$ touch .hidden-file
bash$ ls -l	      
total 10
 -rw-r--r--    1 bozo      4034 Jul 18 22:04 data1.addressbook
 -rw-r--r--    1 bozo      4602 May 25 13:58 data1.addressbook.bak
 -rw-r--r--    1 bozo       877 Dec 17  2000 employment.addressbook


bash$ ls -al	      
total 14
 drwxrwxr-x    2 bozo  bozo      1024 Aug 29 20:54 ./
 drwx------   52 bozo  bozo      3072 Aug 29 20:51 ../
 -rw-r--r--    1 bozo  bozo      4034 Jul 18 22:04 data1.addressbook
 -rw-r--r--    1 bozo  bozo      4602 May 25 13:58 data1.addressbook.bak
 -rw-r--r--    1 bozo  bozo       877 Dec 17  2000 employment.addressbook
 -rw-rw-r--    1 bozo  bozo         0 Aug 29 20:54 .hidden-file
	        
```

When considering directory names, *a single dot* represents the current working directory, and *two dots* denote the parent directory.

```bash
bash$ pwd
/home/bozo/projects

bash$ cd .
bash$ pwd
/home/bozo/projects

bash$ cd ..
bash$ pwd
/home/bozo/
	        
```

The *dot* often appears as the destination (directory) of a file movement command, in this context meaning *current directory*.

```bash
bash$ cp /home/bozo/current_work/junk/* .
	        
```

Copy all the "junk" files to [[another-look-at-variables#^PWDREF|$PWD]].

.

**"dot" character match.** When [[brief-introduction-to-regular-expressions#^REGEXDOT|matching characters]], as part of a [[regexp#^REGEXREF|regular expression]], a "dot" [[brief-introduction-to-regular-expressions#^REGEXDOT|matches a single character]].

### " (double quote)

**[[variables-and-parameters#^DBLQUO|partial quoting]] [[quoting|double quote].** *"STRING"* preserves (from interpretation) most of the special characters within *STRING*. See [Chapter 5]].

### ' (single quote)

**[[variables-and-parameters#^SNGLQUO|full quoting]] [[quoting|single quote].** *'STRING'* preserves all special characters within *STRING*. This is a stronger form of quoting than *"STRING"*. See [Chapter 5]].

### , (comma)

**[[operations-and-related-topics#^COMMAOP|comma operator]].** The *comma operator* [^1] links together a series of arithmetic operations. All are evaluated, but only the last one is returned.

```bash
let "t2 = ((a = 9, 15 / 3))"
# Set "a = 9" and "t2 = 15 / 3"
```

The *comma* operator can also concatenate strings.

```bash
for file in /{,usr/}bin/*calc
#             ^    Find all executable files ending in "calc"
#+                 in /bin and /usr/bin directories.
do
        if [ -x "$file" ]
        then
          echo $file
        fi
done

# /bin/ipcalc
# /usr/bin/kcalc
# /usr/bin/oidcalc
# /usr/bin/oocalc


# Thank you, Rory Winston, for pointing this out.
```

### ,, ,

**[[bash-version-4#^CASEMODPARAMSUB|Lowercase conversion]] in *parameter substitution* (added in [[bash-version-4#^BASH4REF|version 4]] of Bash).**

### \ (backslash)

**[[quoting#^ESCP|escape]] [backslash].** A quoting mechanism for single characters.

**\X** *escapes* the character *X*. This has the effect of "quoting" *X*, equivalent to *'X'*. The \ may be used to quote " and ', so they are expressed literally.

See [[Chapter 5. Quoting|Chapter 5]] for an in-depth explanation of escaped characters.

### / (forward slash)

**Filename path separator [forward slash].** Separates the components of a filename (as in /home/bozo/projects/Makefile).

This is also the division [[operations-and-related-topics#^AROPS1|arithmetic operator]].

### \` (backtick)

**[[command-substitution#^COMMANDSUBREF|command substitution]].** The **`command`** construct makes available the output of **command** for assignment to a variable. This is also known as [[command-substitution#^BACKQUOTESREF|backquotes]] or backticks.

### : (colon)

**null command [[internal-commands-and-builtins#^TRUEREF|colon].** This is the shell equivalent of a "NOP" (*no op*, a do-nothing operation). It may be considered a synonym for the shell builtin [true]]. The ":" command is itself a *Bash* [[internal-commands-and-builtins|builtin]], and its [[exit-and-exit-status#^EXITSTATUSREF|exit status]] is *true* (0).

```bash
:
echo $?   # 0
```

Endless loop:

```bash
while :
do
   operation-1
   operation-2
   ...
   operation-n
done

# Same as:
#    while true
#    do
#      ...
#    done
```

Placeholder in if/then test:

```bash
if condition
then :   # Do nothing and branch ahead
else     # Or else ...
   take-some-action
fi
```

Provide a placeholder where a binary operation is expected, see [[Example 8-2]] and [[parameter-substitution#^DEFPARAM|default parameters]].

```bash
: ${username=`whoami`}
# ${username=`whoami`}   Gives an error without the leading :
#                        unless "username" is a command or builtin...

: ${1?"Usage: $0 ARGUMENT"}     # From "usage-message.sh example script.
```

Provide a placeholder where a command is expected in a [[here-documents#^HEREDOCREF|here document]]. See [[Example 19-10]].

Evaluate string of variables using [[parameter-substitution#^PARAMSUBREF|parameter substitution]] (as in [[Example 10-7]]).

```bash
: ${HOSTNAME?} ${USER?} ${MAIL?}
#  Prints error message
#+ if one or more of essential environmental variables not set.
```

**[[parameter-substitution#^EXPREPL1|Variable expansion / substring replacement]]**.

In combination with the > [[io-redirection|redirection operator]], truncates a file to zero length, without changing its permissions. If the file did not previously exist, creates it.

```bash
: > data.xxx   # File "data.xxx" now empty.	      

# Same effect as   cat /dev/null >data.xxx
# However, this does not fork a new process, since ":" is a builtin.
```

See also [[Example 16-15]].

In combination with the >> redirection operator, has no effect on a pre-existing target file (**: >> target_file**). If the file did not previously exist, creates it.

> [!note] This applies to regular files, not pipes, symlinks, and certain special files.

May be used to begin a comment line, although this is not recommended. Using # for a comment turns off error checking for the remainder of that line, so almost anything may appear in a comment. However, this is not the case with :.

```bash
: This is a comment that generates an error, ( if [ $x -eq 3] ).
```

The ":" serves as a [[special-characters#^FIELDREF|field]] separator, in [[important-files#^DATAFILESREF1|/etc/passwd]], and in the [[another-look-at-variables#^PATHREF|$PATH]] variable.

```bash
bash$ echo $PATH
/usr/local/bin:/bin:/usr/bin:/usr/X11R6/bin:/sbin:/usr/sbin:/usr/games
```

A *colon* is [[functions#^FSTRANGEREF|acceptable as a function name]].

```bash
:()
{
  echo "The name of this function is "$FUNCNAME" "
  # Why use a colon as a function name?
  # It's a way of obfuscating your code.
}

:

# The name of this function is :
```

This is not [[portability-issues|portable]] behavior, and therefore not a recommended practice. In fact, more recent releases of Bash do not permit this usage. An underscore **_** works, though.

A *colon* can serve as a placeholder in an otherwise empty function.

```bash
not_empty ()
{
  :
} # Contains a : (null command), and so is not empty.
```

### ! (exclamation mark)

**reverse (or negate) the sense of a test or exit status [[exit-and-exit-status#^EXITSTATUSREF|bang].** The ! operator inverts the [exit status]] of the command to which it is applied (see [[Example 6-2]]). It also inverts the meaning of a test operator. This can, for example, change the sense of *equal* ( [[other-comparison-operators#^EQUALSIGNREF|=]] ) to *not-equal* ( != ). The ! operator is a Bash [[internal-commands-and-builtins#^keywordref|keyword]].

In a different context, the ! also appears in [[indirect-references#^IVRREF|indirect variable references]].

In yet another context, from the *command line*, the ! invokes the Bash *history mechanism* (see [[history-commands.html|Appendix L]]). Note that within a script, the history mechanism is disabled.

### \* (asterisk)

*

**wild card [[globbing|asterisk]].** The * character serves as a "wild card" for filename expansion in [[globbing|18.2. Globbing]]. By itself, it matches every filename in a given directory.

```bash
bash$ echo *
abs-book.sgml add-drive.sh agram.sh alias.sh
	      
```

The * also represents [[brief-introduction-to-regular-expressions#^ASTERISKREG|any number (or zero) characters]] in a [[regexp#^REGEXREF|regular expression]].

*

**[[operations-and-related-topics#^AROPS1|arithmetic operator]].** In the context of arithmetic operations, the * denotes multiplication.

** A double asterisk can represent the [[operations-and-related-topics#^EXPONENTIATIONREF|exponentiation]] operator or [[bash-version-4#^GLOBSTARREF|extended file-match]] *globbing*.

### ? (question mark)

?

**test operator.** Within certain expressions, the ? indicates a test for a condition.

In a [[operations-and-related-topics.html|double-parentheses construct]], the ? can serve as an element of a C-style *trinary* operator. [^2]

condition**?**result-if-true**:**result-if-false

```bash
(( var0 = var1<98?9:21 ))
#                ^ ^

# if [ "$var1" -lt 98 ]
# then
#   var0=9
# else
#   var0=21
# fi
```

In a [[parameter-substitution#^PARAMSUBREF|parameter substitution]] expression, the ? [[parameter-substitution#^QERRMSG|tests whether a variable has been set]].

?

**wild card.** The ? character serves as a single-character "wild card" for filename expansion in [[globbing.html|globbing]], as well as [[brief-introduction-to-regular-expressions#^QUEXREGEX|representing one character]] in an [[brief-introduction-to-regular-expressions#^EXTREGEX|extended regular expression]].

### $ (dollar sign)

$

**[[varsubn.html|Variable substitution]] (contents of a variable).**

```bash
var1=5
var2=23skidoo

echo $var1     # 5
echo $var2     # 23skidoo
```

A $ prefixing a variable name indicates the *value* the variable holds.

$

**end-of-line.** In a [[regexp#^REGEXREF|regular expression]], a "$" addresses the [[brief-introduction-to-regular-expressions#^DOLLARSIGNREF|end of a line]] of text.

### ${}

**[[parameter-substitution#^PARAMSUBREF|Parameter substitution]].**

### $' ... '

**[[quoting#^STRQ|Quoted string expansion]].** This construct expands single or multiple escaped octal or hex values into ASCII [^3] or [[bash-version-4#^UNICODEREF|Unicode]] characters.

### \$\*, \$@

**[[another-look-at-variables#^APPREF|positional parameters]].**

### \$?

**exit status variable.** The [[exit-and-exit-status#^EXSREF|$? variable]] holds the [[exit-and-exit-status#^EXITSTATUSREF|exit status]] of a command, a [[functions|function]], or of the script itself.

### \$\$

**process ID variable.** The [[another-look-at-variables#^PROCCID|$$ variable]] holds the *process ID* [^4] of the script in which it appears.

### ()

**command group.**

```bash
(a=hello; echo $a)
```

> [!important]
> A listing of commands within *parentheses* starts a [[subshells#^SUBSHELLSREF|subshell]].
>
> Variables inside parentheses, within the subshell, are not visible to the rest of the script. The parent process, the script, [[subshells#^PARVIS|cannot read variables created in the child process]], the subshell.
>
> ```bash
> a=123
> ( a=321; )	      
> 
> echo "a = $a"   # a = 123
> # "a" within parentheses acts like a local variable.
> ```

**array initialization.**

```bash
Array=(element1 element2 element3)
```

### {xxx,yyy,zzz,...}

**Brace expansion.** 

```bash
echo \"{These,words,are,quoted}\"   # " prefix and suffix
# "These" "words" "are" "quoted"


cat {file1,file2,file3} > combined_file
# Concatenates the files file1, file2, and file3 into combined_file.

cp file22.{txt,backup}
# Copies "file22.txt" to "file22.backup"
```

A command may act upon a comma-separated list of file specs within *braces*. [^5] Filename expansion ([globbingref.html|[globbing]]) applies to the file specs between the braces.

> [!caution]
> No spaces allowed within the braces *unless* the spaces are quoted or escaped.
>
> `echo {file1,file2}\ :{\ A," B",' C'}`
>
> file1 : A file1 : B file1 : C file2 : A file2 : B file2 : C|

### {a..z}

**Extended Brace expansion.**

```bash
echo {a..z} # a b c d e f g h i j k l m n o p q r s t u v w x y z
# Echoes characters between a and z.

echo {0..3} # 0 1 2 3
# Echoes characters between 0 and 3.


base64_charset=( {A..Z} {a..z} {0..9} + / = )
# Initializing an array, using extended brace expansion.
# From vladz's "base64.sh" example script.
```

The *{a..z}* [[bashver3#^BRACEEXPREF3|extended brace expansion]] construction is a feature introduced in [[bashver3#^BASH3REF|version 3]] of *Bash*.

### {}

**Block of code \[curly brackets].** Also referred to as an *inline group*, this construct, in effect, creates an *anonymous function* (a function without a name). However, unlike in a "standard" [[functions|function]], the variables inside a code block remain visible to the remainder of the script. [[#^ex3-2]]

```bash
bash$ { local a;
	      a=123; }
bash: local: can only be used in a
function
	      
```

```bash
a=123
{ a=321; }
echo "a = $a"   # a = 321   (value inside code block)

# Thanks, S.C.
```

The code block enclosed in braces may have [[io-redirection|I/O redirected]] to and from it. 

![[Example 3-1]]

![[Example 3-2]]

> [!note]
> Unlike a command group within (parentheses), as above, a code block enclosed by {braces} will *not* normally launch a [[subshells#^SUBSHELLSREF|subshell]]. [^6]
>
> It is possible to [[loops-and-branches#^ITERATIONREF|iterate]] a code block using a [[loops-and-branches#^NODODONE|non-standard *for-loop*]].

### {}

**placeholder for text.** Used after [[complex-commands#^XARGSCURLYREF|xargs -i]] (*replace strings* option). The {} double curly brackets are a placeholder for output text.

```bash
ls . | xargs -i -t cp ./{} $1
#            ^^         ^^

# From "ex42.sh" (copydir.sh) example.
```

### {} \;

**pathname.** Mostly used in [[complex-commands#^FINDREF|find]] constructs. This is *not* a shell [[internal-commands-and-builtins|builtin]].

> Definition: A *pathname* is a *filename* that includes the complete [[another-look-at-variables#^PATHREF|path]]. As an example, /home/bozo/Notes/Thursday/schedule.txt. This is sometimes referred to as the *absolute path*.

> [!note] The ";" ends the -exec option of a **find** command sequence. It needs to be escaped to protect it from interpretation by the shell.

### \[ ]

**test.**

[[tests#^IFTHEN|Test]] expression between **`[ ]`**. Note that **[** is part of the shell *builtin* [[tests#^TTESTREF|test]] (and a synonym for it), *not* a link to the external command /usr/bin/test.

### \[\[ \]\]

**test.**

Test expression between `[[[ ]]`. More flexible than the single-bracket [ ] test, this is a shell [internal#^keywordref|keyword]].

See the discussion on the [[tests#^DBLBRACKETS|`[[ ... ]]` construct]].

### \[ ]

**array element.**

In the context of an [[arrays#^ARRAYREF|array]], brackets set off the numbering of each element of that array.

```bash
Array[1]=slot_1
echo ${Array[1]}
```

### \[ ]

**range of characters.**

As part of a [[regexp#^REGEXREF|regular expression]], brackets delineate a [[brief-introduction-to-regular-expressions#^BRACKETSREF|range of characters]] to match.

### \$\[ ... ]

**integer expansion.**

Evaluate integer expression between $[ ].

```bash
a=3
b=7

echo $[$a+$b]   # 10
echo $[$a*$b]   # 21
```

Note that this usage is *deprecated*, and has been replaced by the [[operations-and-related-topics.html|(( ... ))]] construct.

### (( ))

**integer expansion.**

Expand and evaluate integer expression between (( )).

See the discussion on the [[operations-and-related-topics.html|(( ... )) construct]].

### > &> >& >> < <>

**[[io-redirection|redirection]].**

**scriptname >filename** redirects the output of scriptname to file filename. Overwrite filename if it already exists.

**command &>filename** redirects both the [[a-detailed-introduction-to-io-and-io-redirection#^STDINOUTDEF|stdout]] and the stderr of command to filename.

> [!note]
> This is useful for suppressing output when testing for a condition. For example, let us test whether a certain command exists.
>
> ```bash
> bash$ type bogus_command &>/dev/null
> 
> 
> 
> bash$ echo $?
> 1
>           
> ```
>
> Or in a script:
>
> ```bash
> command_test () { type "$1" &>/dev/null; }
> #                                      ^
> 
> cmd=rmdir            # Legitimate command.
> command_test $cmd; echo $?   # 0
> 
> 
> cmd=bogus_command    # Illegitimate command
> command_test $cmd; echo $?   # 1
> ```

**command >&2** redirects stdout of command to stderr.

**scriptname >>filename** appends the output of scriptname to file filename. If filename does not already exist, it is created.

**[[io-redirection#^FDREF|i]<>filename** opens file filename for reading and writing, and assigns [file descriptor]] i to it. If filename does not exist, it is created.

**[[process-sub#^PROCESSSUBREF|process substitution]].**

**(command)>**

**<(command)**

[[other-comparison-operators#^LTREF|In a different context]], the "<" and ">" characters act as [[other-comparison-operators#^SCOMPARISON1|string comparison operators]].

[[other-comparison-operators#^INTLT|In yet another context]], the "<" and ">" characters act as [[other-comparison-operators#^ICOMPARISON1|integer comparison operators]]. See also [[Example 16-9]].

### <<

**redirection used in a [[here-documents#^HEREDOCREF|here document]].**

### <<<

**redirection used in a [[here-strings#^HERESTRINGSREF|here string]].**

### <, >

**[[other-comparison-operators#^LTREF|ASCII comparison]].**

```bash
veg1=carrots
veg2=tomatoes

if [[ "$veg1" < "$veg2" | "$veg1" < "$veg2" ]]
then
  echo "Although $veg1 precede $veg2 in the dictionary,"
  echo -n "this does not necessarily imply anything "
  echo "about my culinary preferences."
else
  echo "What kind of dictionary are you using, anyhow?"
fi
```

### \<, \>

**[[brief-introduction-to-regular-expressions#^ANGLEBRAC|word boundary]] in a [[regexp#^REGEXREF|regular expression]].**

`bash$ **grep '\<the\>' textfile**`

### |

**pipe.** Passes the output (stdout) of a previous command to the input (stdin) of the next one, or to the shell. This is a method of chaining commands together.

```bash
echo ls -l | sh
#  Passes the output of "echo ls -l" to the shell,
#+ with the same result as a simple "ls -l".


cat *.lst | sort | uniq
# Merges and sorts all ".lst" files, then deletes duplicate lines.
```

> A pipe, as a classic method of interprocess communication, sends the stdout of one [[special-characters#^PROCESSREF|process]] to the stdin of another. In a typical case, a command, such as [[basic#^CATREF|cat]] or [[internal-commands-and-builtins#^ECHOREF|echo]], pipes a stream of data to a *filter*, a command that transforms its input for processing. [^7]
>
> `cat $filename1 $filename2 \| grep $search_word`
>
> For an interesting note on the complexity of using UNIX pipes, see [the UNIX FAQ, Part 3](http://www.faqs.org/faqs/unix-faq/faq/part3/).

The output of a command or commands may be piped to a script.

```bash
#!/bin/bash
# uppercase.sh : Changes input to uppercase.

tr 'a-z' 'A-Z'
#  Letter ranges must be quoted
#+ to prevent filename generation from single-letter filenames.

exit 0
```

Now, let us pipe the output of **ls -l** to this script.

```bash
bash$ ls -l | ./uppercase.sh
-RW-RW-R--    1 BOZO  BOZO       109 APR  7 19:49 1.TXT
 -RW-RW-R--    1 BOZO  BOZO       109 APR 14 16:48 2.TXT
 -RW-R--R--    1 BOZO  BOZO       725 APR 20 20:56 DATA-FILE
	      
```

> [!note]
> The stdout of each process in a pipe must be read as the stdin of the next. If this is not the case, the data stream will *block*, and the pipe will not behave as expected.
>
> ```bash
> cat file1 file2 | ls -l | sort
> # The output from "cat file1 file2" disappears.
> ```
>
> A pipe runs as a [[othertypesv#^CHILDREF|child process]], and therefore cannot alter script variables.
>
> ```bash
> variable="initial_value"
> echo "new_value" | read variable
> echo "variable = $variable"     # variable = initial_value
> ```
>
> If one of the commands in the pipe aborts, this prematurely terminates execution of the pipe. Called a *broken pipe*, this condition sends a *SIGPIPE* [[debugging#^SIGNALD|signal]].

### >|

**force redirection (even if the [[options#^NOCLOBBERREF|noclobber option]] is set).** This will forcibly overwrite an existing file.

### ||

**[[operations-and-related-topics#^ORREF|OR logical operator]].** In a [[tests#^TESTCONSTRUCTS1|test construct]], the || operator causes a return of 0 (success) if *either* of the linked test conditions is true.

### &

**Run job in background.** A command followed by an & will run in the background.

```bash
bash$ sleep 10 &
[1] 850
[1]+  Done                    sleep 10
	      
```

Within a script, commands and even [[loops-and-branches#^FORLOOPREF1|loops]] may run in the background.

![[Example 3-3]]

> [!caution] A command run in the background within a script may cause the script to hang, waiting for a keystroke. Fortunately, there is a [[internal-commands-and-builtins#^WAITHANG|remedy]] for this.|

### &&

**[[operations-and-related-topics#^LOGOPS1|AND logical operator]].** In a [[tests#^TESTCONSTRUCTS1|test construct]], the && operator causes a return of 0 (success) only if *both* the linked test conditions are true.

### -

**option, prefix.** Option flag for a command or filter. Prefix for an operator. Prefix for a [[parameter-substitution#^DEFPARAM1|default parameter]] in [[parameter-substitution#^PARAMSUBREF|parameter substitution]].

**COMMAND -[Option1][Option2][...]**

**ls -al**

**sort -dfu $filename**

```bash
if [ $file1 -ot $file2 ]
then #      ^
  echo "File $file1 is older than $file2."
fi

if [ "$a" -eq "$b" ]
then #    ^
  echo "$a is equal to $b."
fi

if [ "$c" -eq 24 -a "$d" -eq 47 ]
then #    ^              ^
  echo "$c equals 24 and $d equals 47."
fi


param2=${param1:-$DEFAULTVAL}
#               ^
```

**--**

The *double-dash* -- prefixes *long* (verbatim) options to commands.

**sort --ignore-leading-blanks**

Used with a [[internal-commands-and-builtins|Bash builtin]], it means the *end of options* to that particular command.

> [!tip]
> This provides a handy means of removing files whose *names begin with a dash*.
>
> ```bash
> bash$ ls -l
> -rw-r--r-- 1 bozo bozo 0 Nov 25 12:29 -badname
> 
> 
> bash$ rm -- -badname
> 
> bash$ ls -l
> total 0
> ```

The *double-dash* is also used in conjunction with [[internal-commands-and-builtins#^SETREF|set]].

**set -- $variable** (as in [[Example 15-18]])

### -

**redirection from/to stdin or stdout [dash].**

```bash
bash$ **cat -**
**abc**
abc

...

**Ctl-D**
```

As expected, **cat -** echoes stdin, in this case keyboarded user input, to stdout. But, does I/O redirection using **-** have real-world applications?

```bash
(cd /source/directory && tar cf - . ) | (cd /dest/directory && tar xpvf -)
# Move entire file tree from one directory to another
# [courtesy Alan Cox <a.cox@swansea.ac.uk>, with a minor change]

# 1) cd /source/directory
#    Source directory, where the files to be moved are.
# 2) &&
#   "And-list": if the 'cd' operation successful,
#    then execute the next command.
# 3) tar cf - .
#    The 'c' option 'tar' archiving command creates a new archive,
#    the 'f' (file) option, followed by '-' designates the target file
#    as stdout, and do it in current directory tree ('.').
# 4) |
#    Piped to ...
# 5) ( ... )
#    a subshell
# 6) cd /dest/directory
#    Change to the destination directory.
# 7) &&
#   "And-list", as above
# 8) tar xpvf -
#    Unarchive ('x'), preserve ownership and file permissions ('p'),
#    and send verbose messages to stdout ('v'),
#    reading data from stdin ('f' followed by '-').
#
#    Note that 'x' is a command, and 'p', 'v', 'f' are options.
#
# Whew!



# More elegant than, but equivalent to:
#   cd source/directory
#   tar cf - . | (cd ../dest/directory; tar xpvf -)
#
#     Also having same effect:
# cp -a /source/directory/* /dest/directory
#     Or:
# cp -a /source/directory/* /source/directory/.[^.]* /dest/directory
#     If there are hidden files in /source/directory.
```

```bash
bunzip2 -c linux-2.6.16.tar.bz2 | tar xvf -
#  --uncompress tar file--      | --then pass it to "tar"--
#  If "tar" has not been patched to handle "bunzip2",
#+ this needs to be done in two discrete steps, using a pipe.
#  The purpose of the exercise is to unarchive "bzipped" kernel source.
```

Note that in this context the "-" is not itself a Bash operator, but rather an option recognized by certain UNIX utilities that write to stdout, such as **tar**, **cat**, etc.

```bash
bash$ echo "whatever" | cat -
whatever 
```

Where a filename is expected, *-* redirects output to stdout (sometimes seen with **tar cf**), or accepts input from stdin, rather than from a file. This is a method of using a file-oriented utility as a filter in a pipe.

```bash
bash$ file
Usage: file [-bciknvzL] [-f namefile] [-m magicfiles] file...
	      
```

By itself on the command-line, [[file-and-archiving-commands#^FILEREF|file]] fails with an error message.

Add a "-" for a more useful result. This causes the shell to await user input.

```bash
bash$ file -
abc
standard input:              ASCII text



bash$ file -
#!/bin/bash
standard input:              Bourne-Again shell script text executable
	      
```

Now the command accepts input from stdin and analyzes it.

The "-" can be used to pipe stdout to other commands. This permits such stunts as [[assortedtips#^PREPENDREF|prepending lines to a file]].

Using [[file-and-archiving-commands#^DIFFREF|diff]] to compare a file with a *section* of another:

**grep Linux file1 | diff file2 -**

Finally, a real-world example using *-* with [[file-and-archiving-commands#^TARREF|tar]].

![[Example 3-4]]

> [!caution]
> Filenames beginning with "-" may cause problems when coupled with the "-" redirection operator. A script should check for this and add an appropriate prefix to such filenames, for example ./-FILENAME, $PWD/-FILENAME, or $PATHNAME/-FILENAME.
>
> If the value of a variable begins with a *-*, this may likewise create problems.
>
> ```bash
> var="-n"
> echo $var		
> # Has the effect of "echo -n", and outputs nothing.
> ```

### -

**previous working directory.** A **cd -** command changes to the previous working directory. This uses the [[another-look-at-variables#^OLDPWD|$OLDPWD]] [[othertypesv#^ENVREF|environmental variable]].

> [!caution] Do not confuse the "-" used in this sense with the "-" redirection operator just discussed. The interpretation of the "-" depends on the context in which it appears.

### -

**Minus.** Minus sign in an [[operations-and-related-topics#^AROPS1|arithmetic operation]].

### =

**Equals.** [[variables-and-parameters#^EQREF|Assignment operator]]

```bash
a=28
echo $a   # 28
```

In a [[other-comparison-operators#^EQUALSIGNREF|different context]], the "=" is a [[other-comparison-operators#^SCOMPARISON1|string comparison]] operator.

### +

**Plus.** Addition [[operations-and-related-topics#^AROPS1|arithmetic operator]].

In a [[brief-introduction-to-regular-expressions#^PLUSREF|different context]], the + is a [[regexp.html|Regular Expression]] operator.

### +

**Option.** Option flag for a command or filter.

Certain commands and [[internal-commands-and-builtins|builtins]] use the + to enable certain options and the - to disable them. In [[parameter-substitution#^PARAMSUBREF|parameter substitution]], the + prefixes an [[parameter-substitution#^PARAMALTV|alternate value]] that a variable expands to.

### %

**[[operations-and-related-topics#^MODULOREF|modulo]].** Modulo (remainder of a division) [[operations-and-related-topics#^AROPS1|arithmetic operation]].

```bash
let "z = 5 % 3"
echo $z  # 2
```

In a [[parameter-substitution#^PCTPATREF|different context]], the % is a [[parameter-substitution#^PSUB2|pattern matching]] operator.

### ~

**home directory [[another-look-at-variables#^HOMEDIRREF|tilde].** This corresponds to the [$HOME]] internal variable. ~bozo is bozo's home directory, and **ls ~bozo** lists the contents of it. ~/ is the current user's home directory, and **ls ~/** lists the contents of it.

```bash
bash$ echo ~bozo
/home/bozo

bash$ echo ~
/home/bozo

bash$ echo ~/
/home/bozo/

bash$ echo ~:
/home/bozo:

bash$ echo ~nonexistent-user
~nonexistent-user
	      
```

### ~+

**current working directory.** This corresponds to the [[another-look-at-variables#^PWDREF|$PWD]] internal variable.

### ~-

**previous working directory.** This corresponds to the [[another-look-at-variables#^OLDPWD|$OLDPWD]] internal variable.

### =~

**[[bashver3#^REGEXMATCHREF|regular expression match]].** This operator was introduced with [[bashver3#^BASH3REF|version 3]] of Bash.

### ^

**beginning-of-line.** In a [[regexp#^REGEXREF|regular expression]], a "^" addresses the [[brief-introduction-to-regular-expressions#^CARETREF|beginning of a line]] of text.

### ^, ^^

**[[bashver4#^CASEMODPARAMSUB|Uppercase conversion]] in *parameter substitution* (added in [[bashver4#^BASH4REF|version 4]] of Bash).**

## Control Characters

**change the behavior of the terminal or text display.** A control character is a `CONTROL + key` combination (pressed simultaneously). A control character may also be written in *octal* or *hexadecimal* notation, following an *escape*.

Control characters are not normally useful inside a script.

### `Ctl-A`

Moves cursor to beginning of line of text (on the command-line).

### `Ctl-B`

**Backspace** (nondestructive).

### `Ctl-C`

**Break**. Terminate a foreground job.

### `Ctl-D`

*Log out* from a shell (similar to [[exit-and-exit-status|exit]]).

**EOF** (end-of-file). This also terminates input from stdin.

When typing text on the console or in an *xterm* window, `Ctl-D` erases the character under the cursor. When there are no characters present, `Ctl-D` logs out of the session, as expected. In an *xterm* window, this has the effect of closing the window.

### `Ctl-E`

Moves cursor to end of line of text (on the command-line).

### `Ctl-F`

Moves cursor forward one character position (on the command-line).

### `Ctl-G`

**BEL**. On some old-time teletype terminals, this would actually ring a bell. In an *xterm* it might beep.

### `Ctl-H`

**Rubout** (destructive backspace). Erases characters the cursor backs over while backspacing.

```bash
#!/bin/bash
# Embedding Ctl-H in a string.

a="^H^H"                  # Two Ctl-H's -- backspaces
                          # ctl-V ctl-H, using vi/vim
echo "abcdef"             # abcdef
echo
echo -n "abcdef$a "       # abcd f
#  Space at end  ^              ^  Backspaces twice.
echo
echo -n "abcdef$a"        # abcdef
#  No space at end               ^ Doesn't backspace (why?).
                          # Results may not be quite as expected.
echo; echo

# Constantin Hagemeier suggests trying:
# a=$'\010\010'
# a=$'\b\b'
# a=$'\x08\x08'
# But, this does not change the results.

########################################

# Now, try this.

rubout="^H^H^H^H^H"       # 5 x Ctl-H.

echo -n "12345678"
sleep 2
echo -n "$rubout"
sleep 2
```

### `Ctl-I`

**Horizontal tab**.

### `Ctl-J`

**Newline** (line feed). In a script, may also be expressed in octal notation -- '\012' or in hexadecimal -- '\x0a'.

### `Ctl-K`

**Vertical tab**.

When typing text on the console or in an *xterm* window, `Ctl-K` erases from the character under the cursor to end of line. Within a script, `Ctl-K` may behave differently, as in Lee Lee Maschmeyer's example, below.

### `Ctl-L`

**Formfeed** (clear the terminal screen). In a terminal, this has the same effect as the [[terminal-control-commands#^CLEARREF|clear]] command. When sent to a printer, a `Ctl-L` causes an advance to end of the paper sheet.
    
- **Ctl-M**
    
    **Carriage return**.
    
```bash
#!/bin/bash
# Thank you, Lee Maschmeyer, for this example.

read -n 1 -s -p \
$'Control-M leaves cursor at beginning of this line. Press Enter. \x0d'
           # Of course, '0d' is the hex equivalent of Control-M.
echo >&2   #  The '-s' makes anything typed silent,
           #+ so it is necessary to go to new line explicitly.

read -n 1 -s -p $'Control-J leaves cursor on next line. \x0a'
           #  '0a' is the hex equivalent of Control-J, linefeed.
echo >&2

###

read -n 1 -s -p $'And Control-K\x0bgoes straight down.'
echo >&2   #  Control-K is vertical tab.

# A better example of the effect of a vertical tab is:

var=$'\x0aThis is the bottom line\x0bThis is the top line\x0a'
echo "$var"
#  This works the same way as the above example. However:
echo "$var" | col
#  This causes the right end of the line to be higher than the left end.
#  It also explains why we started and ended with a line feed --
#+ to avoid a garbled screen.

# As Lee Maschmeyer explains:
# --------------------------
#  In the [first vertical tab example] . . . the vertical tab
#+ makes the printing go straight down without a carriage return.
#  This is true only on devices, such as the Linux console,
#+ that can't go "backward."
#  The real purpose of VT is to go straight UP, not down.
#  It can be used to print superscripts on a printer.
#  The col utility can be used to emulate the proper behavior of VT.

exit 0
```
    
- **Ctl-N**
    
    Erases a line of text recalled from *history buffer* [^8] (on the command-line).
    
- **Ctl-O**
    
    Issues a *newline* (on the command-line).
    
- **Ctl-P**
    
    Recalls last command from *history buffer* (on the command-line).
    
- **Ctl-Q**
    
    Resume (**XON**).
    
    This resumes stdin in a terminal.
    
- **Ctl-R**
    
    Backwards search for text in *history buffer* (on the command-line).
    
- **Ctl-S**
    
    Suspend (**XOFF**).
    
    This freezes stdin in a terminal. (Use Ctl-Q to restore input.)
    
- **Ctl-T**
    
    Reverses the position of the character the cursor is on with the previous character (on the command-line).
    
- **Ctl-U**
    
    Erase a line of input, from the cursor backward to beginning of line. In some settings, **Ctl-U** erases the entire line of input, *regardless of cursor position*.
    
- **Ctl-V**
    
    When inputting text, **Ctl-V** permits inserting control characters. For example, the following two are equivalent:
    
```bash
echo -e '\x0a'
echo <Ctl-V><Ctl-J>
```
    
    **Ctl-V** is primarily useful from within a text editor.
    
- **Ctl-W**
    
    When typing text on the console or in an xterm window, **Ctl-W** erases from the character under the cursor backwards to the first instance of [[#Whitespace|whitespace]]. In some settings, **Ctl-W** erases backwards to first non-alphanumeric character.
    
- **Ctl-X**
    
    In certain word processing programs, *Cuts* highlighted text and copies to *clipboard*.
    
- **Ctl-Y**
    
    *Pastes* back text previously erased (with **Ctl-U** or **Ctl-W**).
    
- **Ctl-Z**
    
    *Pauses* a foreground job.
    
    *Substitute* operation in certain word processing applications.
    
    **EOF** (end-of-file) character in the MSDOS filesystem.
    

### Whitespace

**functions as a separator between commands and/or variables.** Whitespace consists of either *spaces*, *tabs*, *blank lines*, or any combination thereof. [^9] In some contexts, such as [[gotchas#^WSBAD|variable assignment]], whitespace is not permitted, and results in a syntax error.

Blank lines have no effect on the action of a script, and are therefore useful for visually separating functional sections.

[[another-look-at-variables#^IFSREF|$IFS]], the special variable separating *fields* of input to certain commands. It defaults to whitespace.

> **Definition:** A *field* is a discrete chunk of data expressed as a string of consecutive characters. Separating each field from adjacent fields is either *whitespace* or some other designated character (often determined by the $IFS). In some contexts, a field may be called a *record*.|

To preserve *whitespace* within a string or in a variable, use [[quoting#^QUOTINGREF|quoting]].

UNIX [[special-characters#^FILTERDEF|filters]] can target and operate on *whitespace* using the [[brief-introduction-to-regular-expressions#^POSIXREF|POSIX]] character class [[brief-introduction-to-regular-expressions#^WSPOSIX|[:space:]]].

[^1]: An *operator* is an agent that carries out an *operation*. Some examples are the common [[operations-and-related-topics#^AROPS1|arithmetic operators]], **+ - * /**. In Bash, there is some overlap between the concepts of *operator* and [[internal-commands-and-builtins#^keywordref|keyword]].

[^2]: This is more commonly known as the *ternary* operator. Unfortunately, *ternary* is an ugly word. It doesn't roll off the tongue, and it doesn't elucidate. It obfuscates. *Trinary* is by far the more elegant usage.

[^3]: **A**merican **S**tandard **C**ode for **I**nformation **I**nterchange. This is a system for encoding text characters (alphabetic, numeric, and a limited set of symbols) as 7-bit numbers that can be stored and manipulated by computers. Many of the ASCII characters are represented on a standard keyboard.

[^4]: A *PID*, or *process ID*, is a number assigned to a running process. The *PID*s of running processes may be viewed with a [[system-and-administrative-commands#^PPSSREF|ps]] command.

    **Definition:** A *process* is a currently executing command (or program), sometimes referred to as a *job*.

[^5]: The shell does the *brace expansion*. The command itself acts upon the *result* of the expansion.

[^6]: Exception: a code block in braces as part of a pipe *may* run as a [[subshells#^SUBSHELLSREF|subshell]].

    ```bash
    ls | { read firstline; read secondline; }
    #  Error. The code block in braces runs as a subshell,
    #+ so the output of "ls" cannot be passed to variables within the block.
    echo "First line is $firstline; second line is $secondline"  # Won't work.

    # Thanks, S.C.
    ```

[^7]: Even as in olden times a *philtre* denoted a potion alleged to have magical transformative powers, so does a UNIX *filter* transform its target in (roughly) analogous fashion. (The coder who comes up with a "love philtre" that runs on a Linux machine will likely win accolades and honors.)

[^8]: Bash stores a list of commands previously issued from the command-line in a *buffer*, or memory space, for recall with the [[internal-commands-and-builtins|builtin]] *history* commands.

[^9]: A linefeed (*newline*) is also a whitespace character. This explains why a *blank line*, consisting only of a linefeed, is considered whitespace.
