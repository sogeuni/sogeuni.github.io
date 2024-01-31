---
title: 10. Manipulating Variables
---


## Manipulating Strings

Bash supports a surprising number of string manipulation operations. Unfortunately, these tools lack a unified focus. Some are a subset of [[#Parameter Substitution|parameter substitution]], and others fall under the functionality of the UNIX [[external-filters-programs-and-commands#^EXPRREF|expr]] command. This results in inconsistent command syntax and overlap of functionality, not to mention confusion.

**String Length**

`${#string}`

expr length $string

These are the equivalent of _strlen()_ in _C_.

`expr "$string" : '.*'`

```bash
stringZ=abcABC123ABCabc

echo ${#stringZ}                 # 15
echo `expr length $stringZ`      # 15
echo `expr "$stringZ" : '.*'`    # 15
```

![[Example 10-1|Example 10-1]]

**Length of Matching Substring at Beginning of String**

`expr match "$string" '$substring'`

_$substring_ is a [[regexp#^REGEXREF|regular expression]].

`expr "$string" : '$substring'`

_$substring_ is a regular expression.

```bash
stringZ=abcABC123ABCabc
#       |------|
#       12345678

echo `expr match "$stringZ" 'abc[A-Z]*.2'`   # 8
echo `expr "$stringZ" : 'abc[A-Z]*.2'`       # 8
```

**Index**

`expr index $string $substring`

Numerical position in $string of first character in $substring that matches.

```bash
stringZ=abcABC123ABCabc
#       123456 ...
echo `expr index "$stringZ" C12`             # 6
                                             # C position.

echo `expr index "$stringZ" 1c`              # 3
# 'c' (in #3 position) matches before '1'.
```

This is the near equivalent of _strchr()_ in _C_.

**Substring Extraction**

`${string:position}`

Extracts substring from _`$string`_ at _`$position`_.

If the $string parameter is "*" or "@", then this extracts the [[another-look-at-variables#^POSPARAMREF|positional parameters]], [^1] starting at $position.

`${string:position:length}`

Extracts _`$length`_ characters of substring from _`$string`_ at _`$position`_.

```bash
stringZ=abcABC123ABCabc
#       0123456789.....
#       0-based indexing.

echo ${stringZ:0}                            # abcABC123ABCabc
echo ${stringZ:1}                            # bcABC123ABCabc
echo ${stringZ:7}                            # 23ABCabc

echo ${stringZ:7:3}                          # 23A
                                             # Three characters of substring.



# Is it possible to index from the right end of the string?
    
echo ${stringZ:-4}                           # abcABC123ABCabc
# Defaults to full string, as in ${parameter:-default}.
# However . . .

echo ${stringZ:(-4)}                         # Cabc 
echo ${stringZ: -4}                          # Cabc
# Now, it works.
# Parentheses or added space "escape" the position parameter.

# Thank you, Dan Jacobson, for pointing this out.
```

The _position_ and _length_ arguments can be "parameterized," that is, represented as a variable, rather than as a numerical constant.

![[Example 10-2|Example 10-2]]

If the $string parameter is "*" or "@", then this extracts a maximum of $length positional parameters, starting at $position.

```bash
echo ${*:2}          # Echoes second and following positional parameters.
echo ${@:2}          # Same as above.

echo ${*:2:3}        # Echoes three positional parameters, starting at second.
```

expr substr $string $position $length

Extracts _`$length`_ characters from _`$string`_ starting at _`$position`_.

```bash
stringZ=abcABC123ABCabc
#       123456789......
#       1-based indexing.

echo `expr substr $stringZ 1 2`              # ab
echo `expr substr $stringZ 4 3`              # ABC
```

expr match "$string" '\($substring\)'

Extracts _$substring_ at beginning of _$string_, where _$substring_ is a [[regexp#^REGEXREF|regular expression]].

expr "$string" : '\($substring\)'

Extracts _$substring_ at beginning of _$string_, where _$substring_ is a regular expression.

```bash
stringZ=abcABC123ABCabc
#       =======	    

echo `expr match "$stringZ" '\(.[b-c]*[A-Z]..[0-9]\)'`   # abcABC1
echo `expr "$stringZ" : '\(.[b-c]*[A-Z]..[0-9]\)'`       # abcABC1
echo `expr "$stringZ" : '\(.......\)'`                   # abcABC1
# All of the above forms give an identical result.
```

expr match "$string" '.*\($substring\)'

Extracts _$substring_ at _end_ of _$string_, where _$substring_ is a regular expression.

expr "$string" : '.*\($substring\)'

Extracts _$substring_ at _end_ of _$string_, where _$substring_ is a regular expression.

```bash
stringZ=abcABC123ABCabc
#                ======

echo `expr match "$stringZ" '.*\([A-C][A-C][A-C][a-c]*\)'`    # ABCabc
echo `expr "$stringZ" : '.*\(......\)'`                       # ABCabc
```

**Substring Removal**

${string#substring}

Deletes shortest match of _$substring_ from _front_ of _$string_.

${string##substring}

Deletes longest match of _$substring_ from _front_ of _$string_.

```bash
stringZ=abcABC123ABCabc
#       |----|          shortest
#       |----------|    longest

echo ${stringZ#a*C}      # 123ABCabc
# Strip out shortest match between 'a' and 'C'.

echo ${stringZ##a*C}     # abc
# Strip out longest match between 'a' and 'C'.



# You can parameterize the substrings.

X='a*C'

echo ${stringZ#$X}      # 123ABCabc
echo ${stringZ##$X}     # abc
                        # As above.
```

${string%substring}

Deletes shortest match of _`$substring`_ from _back_ of _`$string`_.

For example:

```bash
# Rename all filenames in $PWD with "TXT" suffix to a "txt" suffix.
# For example, "file1.TXT" becomes "file1.txt" . . .

SUFF=TXT
suff=txt

for i in $(ls *.$SUFF)
do
  mv -f $i ${i%.$SUFF}.$suff
  #  Leave unchanged everything *except* the shortest pattern match
  #+ starting from the right-hand-side of the variable $i . . .
done ### This could be condensed into a "one-liner" if desired.

# Thank you, Rory Winston.
```

`${string%%substring}`

Deletes longest match of _$substring_ from _back_ of _$string_.

```bash
stringZ=abcABC123ABCabc
#                    ||     shortest
#        |------------|     longest

echo ${stringZ%b*c}      # abcABC123ABCa
# Strip out shortest match between 'b' and 'c', from back of $stringZ.

echo ${stringZ%%b*c}     # a
# Strip out longest match between 'b' and 'c', from back of $stringZ.
```

This operator is useful for generating filenames.

![[Example 10-3|Example 10-3]]

![[Example 10-4|Example 10-4]]

A simple emulation of [[miscellaneous-commands#^GETOPTY|getopt]] using substring-extraction constructs.

![[Example 10-5|Example 10-5]]

**Substring Replacement**

${string/substring/replacement}

Replace first _match_ of _$substring_ with _$replacement_. [^2]

${string//substring/replacement}

Replace all matches of _$substring_ with _$replacement_.

```bash
stringZ=abcABC123ABCabc

echo ${stringZ/abc/xyz}       # xyzABC123ABCabc
                              # Replaces first match of 'abc' with 'xyz'.

echo ${stringZ//abc/xyz}      # xyzABC123ABCxyz
                              # Replaces all matches of 'abc' with # 'xyz'.

echo  ---------------
echo "$stringZ"               # abcABC123ABCabc
echo  ---------------
                              # The string itself is not altered!

# Can the match and replacement strings be parameterized?
match=abc
repl=000
echo ${stringZ/$match/$repl}  # 000ABC123ABCabc
#              ^      ^         ^^^
echo ${stringZ//$match/$repl} # 000ABC123ABC000
# Yes!          ^      ^        ^^^         ^^^

echo

# What happens if no $replacement string is supplied?
echo ${stringZ/abc}           # ABC123ABCabc
echo ${stringZ//abc}          # ABC123ABC
# A simple deletion takes place.
```

${string/#substring/replacement}

If _$substring_ matches _front_ end of _$string_, substitute _$replacement_ for _$substring_.

${string/%substring/replacement}

If _$substring_ matches _back_ end of _$string_, substitute _$replacement_ for _$substring_.

```bash
stringZ=abcABC123ABCabc

echo ${stringZ/#abc/XYZ}          # XYZABC123ABCabc
                                  # Replaces front-end match of 'abc' with 'XYZ'.

echo ${stringZ/%abc/XYZ}          # abcABC123ABCXYZ
                                  # Replaces back-end match of 'abc' with 'XYZ'.
```

### Manipulating strings using awk

A Bash script may invoke the string manipulation facilities of [[awk#^AWKREF|awk]] as an alternative to using its built-in operations.

![[Example 10-6|Example 10-6]]

### Further Reference

For more on string manipulation in scripts, refer to [[parameter-substitution|Section 10.2]] and the [[external-filters-programs-and-commands#^EXPEXTRSUB|relevant section]] of the [[external-filters-programs-and-commands#^EXPRREF|expr]] command listing.

Script examples:

1. [[Example 16-9|Example 16-9]]
2. [[Example 10-9|Example 10-9. Length of a variable]]
3. [[Example 10-10|Example 10-10. Pattern matching in parameter substitution]]
4. [[Example 10-11|Example 10-11. Renaming file extensions:]]
5. [[Example 10-13|Example 10-13. Matching patterns at prefix or suffix of string]]
6. [[Example A-36|Example A-36]]
7. [[Example A-41|Example A-41]]

## Parameter Substitution

**Manipulating and/or expanding variables**

**${parameter}**

Same as _`$parameter`_, i.e., value of the variable _parameter_. In certain contexts, only the less ambiguous _`${parameter}`_ form works.

May be used for concatenating variables with strings.

```bash
your_id=${USER}-on-${HOSTNAME}
echo "$your_id"
#
echo "Old \$PATH = $PATH"
PATH=${PATH}:/opt/bin  # Add /opt/bin to $PATH for duration of script.
echo "New \$PATH = $PATH"
```

**${parameter-default}**, **${parameter:-default}**

If parameter not set, use default.

```bash
var1=1
var2=2
# var3 is unset.

echo ${var1-$var2}   # 1
echo ${var3-$var2}   # 2
#           ^          Note the $ prefix.



echo ${username-`whoami`}
# Echoes the result of `whoami`, if variable $username is still unset.
```

> [!note]
> _${parameter-default}_ and _${parameter:-default}_ are almost equivalent. The extra : makes a difference only when _parameter_ has been declared, but is null.

```bash
#!/bin/bash
# param-sub.sh

#  Whether a variable has been declared
#+ affects triggering of the default option
#+ even if the variable is null.

username0=
echo "username0 has been declared, but is set to null."
echo "username0 = ${username0-`whoami`}"
# Will not echo.

echo

echo username1 has not been declared.
echo "username1 = ${username1-`whoami`}"
# Will echo.

username2=
echo "username2 has been declared, but is set to null."
echo "username2 = ${username2:-`whoami`}"
#                            ^
# Will echo because of :- rather than just - in condition test.
# Compare to first instance, above.


#

# Once again:

variable=
# variable has been declared, but is set to null.

echo "${variable-0}"    # (no output)
echo "${variable:-1}"   # 1
#               ^

unset variable

echo "${variable-2}"    # 2
echo "${variable:-3}"   # 3

exit 0
```

The _default parameter_ construct finds use in providing "missing" command-line arguments in scripts.

```bash
DEFAULT_FILENAME=generic.data
filename=${1:-$DEFAULT_FILENAME}
#  If not otherwise specified, the following command block operates
#+ on the file "generic.data".
#  Begin-Command-Block
#  ...
#  ...
#  ...
#  End-Command-Block



#  From "hanoi2.bash" example:
DISKS=${1:-E_NOPARAM}   # Must specify how many disks.
#  Set $DISKS to $1 command-line-parameter,
#+ or to $E_NOPARAM if that is unset.
```

See also [[Example 3-4|Example 3-4. Backup of all files changed in last day]], [[Example 31-2|Example 31-2]], and [[Example A-6|Example A-6]].

Compare this method with [[list-constructs#^ANDDEFAULT|using an _and list_ to supply a default command-line argument]].

**${parameter=default}**, **${parameter:=default}**

If parameter not set, set it to _default_.

Both forms nearly equivalent. The : makes a difference only when $parameter has been declared and is null, [^3] as above.

```bash
echo ${var=abc}   # abc
echo ${var=xyz}   # abc
# $var had already been set to abc, so it did not change.
```

**${parameter+alt_value}**, **${parameter:+alt_value}**

If parameter set, use **alt_value**, else use null string.

Both forms nearly equivalent. The : makes a difference only when _parameter_ has been declared and is null, see below.

```bash
echo "###### \${parameter+alt_value} ########"
echo

a=${param1+xyz}
echo "a = $a"      # a =

param2=
a=${param2+xyz}
echo "a = $a"      # a = xyz

param3=123
a=${param3+xyz}
echo "a = $a"      # a = xyz

echo
echo "###### \${parameter:+alt_value} ########"
echo

a=${param4:+xyz}
echo "a = $a"      # a =

param5=
a=${param5:+xyz}
echo "a = $a"      # a =
# Different result from   a=${param5+xyz}

param6=123
a=${param6:+xyz}
echo "a = $a"      # a = xyz
```

**${parameter?err_msg}**, **${parameter:?err_msg}**

If parameter set, use it, else print _err_msg_ and _abort the script_ with an [[exit-and-exit-status#^EXITSTATUSREF|exit status]] of 1.

Both forms nearly equivalent. The : makes a difference only when _parameter_ has been declared and is null, as above.

![[Example 10-7|Example 10-7]]

![[Example 10-8|Example 10-8]]

**Parameter substitution and/or expansion.** The following expressions are the complement to the **match** _in_ **expr** string operations (see [[Example 16-9|Example 16-9]]). These particular ones are used mostly in parsing file path names.

**Variable length / Substring removal**

**${#var}**

**String length** (number of characters in $var). For an [[arrays#^ARRAYREF|array]], **${#array}** is the length of the first element in the array.

> [!note] Exceptions:
>
> - **${#*}** and **${#@}** give the _number of positional parameters_.
> - For an array, **${#array[*]}** and **${#array[@]}** give the number of elements in the array.|

![[Example 10-9|Example 10-9]]

**${var#Pattern}**, **${var##Pattern}**

**${var#Pattern}** Remove from $var the _shortest_ part of $Pattern that matches the _front end_ of $var.

**${var##Pattern}** Remove from $var the _longest_ part of $Pattern that matches the _front end_ of $var.

A usage illustration from [[Example A-7|Example A-7]]:

```bash
# Function from "days-between.sh" example.
# Strips leading zero(s) from argument passed.

strip_leading_zero () #  Strip possible leading zero(s)
{                     #+ from argument passed.
  return=${1#0}       #  The "1" refers to "$1" -- passed arg.
}                     #  The "0" is what to remove from "$1" -- strips zeros.
```

Manfred Schwarb's more elaborate variation of the above:

```bash
strip_leading_zero2 () # Strip possible leading zero(s), since otherwise
{                      # Bash will interpret such numbers as octal values.
  shopt -s extglob     # Turn on extended globbing.
  local val=${1##+(0)} # Use local variable, longest matching series of 0's.
  shopt -u extglob     # Turn off extended globbing.
  _strip_leading_zero2=${val:-0}
                       # If input was 0, return 0 instead of "".
}
```

Another usage illustration:

```bash
echo `basename $PWD`        # Basename of current working directory.
echo "${PWD##*/}"           # Basename of current working directory.
echo
echo `basename $0`          # Name of script.
echo $0                     # Name of script.
echo "${0##*/}"             # Name of script.
echo
filename=test.data
echo "${filename##*.}"      # data
                            # Extension of filename.
```

**${var%Pattern}**, **${var%%Pattern}**

**${var%Pattern}** Remove from $var the _shortest_ part of $Pattern that matches the _back end_ of $var.

**${var%%Pattern}** Remove from $var the _longest_ part of $Pattern that matches the _back end_ of $var.

[[bashver2#^BASH2REF|Version 2]] of Bash added additional options.

![[Example 10-10|Example 10-10]]

![[Example 10-11|Example 10-11]]

**Variable expansion / Substring replacement**

These constructs have been adopted from _ksh_.

**${var:pos}**

Variable _var_ expanded, starting from offset _pos_.

**${var:pos:len}**

Expansion to a max of _len_ characters of variable _var_, from offset _pos_. See [[Example A-13|Example A-13]] for an example of the creative use of this operator.

**${var/Pattern/Replacement}**

First match of _Pattern_, within _var_ replaced with _Replacement_.

If _Replacement_ is omitted, then the first match of _Pattern_ is replaced by _nothing_, that is, deleted.

**${var//Pattern/Replacement}**

**Global replacement.** All matches of _Pattern_, within _var_ replaced with _Replacement_.

As above, if _Replacement_ is omitted, then all occurrences of _Pattern_ are replaced by _nothing_, that is, deleted.

![[Example 10-12|Example 10-12]]

**${var/#Pattern/Replacement}**

If _prefix_ of _var_ matches _Pattern_, then substitute _Replacement_ for _Pattern_.

**${var/%Pattern/Replacement}**

If _suffix_ of _var_ matches _Pattern_, then substitute _Replacement_ for _Pattern_.

![[Example 10-13|Example 10-13]]

**${!varprefix*}**, **${!varprefix@}**

Matches _names_ of all previously declared variables beginning with _varprefix_.

```bash
# This is a variation on indirect reference, but with a * or @.
# Bash, version 2.04, adds this feature.

xyz23=whatever
xyz24=

a=${!xyz*}         #  Expands to *names* of declared variables
# ^ ^   ^           + beginning with "xyz".
echo "a = $a"      #  a = xyz23 xyz24
a=${!xyz@}         #  Same as above.
echo "a = $a"      #  a = xyz23 xyz24

echo "---"

abc23=something_else
b=${!abc*}
echo "b = $b"      #  b = abc23
c=${!b}            #  Now, the more familiar type of indirect reference.
echo $c            #  something_else
```

[^1]: This applies to either command-line arguments or parameters passed to a [[functions|function]].

[^2]: Note that _$substring_ and _$replacement_ may refer to either _literal strings_ or _variables_, depending on context. See the first usage example.

[^3]: If $parameter is null in a non-interactive script, it will terminate with a [[exit-codes-with-special-meanings#^EXITCODESREF|127 exit status]] (the Bash error code for "command not found").
