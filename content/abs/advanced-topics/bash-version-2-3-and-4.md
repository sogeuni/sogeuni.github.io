**Table of Contents**

37.2. [[bash-version-2-3-and-4|Bash, version 3]]

37.2.1. [[bash-version-2-3-and-4#^AEN20956|Bash, version 3.1]]

37.2.2. [[bash-version-2-3-and-4#^AEN20987|Bash, version 3.2]]

37.3. [[bash-version-2-3-and-4|Bash, version 4]]

37.3.1. [[bash-version-2-3-and-4#^AEN21183|Bash, version 4.1]]

37.3.2. [[bash-version-2-3-and-4#^AEN21220|Bash, version 4.2]]

## Bash, version 2

The current version of _Bash_, the one you have running on your machine, is most likely version 2.xx.yy, 3.xx.yy, or 4.xx.yy.

```bash
bash$ echo $BASH_VERSION
3.2.25(1)-release
	      
```

The version 2 update of the classic Bash scripting language added array variables, string and parameter expansion, and a better method of indirect variable references, among other features.

![[Example 37-1|Example 37-1]]

![[Example 37-2|Example 37-2]]

![[Example 37-3|Example 37-3]]

![[Example 37-4|Example 37-4]]

## Bash, version 3

On July 27, 2004, Chet Ramey released version 3 of Bash. This update fixed quite a number of bugs and added new features.

Some of the more important added features:

- A new, more generalized **{a..z}** [[special-characters#^BRACEEXPREF|brace expansion]] operator.
    
```bash
#!/bin/bash

for i in {1..10}
#  Simpler and more straightforward than
#+ for i in $(seq 10)
do
  echo -n "$i "
done

echo

# 1 2 3 4 5 6 7 8 9 10



# Or just . . .

echo {a..z}    #  a b c d e f g h i j k l m n o p q r s t u v w x y z
echo {e..m}    #  e f g h i j k l m
echo {z..a}    #  z y x w v u t s r q p o n m l k j i h g f e d c b a
               #  Works backwards, too.
echo {25..30}  #  25 26 27 28 29 30
echo {3..-2}   #  3 2 1 0 -1 -2
echo {X..d}    #  X Y Z [  ] ^ _ ` a b c d
               #  Shows (some of) the ASCII characters between Z and a,
               #+ but don't rely on this type of behavior because . . .
echo {]..a}    #  {]..a}
               #  Why?


# You can tack on prefixes and suffixes.
echo "Number #"{1..4}, "..."
     # Number #1, Number #2, Number #3, Number #4, ...


# You can concatenate brace-expansion sets.
echo {1..3}{x..z}" +" "..."
     # 1x + 1y + 1z + 2x + 2y + 2z + 3x + 3y + 3z + ...
     # Generates an algebraic expression.
     # This could be used to find permutations.

# You can nest brace-expansion sets.
echo {{a..c},{1..3}}
     # a b c 1 2 3
     # The "comma operator" splices together strings.

# ########## ######### ############ ########### ######### ###############
# Unfortunately, brace expansion does not lend itself to parameterization.
var1=1
var2=5
echo {$var1..$var2}   # {1..5}


# Yet, as Emiliano G. points out, using "eval" overcomes this limitation.

start=0
end=10
for index in $(eval echo {$start..$end})
do
  echo -n "$index "   # 0 1 2 3 4 5 6 7 8 9 10 
done

echo
```

- The **`${!array[@]}`** operator, which expands to all the indices of a given [[arrays#^ARRAYREF|array]].

```bash
#!/bin/bash

Array=(element-zero element-one element-two element-three)

echo ${Array[0]}   # element-zero
                   # First element of array.

echo ${!Array[@]}  # 0 1 2 3
                   # All the indices of Array.

for i in ${!Array[@]}
do
  echo ${Array[i]} # element-zero
                   # element-one
                   # element-two
                   # element-three
                   #
                   # All the elements in Array.
done
```

- The **=~** [[regexp#^REGEXREF|Regular Expression]] matching operator within a [[tests#^DBLBRACKETS|double brackets]] test expression. (Perl has a similar operator.)

```bash
#!/bin/bash

variable="This is a fine mess."

echo "$variable"

# Regex matching with =~ operator within [[ double brackets ]].
if [[ "$variable" =~ T.........fin*es* ]]
# NOTE: As of version 3.2 of Bash, expression to match no longer quoted.
then
  echo "match found"
      # match found
fi
```

    Or, more usefully:

```bash
#!/bin/bash

input=$1


if [[ "$input" =~ "[0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9][0-9][0-9]" ]]
#                 ^ NOTE: Quoting not necessary, as of version 3.2 of Bash.
# NNN-NN-NNNN (where each N is a digit).
then
  echo "Social Security number."
  # Process SSN.
else
  echo "Not a Social Security number!"
  # Or, ask for corrected input.
fi
```

For additional examples of using the **=~** operator, see [[Example A-29|Example A-29]], [[Example 19-14|Example 19-14]], [[Example A-35|Example A-35]], and [[Example A-24|Example A-24]].

- The new `set -o pipefail` option is useful for debugging [[special-characters#^PIPEREF|pipes]]. If this option is set, then the [[exit-and-exit-status#^EXITSTATUSREF|exit status]] of a pipe is the exit status of the last command in the pipe to _fail_ (return a non-zero value), rather than the actual final command in the pipe.

    See [[Example 16-43|Example 16-43]].

> [!caution]
> The update to version 3 of Bash breaks a few scripts that worked under earlier versions. _Test critical legacy scripts to make sure they still work!_

As it happens, a couple of the scripts in the _Advanced Bash Scripting Guide_ had to be fixed up (see [[Example 9-4|Example 9-4]], for instance).|

### Bash, version 3.1

The version 3.1 update of Bash introduces a number of bugfixes and a few minor changes.

- The += operator is now permitted in in places where previously only the = assignment operator was recognized.

```bash
a=1
echo $a        # 1

a+=5           # Won't work under versions of Bash earlier than 3.1.
echo $a        # 15

a+=Hello
echo $a        # 15Hello
```

Here, += functions as a _string concatenation_ operator. Note that its behavior in this particular context is different than within a [[internal-commands-and-builtins#^LETREF|let]] construct.

```bash
a=1
echo $a        # 1

let a+=5       # Integer arithmetic, rather than string concatenation.
echo $a        # 6

let a+=Hello   # Doesn't "add" anything to a.
echo $a        # 6
```

Jeffrey Haemer points out that this concatenation operator can be quite useful. In this instance, we append a directory to the $PATH.

```bash
bash$ echo $PATH
/usr/bin:/bin:/usr/local/bin:/usr/X11R6/bin/:/usr/games


bash$ PATH+=:/opt/bin

bash$ echo $PATH
/usr/bin:/bin:/usr/local/bin:/usr/X11R6/bin/:/usr/games:/opt/bin
      
```

### Bash, version 3.2

This is pretty much a bugfix update.

- In [[parameter-substitution#^PSGLOB|_global_ parameter substitutions]], the pattern no longer anchors at the start of the string.

- The --wordexp option disables [[process-substitution#^PROCESSSUBREF|process substitution]].

- The **=~** [[bash-version-2-3-and-4#^REGEXMATCHREF|Regular Expression match operator]] no longer requires [[Chapter 5. Quoting#^QUOTINGREF|quoting]] of the _pattern_ within [[tests#^DBLBRACKETS|[[ ... | ... ]]]].

> [!caution]
> In fact, quoting in this context is _not_ advisable as it may cause _regex_ evaluation to fail. Chet Ramey states in the [[biblio#^BASHFAQ|Bash FAQ]] that quoting explicitly disables regex evaluation. See also the [Ubuntu Bug List](https://bugs.launchpad.net/ubuntu-website/+bug/109931) and [Wikinerds on Bash syntax](http://en.wikinerds.org/index.php/Bash_syntax_and_semantics).
>
> Setting _shopt -s compat31_ in a script causes reversion to the original behavior.

## Bash, version 4

Chet Ramey announced Version 4 of Bash on the 20th of February, 2009. This release has a number of significant new features, as well as some important bugfixes.

Among the new goodies:

- Associative arrays. [^1]
    
> An _associative_ array can be thought of as a set of two linked arrays -- one holding the _data_, and the other the _keys_ that index the individual elements of the _data_ array.
    
![[Example 37-5|Example 37-5]]
    
![[Example 37-6|Example 37-6]]
    
    See [[Example A-53|Example A-53]] for an interesting usage of an _associative array_.
    
> [!caution]
> Elements of the _index_ array may include embedded [[special-characters#Whitespace|space characters]], or even leading and/or trailing space characters. However, index array elements containing _only_ _whitespace_ are _not_ permitted.
>
> ```bash
> address[   ]="Blank"   # Error!
> ```
    
- Enhancements to the [[loops-and-branches#^CASEESAC1|case]] construct: the _;;&_ and _;&_ terminators.
    
![[Example 37-7|Example 37-7]]
    
- The new **coproc** builtin enables two parallel [[special-characters#^PROCESSREF|processes]] to communicate and interact. As Chet Ramey states in the [[bibliography#^BASHFAQ|Bash FAQ]] [^2] , ver. 4.01:
    
    >     There is a new 'coproc' reserved word that specifies a coprocess:  
    >     an asynchronous command run with two pipes connected to the creating  
    >     shell. Coprocs can be named. The input and output file descriptors  
    >     and the PID of the coprocess are available to the calling shell in  
    >     variables with coproc-specific names.  
    >   
    >     George Dimitriu explains,  
    >     "... coproc ... is a feature used in Bash process substitution,  
    >     which now is made publicly available."  
    >     This means it can be explicitly invoked in a script, rather than  
    >     just being a behind-the-scenes mechanism used by Bash.  
    >       
    
    Coprocesses use _file descriptors_. [[io-redirection#^FDREF2|File descriptors enable processes and pipes to communicate]].
    
```bash
#!/bin/bash4
# A coprocess communicates with a while-read loop.


coproc { cat mx_data.txt; sleep 2; }
#                         ^^^^^^^
# Try running this without "sleep 2" and see what happens.

while read -u ${COPROC[0]} line    #  ${COPROC[0]} is the
do                                 #+ file descriptor of the coprocess.
  echo "$line" | sed -e 's/line/NOT-ORIGINAL-TEXT/'
done

kill $COPROC_PID                   #  No longer need the coprocess,
                                   #+ so kill its PID.
```
    
    But, be careful!
    
```bash
#!/bin/bash4

echo; echo
a=aaa
b=bbb
c=ccc

coproc echo "one two three"
while read -u ${COPROC[0]} a b c;  #  Note that this loop
do                                 #+ runs in a subshell.
  echo "Inside while-read loop: ";
  echo "a = $a"; echo "b = $b"; echo "c = $c"
  echo "coproc file descriptor: ${COPROC[0]}"
done 

# a = one
# b = two
# c = three
# So far, so good, but ...

echo "-----------------"
echo "Outside while-read loop: "
echo "a = $a"  # a =
echo "b = $b"  # b =
echo "c = $c"  # c =
echo "coproc file descriptor: ${COPROC[0]}"
echo
#  The coproc is still running, but ...
#+ it still doesn't enable the parent process
#+ to "inherit" variables from the child process, the while-read loop.

#  Compare this to the "badread.sh" script.
```
    
> [!caution]
> The coprocess is _asynchronous_, and this might cause a problem. It may terminate before another process has finished communicating with it.
>
> ```bash
> #!/bin/bash4
> 
> coproc cpname { for i in {0..10}; do echo "index = $i"; done; }
> #      ^^^^^^ This is a *named* coprocess.
> read -u ${cpname[0]}
> echo $REPLY         #  index = 0
> echo ${COPROC[0]}   #+ No output ... the coprocess timed out
> #  after the first loop iteration.
> 
> 
> 
> # However, George Dimitriu has a partial fix.
> 
> coproc cpname { for i in {0..10}; do echo "index = $i"; done; sleep 1;
> echo hi > myo; cat - >> myo; }
> #       ^^^^^ This is a *named* coprocess.
> 
> echo "I am main"$'\04' >&${cpname[1]}
> myfd=${cpname[0]}
> echo myfd=$myfd
> 
> ### while read -u $myfd
> ### do
> ###   echo $REPLY;
> ### done
> 
> echo $cpname_PID
> 
> #  Run this with and without the commented-out while-loop, and it is
> #+ apparent that each process, the executing shell and the coprocess,
> #+ waits for the other to finish writing in its own write-enabled pipe.
> ```
    
- The new **mapfile** builtin makes it possible to load an array with the contents of a text file without using a loop or [[arrays#^ARRAYINITCS|command substitution]].

```bash
#!/bin/bash4

mapfile Arr1 < $0
# Same result as     Arr1=( $(cat $0) )
echo "${Arr1[@]}"  # Copies this entire script out to stdout.

echo "--"; echo

# But, not the same as   read -a   !!!
read -a Arr2 < $0
echo "${Arr2[@]}"  # Reads only first line of script into the array.

exit
```
    
- The [[internal-commands-and-builtins#^READREF|read]] builtin got a minor facelift. The -t [[internal-commands-and-builtins#^READTIMED|timeout]] option now accepts (decimal) fractional values [^3] and the -i option permits preloading the edit buffer. [^4] Unfortunately, these enhancements are still a work in progress and not (yet) usable in scripts.
    
- [[parameter-substitution#^PARAMSUBREF|Parameter substitution]] gets _case-modification_ operators.
    
```bash
#!/bin/bash4

var=veryMixedUpVariable
echo ${var}            # veryMixedUpVariable
echo ${var^}           # VeryMixedUpVariable
#         *              First char --> uppercase.
echo ${var^^}          # VERYMIXEDUPVARIABLE
#         **             All chars  --> uppercase.
echo ${var,}           # veryMixedUpVariable
#         *              First char --> lowercase.
echo ${var,,}          # verymixedupvariable
#         **             All chars  --> lowercase.
```
    
- The [[typing-variables.html|declare]] builtin now accepts the -l _lowercase_ and -c _capitalize_ options.
    
```bash
#!/bin/bash4

declare -l var1            # Will change to lowercase
var1=MixedCaseVARIABLE
echo "$var1"               # mixedcasevariable
# Same effect as             echo $var1 | tr A-Z a-z

declare -c var2            # Changes only initial char to uppercase.
var2=originally_lowercase
echo "$var2"               # Originally_lowercase
# NOT the same effect as     echo $var2 | tr a-z A-Z
```
    
- [[special-characters#^BRACEEXPREF|Brace expansion]] has more options.
    
    _Increment/decrement_, specified in the final term within braces.
    
```bash
#!/bin/bash4

echo {40..60..2}
# 40 42 44 46 48 50 52 54 56 58 60
# All the even numbers, between 40 and 60.

echo {60..40..2}
# 60 58 56 54 52 50 48 46 44 42 40
# All the even numbers, between 40 and 60, counting backwards.
# In effect, a decrement.
echo {60..40..-2}
# The same output. The minus sign is not necessary.

# But, what about letters and symbols?
echo {X..d}
# X Y Z [  ] ^ _ ` a b c d
# Does not echo the \ which escapes a space.
```
    
    _Zero-padding_, specified in the first term within braces, prefixes each term in the output with the _same number_ of zeroes.
    
```bash
bash4$ echo {010..15}
010 011 012 013 014 015


bash4$ echo {000..10}
000 001 002 003 004 005 006 007 008 009 010
      
```
    
- [[bash-version-2-3-and-4#^SUBSTREXTREF4|_Substring extraction_ on _positional parameters_]] now starts with [[othertypesv#^SCRNAMEPARAM|$0]] as the _zero-index_. (This corrects an inconsistency in the treatment of positional parameters.)
    
```bash
#!/bin/bash
# show-params.bash
# Requires version 4+ of Bash.

# Invoke this scripts with at least one positional parameter.

E_BADPARAMS=99

if [ -z "$1" ]
then
  echo "Usage $0 param1 ..."
  exit $E_BADPARAMS
fi

echo ${@:0}

# bash3 show-params.bash4 one two three
# one two three

# bash4 show-params.bash4 one two three
# show-params.bash4 one two three

# $0                $1  $2  $3
```
    
- The new ** [[globbing.html|globbing]] operator matches filenames and directories [[local-variables#^RECURSIONREF0|recursively]].
    
```bash
#!/bin/bash4
# filelist.bash4

shopt -s globstar  # Must enable globstar, otherwise ** doesn't work.
                   # The globstar shell option is new to version 4 of Bash.

echo "Using *"; echo
for filename in *
do
  echo "$filename"
done   # Lists only files in current directory ($PWD).

echo; echo "--------------"; echo

echo "Using **"
for filename in **
do
  echo "$filename"
done   # Lists complete file tree, recursively.

exit

Using *

allmyfiles
filelist.bash4

--------------

Using **

allmyfiles
allmyfiles/file.index.txt
allmyfiles/my_music
allmyfiles/my_music/me-singing-60s-folksongs.ogg
allmyfiles/my_music/me-singing-opera.ogg
allmyfiles/my_music/piano-lesson.1.ogg
allmyfiles/my_pictures
allmyfiles/my_pictures/at-beach-with-Jade.png
allmyfiles/my_pictures/picnic-with-Melissa.png
filelist.bash4
```
    
- The new [[another-look-at-variables#^BASHPIDREF|$BASHPID]] internal variable.
    
- There is a new [[internal-commands-and-builtins|builtin]] error-handling function named **command_not_found_handle**.
    
```bash
#!/bin/bash4

command_not_found_handle ()
{ # Accepts implicit parameters.
  echo "The following command is not valid: \""$1\"""
  echo "With the following argument(s): \""$2\"" \""$3\"""   # $4, $5 ...
} # $1, $2, etc. are not explicitly passed to the function.

bad_command arg1 arg2

# The following command is not valid: "bad_command"
# With the following argument(s): "arg1" "arg2"
```

> _Editorial comment_
>
> Associative arrays? Coprocesses? Whatever happened to the lean and mean Bash we have come to know and love? Could it be suffering from (horrors!) "feature creep"? Or perhaps even Korn shell envy?
>
> _Note to Chet Ramey:_ Please add only _essential_ features in future Bash releases -- perhaps _for-each_ loops and support for multi-dimensional arrays. [^5] Most Bash users won't need, won't use, and likely won't greatly appreciate complex "features" like built-in debuggers, Perl interfaces, and bolt-on rocket boosters.

### Bash, version 4.1

Version 4.1 of Bash, released in May, 2010, was primarily a bugfix update.

- The [[internal-commands-and-builtins#^PRINTFREF|printf]] command now accepts a -v option for setting [[arrays#^ARRAYREF|array]] indices.
    
- Within [[tests#^DBLBRACKETS|double brackets]], the **>** and **<** string comparison operators now conform to the [[localization#^LOCALEREF|locale]]. Since the locale setting may affect the sorting order of string expressions, this has side-effects on comparison tests within _[[ ... | ... ]]_ expressions.
    
- The [[internal-commands-and-builtins#^READREF|read]] builtin now takes a -N option (_read -N chars_), which causes the _read_ to terminate after _chars_ characters.
    
![[Example 37-8|Example 37-8]]
    
- [[here-documents#^HEREDOCREF|Here documents]] embedded in [[varassignment#^COMMANDSUBREF0|**$( ... )** command substitution]] constructs may terminate with a simple **)**.
    
![[Example 37-9|Example 37-9]]

### Bash, version 4.2

Version 4.2 of Bash, released in February, 2011, contains a number of new features and enhancements, in addition to bugfixes.

- Bash now supports the the _\u_ and _\U_ _Unicode_ escape.

> Unicode is a cross-platform standard for encoding into numerical values letters and graphic symbols. This permits representing and displaying characters in foreign alphabets and unusual fonts.

```bash
echo -e '\u2630'   # Horizontal triple bar character.
# Equivalent to the more roundabout:
echo -e "\xE2\x98\xB0"
                   # Recognized by earlier Bash versions.

echo -e '\u220F'   # PI (Greek letter and mathematical symbol)
echo -e '\u0416'   # Capital "ZHE" (Cyrillic letter)
echo -e '\u2708'   # Airplane (Dingbat font) symbol
echo -e '\u2622'   # Radioactivity trefoil

echo -e "The amplifier circuit requires a 100 \u2126 pull-up resistor."


unicode_var='\u2640'
echo -e $unicode_var      # Female symbol
printf "$unicode_var \n"  # Female symbol, with newline


#  And for something a bit more elaborate . . .

#  We can store Unicode symbols in an associative array,
#+ then retrieve them by name.
#  Run this in a gnome-terminal or a terminal with a large, bold font
#+ for better legibility.

declare -A symbol  # Associative array.

symbol[script_E]='\u2130'
symbol[script_F]='\u2131'
symbol[script_J]='\u2110'
symbol[script_M]='\u2133'
symbol[Rx]='\u211E'
symbol[TEL]='\u2121'
symbol[FAX]='\u213B'
symbol[care_of]='\u2105'
symbol[account]='\u2100'
symbol[trademark]='\u2122'


echo -ne "${symbol[script_E]}   "
echo -ne "${symbol[script_F]}   "
echo -ne "${symbol[script_J]}   "
echo -ne "${symbol[script_M]}   "
echo -ne "${symbol[Rx]}   "
echo -ne "${symbol[TEL]}   "
echo -ne "${symbol[FAX]}   "
echo -ne "${symbol[care_of]}   "
echo -ne "${symbol[account]}   "
echo -ne "${symbol[trademark]}   "
echo
```

> [!note]
> The above example uses the [[quoting#^STRQ|** ... '**]] _string-expansion_ construct.
    
- When the _lastpipe_ shell option is set, the last command in a [[special-characters#^PIPEREF|pipe]] _doesn't run in a subshell_.
    
![[Example 37-10|Example 37-10]]
    
    This option offers possible "fixups" for these example scripts: [[Example 34-3|Example 34-3]] and [[Example 15-8|Example 15-8]].
    
- Negative [[arrays#^ARRAYREF|array]] indices permit counting backwards from the end of an array.
    
![[Example 37-11|Example 37-11]]
    
- [[manipulating-variables#^SUBSTREXTR01|Substring extraction]] uses a negative _length_ parameter to specify an offset from the _end_ of the target string.
    
![[Example 37-12|Example 37-12]]

[^1]: To be more specific, Bash 4+ has _limited_ support for associative arrays. It's a bare-bones implementation, and it lacks the much of the functionality of such arrays in other programming languages. Note, however, that [[optimizations#^ASSOCARRTST|associative arrays in Bash seem to execute faster and more efficiently than numerically-indexed arrays]].

[^2]: Copyright 1995-2009 by Chester Ramey.

[^3]: This only works with [[special-characters#^PIPEREF|pipes]] and certain other _special_ files.

[^4]: But only in conjunction with [[internal-commands-and-builtins#^READLINEREF|readline]], i.e., from the command-line.

[^5]: And while you're at it, consider fixing the notorious [[internal-commands-and-builtins#^PIPEREADREF0|piped read]] problem.
