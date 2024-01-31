---
title: Appendix B. Reference Cards
---


The following reference cards provide a useful _summary_ of certain scripting concepts. The foregoing text treats these matters in more depth, as well as giving usage examples.

**Table B-1. Special Shell Variables**

|Variable|Meaning|
|:--|:--|
|$0|Filename of script|
|$1|Positional parameter #1|
|$2 - $9|Positional parameters #2 - #9|
|${10}|Positional parameter #10|
|$#|Number of positional parameters|
|"$*"|All the positional parameters (as a single word) *|
|"$@"|All the positional parameters (as separate strings)|
|${#*}|Number of positional parameters|
|${#@}|Number of positional parameters|
|$?|Return value|
|$$|Process ID (PID) of script|
|$-|Flags passed to script (using _set_)|
|$_|Last argument of previous command|
|$!|Process ID (PID) of last job run in background|

***** _Must be quoted_, otherwise it defaults to $@.

**Table B-2. TEST Operators: Binary Comparison**

|Operator|Meaning|-----|Operator|Meaning|
|:--|:--|:--|:--|:--|
||||||
|[[other-comparison-operators#^ICOMPARISON1|Arithmetic Comparison]]|||[[other-comparison-operators#^SCOMPARISON1|String Comparison]]||
|-eq|Equal to||=|Equal to|
||||==|Equal to|
|-ne|Not equal to||!=|Not equal to|
|-lt|Less than||\<|Less than ([[special-characters#^ASCIIDEF|ASCII]]) *|
|-le|Less than or equal to||||
|-gt|Greater than||\>|Greater than (ASCII) *|
|-ge|Greater than or equal to||||
||||-z|String is empty|
||||-n|String is not empty|
||||||
|Arithmetic Comparison|[[tests#^DBLPRX|within double parentheses]] (( ... ))||||
|>|Greater than||||
|>=|Greater than or equal to||||
|<|Less than||||
|<=|Less than or equal to||||

***** _If within a double-bracket_ [[ ... | ... ]] _test construct, then no escape_ \ _is needed._

**Table B-3. TEST Operators: Files**

|Operator|Tests Whether|-----|Operator|Tests Whether|
|:--|:--|:--|:--|:--|
|-e|File exists||-s|File is not zero size|
|-f|File is a _regular_ file||||
|-d|File is a _directory_||-r|File has _read_ permission|
|-h|File is a [[basic#^SYMLINKREF|symbolic link]]||-w|File has _write_ permission|
|-L|File is a _symbolic link_||-x|File has _execute_ permission|
|-b|File is a [[dev#^BLOCKDEVREF|block device]]||||
|-c|File is a [[dev#^CHARDEVREF|character device]]||-g|_sgid_ flag set|
|-p|File is a [[special-characters#^PIPEREF|pipe]]||-u|_suid_ flag set|
|-S|File is a [[dev#^SOCKETREF|socket]]||-k|"sticky bit" set|
|-t|File is associated with a _terminal_||||
||||||
|-N|File modified since it was last read||F1 -nt F2|File F1 is _newer_ than F2 *|
|-O|You own the file||F1 -ot F2|File F1 is _older_ than F2 *|
|-G|_Group id_ of file same as yours||F1 -ef F2|Files F1 and F2 are _hard links_ to the same file *|
||||||
|!|NOT (inverts sense of above tests)||||

***** _Binary_ operator (requires two operands).

**Table B-4. Parameter Substitution and Expansion**

|Expression|Meaning|
|:--|:--|
|${var}|Value of _var_ (same as _$var_)|
|||
|${var-$DEFAULT}|If _var_ not set, [[internal-commands-and-builtins#^EVALREF|evaluate]] expression as _$DEFAULT_ *|
|${var:-$DEFAULT}|If _var_ not set or is empty, _evaluate_ expression as _$DEFAULT_ *|
|||
|${var=$DEFAULT}|If _var_ not set, evaluate expression as _$DEFAULT_ *|
|${var:=$DEFAULT}|If _var_ not set or is empty, evaluate expression as _$DEFAULT_ *|
|||
|${var+$OTHER}|If _var_ set, evaluate expression as _$OTHER_, otherwise as null string|
|${var:+$OTHER}|If _var_ set, evaluate expression as _$OTHER_, otherwise as null string|
|||
|${var?$ERR_MSG}|If _var_ not set, print _$ERR_MSG_ and abort script with an exit status of 1.*|
|${var:?$ERR_MSG}|If _var_ not set, print _$ERR_MSG_ and abort script with an exit status of 1.*|
|||
|${!varprefix*}|Matches all previously declared variables beginning with _varprefix_|
|${!varprefix@}|Matches all previously declared variables beginning with _varprefix_|

***** If _var_ _is_ set, evaluate the expression as _$var_ with no side-effects.

**# Note** that some of the above behavior of operators has changed from earlier versions of Bash.

**Table B-5. String Operations**

|Expression|Meaning|
|:--|:--|
|${#string}|Length of _$string_|
|||
|${string:position}|Extract substring from _$string_ at _$position_|
|${string:position:length}|Extract _$length_ characters substring from _$string_ at _$position_ [zero-indexed, first character is at position 0]|
|||
|${string#substring}|Strip shortest match of _$substring_ from front of _$string_|
|${string##substring}|Strip longest match of _$substring_ from front of _$string_|
|${string%substring}|Strip shortest match of _$substring_ from back of _$string_|
|${string%%substring}|Strip longest match of _$substring_ from back of _$string_|
|||
|${string/substring/replacement}|Replace first match of _$substring_ with _$replacement_|
|${string//substring/replacement}|Replace _all_ matches of _$substring_ with _$replacement_|
|${string/#substring/replacement}|If _$substring_ matches _front_ end of _$string_, substitute _$replacement_ for _$substring_|
|${string/%substring/replacement}|If _$substring_ matches _back_ end of _$string_, substitute _$replacement_ for _$substring_|
|||
|||
|expr match "$string" '$substring'|Length of matching _$substring_* at beginning of _$string_|
|expr "$string" : '$substring'|Length of matching _$substring_* at beginning of _$string_|
|expr index "$string" $substring|Numerical position in _$string_ of first character in _$substring_* that matches [0 if no match, first character counts as position 1]|
|expr substr $string $position $length|Extract _$length_ characters from _$string_ starting at _$position_ [0 if no match, first character counts as position 1]|
|expr match "$string" '\($substring\)'|Extract _$substring_*, searching from beginning of _$string_|
|expr "$string" : '\($substring\)'|Extract _$substring_* , searching from beginning of _$string_|
|expr match "$string" '.*\($substring\)'|Extract _$substring_*, searching from end of _$string_|
|expr "$string" : '.*\($substring\)'|Extract _$substring_*, searching from end of _$string_|

***** Where _$substring_ is a [[regexp#^REGEXREF|Regular Expression]].

**Table B-6. Miscellaneous Constructs**

|Expression|Interpretation|
|:--|:--|
|||
|[[brief-introduction-to-regular-expressions#^BRACKETSREF|Brackets]]||
|if [[special-characters#^LEFTBRACKET| CONDITION ]|[Test construct]]|
|if [[tests#^DBLBRACKETS|[ CONDITION ]]|[Extended test construct]]|
|Array[[arrays#^ARRAYREF|1]=element1|[Array initialization]]|
|[[brief-introduction-to-regular-expressions#^BRACKETSREF|a-z]|[Range of characters]] within a [[regexp#^REGEXREF|Regular Expression]]|
|||
|Curly Brackets||
|${variable}|[[parameter-substitution#^PARAMSUBREF|Parameter substitution]]|
|${!variable}|[[indirect-references#^IVRREF|Indirect variable reference]]|
|{ command1; command2; . . . commandN; }|[[special-characters#^CODEBLOCKREF|Block of code]]|
|{string1,string2,string3,...}|[[special-characters#^BRACEEXPREF|Brace expansion]]|
|{a..z}|[[bashver3#^BRACEEXPREF3|Extended brace expansion]]|
|{}|Text replacement, after [[external-filters-programs-and-commands#^CURLYBRACKETSREF|find]] and [[external-filters-programs-and-commands#^XARGSCURLYREF|xargs]]|
|||
|||
|[[special-characters#^PARENSREF|Parentheses]]||
|( command1; command2 )|Command group executed within a [[subshells#^SUBSHELLSREF|subshell]]|
|Array=(element1 element2 element3)|[[arrays#^ARRAYINIT0|Array initialization]]|
|result=$(COMMAND)|[[command-substitution#^CSPARENS|Command substitutio[Process substitution](Chapter%2023.%20Process%20Substitution.md#^PROCESSSUBREF)SUBREF|Process substitution]]|
|<(COMMAND)|Process substitution|
|||
|[[operations-and-related-topics.html|Double Parentheses]]||
|(( var = 78 ))|[[operations-and-related-topics#^DBLPARENSREF|Integer arithmetic]]|
|var=$(( 20 + 5 ))|Integer arithmetic, with variable assignment|
|(( var++ ))|_C-style_ [[operations-and-related-topics#^PLUSPLUSREF|variable increment]]|
|(( var-- ))|_C-style_ [[operations-and-related-topics#^PLUSPLUSREF|variable decrement]]|
|(( var0 = var1<98?9:21 ))|_C-style_ [[special-characters#^CSTRINARY|ternary]] operation|
|||
|[[quoting#^QUOTINGREF|Quoting]]||
|"$variable"|[[varsubn#^DBLQUO|"Weak" quoting]]|
|'string'|[[varsubn#^SNGLQUO|'Strong' quoting]]|
|||
|[[command-substitution#^BACKQUOTESREF|Back Quotes]]||
|result=`COMMAND`|[[command-substitution#^COMMANDSUBREF|Command substitution]], classic style|
