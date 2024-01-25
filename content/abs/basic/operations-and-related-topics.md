---
title: 8. Operations and Related Topics
---


## Operators

**assignment**

_variable assignment_

Initializing or changing the value of a variable

=

All-purpose assignment operator, which works for both arithmetic and string assignments.

```bash
var=27
category=minerals  # No spaces allowed after the "=".
```

> [!caution]
> Do not confuse the "=" assignment operator with the [[tests#EQUALSIGNREF|= test operator]].
>
> ```bash
> #   =  as a test operator
> 
> if [ "$string1" = "$string2" ]
> then
>    command
> fi
> 
> #  if [ "X$string1" = "X$string2" ] is safer,
> #+ to prevent an error message should one of the variables be empty.
> #  (The prepended "X" characters cancel out.)
> ```

**arithmetic operators**

+

plus

-

minus

*

multiplication

/

division

**

exponentiation

```bash
# Bash, version 2.02, introduced the "**" exponentiation operator.

let "z=5**3"    # 5 * 5 * 5
echo "z = $z"   # z = 125
```

%

modulo, or mod (returns the _remainder_ of an integer division operation)

```bash
bash$ expr 5 % 3
2
	    
```

_5/3 = 1, with remainder 2_

This operator finds use in, among other things, generating numbers within a specific range (see [[another-look-at-variables#EX21|Example 9-11]] and [[another-look-at-variables#RANDOMTEST|Example 9-15]]) and formatting program output (see [[arrays#^QFUNCTION|Example 27-16]] and [[contributed-scripts#COLLATZ|Example A-6]]). It can even be used to generate prime numbers, (see [[contributed-scripts#PRIMES|Example A-15]]). Modulo turns up surprisingly often in numerical recipes.

###### Example 8-1. Greatest common divisor

```bash

#!/bin/bash
# gcd.sh: greatest common divisor
#         Uses Euclid's algorithm

#  The "greatest common divisor" (gcd) of two integers
#+ is the largest integer that will divide both, leaving no remainder.

#  Euclid's algorithm uses successive division.
#    In each pass,
#+      dividend <---  divisor
#+      divisor  <---  remainder
#+   until remainder = 0.
#    The gcd = dividend, on the final pass.
#
#  For an excellent discussion of Euclid's algorithm, see
#+ Jim Loy's site, http://www.jimloy.com/number/euclids.htm.


# ------------------------------------------------------
# Argument check
ARGS=2
E_BADARGS=85

if [ $# -ne "$ARGS" ]
then
  echo "Usage: `basename $0` first-number second-number"
  exit $E_BADARGS
fi
# ------------------------------------------------------


gcd ()
{

  dividend=$1             #  Arbitrary assignment.
  divisor=$2              #! It doesn't matter which of the two is larger.
                          #  Why not?

  remainder=1             #  If an uninitialized variable is used inside
                          #+ test brackets, an error message results.

  until [ "$remainder" -eq 0 ]
  do    #  ^^^^^^^^^^  Must be previously initialized!
    let "remainder = $dividend % $divisor"
    dividend=$divisor     # Now repeat with 2 smallest numbers.
    divisor=$remainder
  done                    # Euclid's algorithm

}                         # Last $dividend is the gcd.


gcd $1 $2

echo; echo "GCD of $1 and $2 = $dividend"; echo


# Exercises :
# ---------
# 1) Check command-line arguments to make sure they are integers,
#+   and exit the script with an appropriate error message if not.
# 2) Rewrite the gcd () function to use local variables.

exit 0
```

+=

_plus-equal_ (increment variable by a constant) [^1]

**let "var += 5"** results in _var_ being incremented by 5.

-=

_minus-equal_ (decrement variable by a constant)

*=

_times-equal_ (multiply variable by a constant)

**let "var *= 4"** results in _var_ being multiplied by 4.

/=

_slash-equal_ (divide variable by a constant)

%=

_mod-equal_ (_remainder_ of dividing variable by a constant)

_Arithmetic operators often occur in an [expr](moreadv.html#EXPRREF) or [let](internal.html#LETREF) expression._

###### Example 8-2. Using Arithmetic Operations

```bash
#!/bin/bash
# Counting to 11 in 10 different ways.

n=1; echo -n "$n "

let "n = $n + 1"   # let "n = n + 1"  also works.
echo -n "$n "


: $((n = $n + 1))
#  ":" necessary because otherwise Bash attempts
#+ to interpret "$((n = $n + 1))" as a command.
echo -n "$n "

(( n = n + 1 ))
#  A simpler alternative to the method above.
#  Thanks, David Lombard, for pointing this out.
echo -n "$n "

n=$(($n + 1))
echo -n "$n "

: $[ n = $n + 1 ]
#  ":" necessary because otherwise Bash attempts
#+ to interpret "$[ n = $n + 1 ]" as a command.
#  Works even if "n" was initialized as a string.
echo -n "$n "

n=$[ $n + 1 ]
#  Works even if "n" was initialized as a string.
#* Avoid this type of construct, since it is obsolete and nonportable.
#  Thanks, Stephane Chazelas.
echo -n "$n "

# Now for C-style increment operators.
# Thanks, Frank Wang, for pointing this out.

let "n++"          # let "++n"  also works.
echo -n "$n "

(( n++ ))          # (( ++n ))  also works.
echo -n "$n "

: $(( n++ ))       # : $(( ++n )) also works.
echo -n "$n "

: $[ n++ ]         # : $[ ++n ] also works
echo -n "$n "

echo

exit 0
```

> [!note]
> Integer variables in older versions of Bash were signed _long_ (32-bit) integers, in the range of -2147483648 to 2147483647. An operation that took a variable outside these limits gave an erroneous result.
>
> ```bash
> echo $BASH_VERSION   # 1.14
> 
> a=2147483646
> echo "a = $a"        # a = 2147483646
> let "a+=1"           # Increment "a".
> echo "a = $a"        # a = 2147483647
> let "a+=1"           # increment "a" again, past the limit.
> echo "a = $a"        # a = -2147483648
>                      #      ERROR: out of range,
>                      # +    and the leftmost bit, the sign bit,
>                      # +    has been set, making the result negative.
> ```
>
> As of version >= 2.05b, Bash supports 64-bit integers.

> [!caution]
> Bash does not understand floating point arithmetic. It treats numbers containing a decimal point as strings.
>
> ```bash
> a=1.5
> 
> let "b = $a + 1.3"  # Error.
> # t2.sh: let: b = 1.5 + 1.3: syntax error in expression
> #                            (error token is ".5 + 1.3")
> 
> echo "b = $b"       # b=1
> ```
>
> Use [[math-commands#BCREF|bc]] in scripts that that need floating point calculations or math library functions.

**bitwise operators.** The bitwise operators seldom make an appearance in shell scripts. Their chief use seems to be manipulating and testing values read from ports or [[dev#SOCKETREF|sockets]]. "Bit flipping" is more relevant to compiled languages, such as C and C++, which provide direct access to system hardware. However, see _vladz's_ ingenious use of bitwise operators in his _base64.sh_ ([[contributed-scripts#BASE64|Example A-54]]) script.

**bitwise operators**

<<

bitwise left shift (multiplies by 2 for each shift position)

<<=

_left-shift-equal_

**let "var <<= 2"** results in _var_ left-shifted 2 bits (multiplied by 4)

>>

bitwise right shift (divides by 2 for each shift position)

>>=

_right-shift-equal_ (inverse of <<=)

&

bitwise AND

&=

bitwise _AND-equal_

|

bitwise OR

|=

bitwise _OR-equal_

~

bitwise NOT

^

bitwise XOR

^=

bitwise _XOR-equal_

**logical (boolean) operators**

!

NOT

```bash
if [ ! -f $FILENAME ]
then
  ...
```

&&

AND

```bash
if [ $condition1 ] && [ $condition2 ]
#  Same as:  if [ $condition1 -a $condition2 ]
#  Returns true if both condition1 and condition2 hold true...

if [[ $condition1 && $condition2 ]]    # Also works.
#  Note that && operator not permitted inside brackets
#+ of [ ... ] construct.
```

> [!note] && may also be used, depending on context, in an [[list-constructs#LISTCONSREF|and list]] to concatenate commands.

||

OR

```bash
if [ $condition1 ] || [ $condition2 ]
# Same as:  if [ $condition1 -o $condition2 ]
# Returns true if either condition1 or condition2 holds true...

if [[ $condition1 || $condition2 ]]    # Also works.
#  Note that || operator not permitted inside brackets
#+ of a [ ... ] construct.
```

> [!note] Bash tests the [[exit-and-exit-status#EXITSTATUSREF|exit status]] of each statement linked with a logical operator.

###### Example 8-3. Compound Condition Tests Using && and ||

```bash
#!/bin/bash

a=24
b=47

if [ "$a" -eq 24 ] && [ "$b" -eq 47 ]
then
  echo "Test #1 succeeds."
else
  echo "Test #1 fails."
fi

# ERROR:   if [ "$a" -eq 24 && "$b" -eq 47 ]
#+         attempts to execute  ' [ "$a" -eq 24 '
#+         and fails to finding matching ']'.
#
#  Note:  if [[ $a -eq 24 && $b -eq 24 ]]  works.
#  The double-bracket if-test is more flexible
#+ than the single-bracket version.       
#    (The "&&" has a different meaning in line 17 than in line 6.)
#    Thanks, Stephane Chazelas, for pointing this out.


if [ "$a" -eq 98 ] || [ "$b" -eq 47 ]
then
  echo "Test #2 succeeds."
else
  echo "Test #2 fails."
fi


#  The -a and -o options provide
#+ an alternative compound condition test.
#  Thanks to Patrick Callahan for pointing this out.


if [ "$a" -eq 24 -a "$b" -eq 47 ]
then
  echo "Test #3 succeeds."
else
  echo "Test #3 fails."
fi


if [ "$a" -eq 98 -o "$b" -eq 47 ]
then
  echo "Test #4 succeeds."
else
  echo "Test #4 fails."
fi


a=rhino
b=crocodile
if [ "$a" = rhino ] && [ "$b" = crocodile ]
then
  echo "Test #5 succeeds."
else
  echo "Test #5 fails."
fi

exit 0
```

The && and || operators also find use in an arithmetic context.

```bash
bash$ echo $(( 1 && 2 )) $((3 && 0)) $((4 || 0)) $((0 || 0))
1 0 1 0
	      
```

**miscellaneous operators**

,

Comma operator

The **comma operator** chains together two or more arithmetic operations. All the operations are evaluated (with possible _side effects_. [^2]

```bash
let "t1 = ((5 + 3, 7 - 1, 15 - 4))"
echo "t1 = $t1"           ^^^^^^  # t1 = 11
# Here t1 is set to the result of the last operation. Why?

let "t2 = ((a = 9, 15 / 3))"      # Set "a" and calculate "t2".
echo "t2 = $t2    a = $a"         # t2 = 5    a = 9
```

The comma operator finds use mainly in [[loops#FORLOOPREF1|for loops]]. See [[loops.html#FORLOOPC|Example 11-13]].

## Numerical Constants

A shell script interprets a number as decimal (base 10), unless that number has a special prefix or notation. A number preceded by a _0_ is _octal_ (base 8). A number preceded by _0x_ is _hexadecimal_ (base 16). A number with an embedded _#_ evaluates as _BASE#NUMBER_ (with range and notational restrictions).

###### Example 8-4. Representation of numerical constants

```bash
#!/bin/bash
# numbers.sh: Representation of numbers in different bases.

# Decimal: the default
let "dec = 32"
echo "decimal number = $dec"             # 32
# Nothing out of the ordinary here.


# Octal: numbers preceded by '0' (zero)
let "oct = 032"
echo "octal number = $oct"               # 26
# Expresses result in decimal.
# --------- ------ -- -------


# Hexadecimal: numbers preceded by '0x' or '0X'
let "hex = 0x32"
echo "hexadecimal number = $hex"         # 50

echo $((0x9abc))                         # 39612
#     ^^      ^^   double-parentheses arithmetic expansion/evaluation
# Expresses result in decimal.



# Other bases: BASE#NUMBER
# BASE between 2 and 64.
# NUMBER must use symbols within the BASE range, see below.


let "bin = 2#111100111001101"
echo "binary number = $bin"              # 31181

let "b32 = 32#77"
echo "base-32 number = $b32"             # 231

let "b64 = 64#@_"
echo "base-64 number = $b64"             # 4031
# This notation only works for a limited range (2 - 64) of ASCII characters.
# 10 digits + 26 lowercase characters + 26 uppercase characters + @ + _


echo

echo $((36#zz)) $((2#10101010)) $((16#AF16)) $((53#1aA))
                                         # 1295 170 44822 3375


#  Important note:
#  --------------
#  Using a digit out of range of the specified base notation
#+ gives an error message.

let "bad_oct = 081"
# (Partial) error message output:
#  bad_oct = 081: value too great for base (error token is "081")
#              Octal numbers use only digits in the range 0 - 7.

exit $?   # Exit value = 1 (error)

# Thanks, Rich Bartell and Stephane Chazelas, for clarification.
```

## The Double-Parentheses Construct

Similar to the [[internal-commands-and-builtins#^LETREF|let]] command, the **(( ... ))** construct permits arithmetic expansion and evaluation. In its simplest form, **a=$(( 5 + 3 ))** would set **a** to **5 + 3**, or **8**. However, this double-parentheses construct is also a mechanism for allowing C-style manipulation of variables in Bash, for example, (( var++ )).

###### Example 8-5. C-style manipulation of variables

```bash
#!/bin/bash
# c-vars.sh
# Manipulating a variable, C-style, using the (( ... )) construct.


echo

(( a = 23 ))  #  Setting a value, C-style,
              #+ with spaces on both sides of the "=".
echo "a (initial value) = $a"   # 23

(( a++ ))     #  Post-increment 'a', C-style.
echo "a (after a++) = $a"       # 24

(( a-- ))     #  Post-decrement 'a', C-style.
echo "a (after a--) = $a"       # 23


(( ++a ))     #  Pre-increment 'a', C-style.
echo "a (after ++a) = $a"       # 24

(( --a ))     #  Pre-decrement 'a', C-style.
echo "a (after --a) = $a"       # 23

echo

########################################################
#  Note that, as in C, pre- and post-decrement operators
#+ have different side-effects.

n=1; let --n && echo "True" || echo "False"  # False
n=1; let n-- && echo "True" || echo "False"  # True

#  Thanks, Jeroen Domburg.
########################################################

echo

(( t = a<45?7:11 ))   # C-style trinary operator.
#       ^  ^ ^
echo "If a < 45, then t = 7, else t = 11."  # a = 23
echo "t = $t "                              # t = 7

echo


# -----------------
# Easter Egg alert!
# -----------------
#  Chet Ramey seems to have snuck a bunch of undocumented C-style
#+ constructs into Bash (actually adapted from ksh, pretty much).
#  In the Bash docs, Ramey calls (( ... )) shell arithmetic,
#+ but it goes far beyond that.
#  Sorry, Chet, the secret is out.

# See also "for" and "while" loops using the (( ... )) construct.

# These work only with version 2.04 or later of Bash.

exit
```

See also [[loops#^FORLOOPC|Example 11-13]] and [[operations-and-related-topics#^NUMBERS|Example 8-4]].

## Operator Precedence

In a script, operations execute in order of _precedence_: the higher precedence operations execute _before_ the lower precedence ones. [^3]

| Operator | Meaning | Comments |
| :--- | :--- | :--- |
|  |  | **HIGHEST PRECEDENCE** |
| var++ var-- | post-increment, post-decrement | [[assorted-tips#^CSTYLE\|C-style]] operators |
| ++var --var | pre-increment, pre-decrement |  |
|  |  |  |
| ! ~ | [[special-chars#NOTREF\|negation]] | logical / bitwise, inverts sense of following operator |
|  |  |  |
| ** | [[ops#EXPONENTIATIONREF\|exponentiation]] | [[ops#AROPS1\|arithmetic operation]] |
| * / % | multiplication, division, modulo | arithmetic operation |
| + - | addition, subtraction | arithmetic operation |
|  |  |  |
| << >> | left, right shift | [[ops#BITWSOPS1\|bitwise]] |
|  |  |  |
| -z -n | _unary_ comparison | string is/is-not [[tests#STRINGNULL\|null]] |
| -e -f -t -x, etc. | _unary_ comparison | [[fto\|file-test]] |
| < -lt > -gt <= -le >= -ge | _compound_ comparison | string and integer |
| -nt -ot -ef | _compound_ comparison | file-test |
| == -eq [[tests#NOTEQUAL\|!=]] -ne | equality / inequality | test operators, string and integer |
|  |  |  |
| & | AND | bitwise |
| ^ | XOR | _exclusive_ OR, bitwise |
| \| | OR | bitwise |
|  |  |  |
| && -a | AND | [[ops#LOGOPS1\|logical]], _compound_ comparison |
| \| -o | OR | logical, _compound_ comparison |
|  |  |  |
| ?: | [[special-chars#CSTRINARY\|trinary operator]] | C-style |
| = | [[varassignment#EQREF\|assignment]] | (do not confuse with equality _test_) |
| *= /= %= += -= <<= >>= &= | [[ops#ARITHOPSCOMB\|combination assignment]] | times-equal, divide-equal, mod-equal, etc. |
|  |  |  |
| , | [[ops#COMMAOP\|comma]] | links a sequence of operations |
|  |  | **LOWEST PRECEDENCE** |
**Table 8-1. Operator Precedence** ^1

In practice, all you really need to remember is the following:

- The "My Dear Aunt Sally" mantra (_multiply, divide, add, subtract_) for the familiar [[ops#AROPS1|arithmetic operations]].
- The _compound_ logical operators, **&&**, **||**, **-a**, and **-o** have low precedence.
- The order of evaluation of equal-precedence operators is usually _left-to-right_.

Now, let's utilize our knowledge of operator precedence to analyze a couple of lines from the /etc/init.d/functions file, as found in the _Fedora Core_ Linux distro.

```bash
while [ -n "$remaining" -a "$retry" -gt 0 ]; do

# This looks rather daunting at first glance.


# Separate the conditions:
while [ -n "$remaining" -a "$retry" -gt 0 ]; do
#       --condition 1-- ^^ --condition 2-

#  If variable "$remaining" is not zero length
#+      AND (-a)
#+ variable "$retry" is greater-than zero
#+ then
#+ the [ expresion-within-condition-brackets ] returns success (0)
#+ and the while-loop executes an iteration.
#  ==============================================================
#  Evaluate "condition 1" and "condition 2" ***before***
#+ ANDing them. Why? Because the AND (-a) has a lower precedence
#+ than the -n and -gt operators,
#+ and therefore gets evaluated *last*.

#################################################################

if [ -f /etc/sysconfig/i18n -a -z "${NOLOCALE:-}" ] ; then


# Again, separate the conditions:
if [ -f /etc/sysconfig/i18n -a -z "${NOLOCALE:-}" ] ; then
#    --condition 1--------- ^^ --condition 2-----

#  If file "/etc/sysconfig/i18n" exists
#+      AND (-a)
#+ variable $NOLOCALE is zero length
#+ then
#+ the [ test-expresion-within-condition-brackets ] returns success (0)
#+ and the commands following execute.
#
#  As before, the AND (-a) gets evaluated *last*
#+ because it has the lowest precedence of the operators within
#+ the test brackets.
#  ==============================================================
#  Note:
#  ${NOLOCALE:-} is a parameter expansion that seems redundant.
#  But, if $NOLOCALE has not been declared, it gets set to *null*,
#+ in effect declaring it.
#  This makes a difference in some contexts.
```

> [!tip]
> To avoid confusion or error in a complex sequence of test operators, break up the sequence into bracketed sections.
>
> ```bash
> if [ "$v1" -gt "$v2"  -o  "$v1" -lt "$v2"  -a  -e "$filename" ]
> # Unclear what's going on here...
> 
> if [[ "$v1" -gt "$v2" ]] || [[ "$v1" -lt "$v2" ]] && [[ -e "$filename" ]]
> # Much better -- the condition tests are grouped in logical sections.
> ```

[^1]: In a different context, **+=** can serve as a _string concatenation_ operator. This can be useful for [[bash-version-3#^PATHAPPEND|modifying _environmental variables_]].

[^2]: _Side effects_ are, of course, unintended -- and usually undesirable -- consequences.

[^3]: _Precedence_, in this context, has approximately the same meaning as _priority_
