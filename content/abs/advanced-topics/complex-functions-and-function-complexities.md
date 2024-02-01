---
title: 24.1. Complex Functions and Function Complexities
---


Functions may process arguments passed to them and return an [[exit-and-exit-status#^EXITSTATUSREF|exit status]] to the script for further processing.

```bash
function_name $arg1 $arg2
```

The function refers to the passed arguments by position (as if they were [[another-look-at-variables#^POSPARAMREF|positional parameters]]), that is, $1, $2, and so forth.

![[Example 24-2|Example 24-2]]

> [!important]
> The [[othertypesv#^SHIFTREF|shift]] command works on arguments passed to functions (see [[Example 36-18|Example 36-18]]).

But, what about command-line arguments passed to the script? Does a function see them? Well, let's clear up the confusion.

![[Example 24-3|Example 24-3]]

In contrast to certain other programming languages, shell scripts normally pass only value parameters to functions. Variable names (which are actually _pointers_), if passed as parameters to functions, will be treated as string literals. _Functions interpret their arguments literally._

[[indirect-references#^IVRREF|Indirect variable references]] (see [[Example 37-2|Example 37-2]]) provide a clumsy sort of mechanism for passing variable pointers to functions.

![[Example 24-4|Example 24-4]]

The next logical question is whether parameters can be dereferenced _after_ being passed to a function.

![[Example 24-5|Example 24-5]]

![[Example 24-6|Example 24-6]]

**Exit and Return**

**exit status**

Functions return a value, called an _exit status_. This is analogous to the [[exit-and-exit-status#^EXITSTATUSREF|exit status]] returned by a command. The exit status may be explicitly specified by a **return** statement, otherwise it is the exit status of the last command in the function (0 if successful, and a non-zero error code if not). This [[exit-and-exit-status#^EXITSTATUSREF|exit status]] may be used in the script by referencing it as [[another-look-at-variables#^XSTATVARREF|$?]]. This mechanism effectively permits script functions to have a "return value" similar to C functions.

**return**

Terminates a function. A **return** command [^1] optionally takes an _integer_ argument, which is returned to the calling script as the "exit status" of the function, and this exit status is assigned to the variable [[another-look-at-variables#^XSTATVARREF|$?]].

![[Example 24-7|Example 24-7]]

> [!tip]
> For a function to return a string or array, use a dedicated variable.
>
> ```bash
> count_lines_in_etc_passwd()
> {
>   [[ -r /etc/passwd ]] && REPLY=$(echo $(wc -l < /etc/passwd))
>   #  If /etc/passwd is readable, set REPLY to line count.
>   #  Returns both a parameter value and status information.
>   #  The 'echo' seems unnecessary, but . . .
>   #+ it removes excess whitespace from the output.
> }
> 
> if count_lines_in_etc_passwd
> then
>   echo "There are $REPLY lines in /etc/passwd."
> else
>   echo "Cannot count lines in /etc/passwd."
> fi  
> 
> # Thanks, S.C.
> ```

![[Example 24-8|Example 24-8]]

See also [[Example 11-29|Example 11-29]].

> [!important]
> The largest positive integer a function can return is 255. The **return** command is closely tied to the concept of [[exit-and-exit-status#^EXITSTATUSREF|exit status]], which accounts for this particular limitation. Fortunately, there are various [[assortedtips#^RVT|workarounds]] for those situations requiring a large integer return value from a function.
>
> **Example 24-9. Testing large return values in a function**
>
> ```bash
> #!/bin/bash
> # return-test.sh
> 
> # The largest positive value a function can return is 255.
> 
> return_test ()         # Returns whatever passed to it.
> {
>   return $1
> }
> 
> return_test 27         # o.k.
> echo $?                # Returns 27.
>   
> return_test 255        # Still o.k.
> echo $?                # Returns 255.
> 
> return_test 257        # Error!
> echo $?                # Returns 1 (return code for miscellaneous error).
> 
> # =========================================================
> return_test -151896    # Do large negative numbers work?
> echo $?                # Will this return -151896?
>                        # No! It returns 168.
> #  Version of Bash before 2.05b permitted
> #+ large negative integer return values.
> #  It happened to be a useful feature.
> #  Newer versions of Bash unfortunately plug this loophole.
> #  This may break older scripts.
> #  Caution!
> # =========================================================
> 
> exit 0
> ```
>
> A workaround for obtaining large integer "return values" is to simply assign the "return value" to a global variable.
>
> ```bash
> Return_Val=   # Global variable to hold oversize return value of function.
> 
> alt_return_test ()
> {
>   fvar=$1
>   Return_Val=$fvar
>   return   # Returns 0 (success).
> }
> 
> alt_return_test 1
> echo $?                              # 0
> echo "return value = $Return_Val"    # 1
> 
> alt_return_test 256
> echo "return value = $Return_Val"    # 256
> 
> alt_return_test 257
> echo "return value = $Return_Val"    # 257
> 
> alt_return_test 25701
> echo "return value = $Return_Val"    #25701
> ```
>
> A more elegant method is to have the function **echo** its "return value to stdout," and then capture it by [[command-substitution#^COMMANDSUBREF|command substitution]]. See the [[assorted-tips#^RVT|discussion of this]] in [[assorted-tips|Section 36.7]].
>
> **Example 24-10. Comparing two large integers**
>
> ```bash
> #!/bin/bash
> # max2.sh: Maximum of two LARGE integers.
> 
> #  This is the previous "max.sh" example,
> #+ modified to permit comparing large integers.
> 
> EQUAL=0             # Return value if both params equal.
> E_PARAM_ERR=-99999  # Not enough params passed to function.
> #           ^^^^^^    Out of range of any params that might be passed.
> 
> max2 ()             # "Returns" larger of two numbers.
> {
> if [ -z "$2" ]
> then
>   echo $E_PARAM_ERR
>   return
> fi
> 
> if [ "$1" -eq "$2" ]
> then
>   echo $EQUAL
>   return
> else
>   if [ "$1" -gt "$2" ]
>   then
>     retval=$1
>   else
>     retval=$2
>   fi
> fi
> 
> echo $retval        # Echoes (to stdout), rather than returning value.
>                     # Why?
> }
> 
> 
> return_val=$(max2 33001 33997)
> #            ^^^^             Function name
> #                 ^^^^^ ^^^^^ Params passed
> #  This is actually a form of command substitution:
> #+ treating a function as if it were a command,
> #+ and assigning the stdout of the function to the variable "return_val."
> 
> 
> # ========================= OUTPUT ========================
> if [ "$return_val" -eq "$E_PARAM_ERR" ]
>   then
>   echo "Error in parameters passed to comparison function!"
> elif [ "$return_val" -eq "$EQUAL" ]
>   then
>     echo "The two numbers are equal."
> else
>     echo "The larger of the two numbers is $return_val."
> fi
> # =========================================================
>   
> exit 0
> 
> #  Exercises:
> #  ---------
> #  1) Find a more elegant way of testing
> #+    the parameters passed to the function.
> #  2) Simplify the if/then structure at "OUTPUT."
> #  3) Rewrite the script to take input from command-line parameters.
> ```
>
> Here is another example of capturing a function "return value." Understanding it requires[[C.2.%20Awk.md#^AWKREF|C.2. Awk]] [[awk#^AWKREF|awk]].
>
> ```bash
> month_length ()  # Takes month number as an argument.
> {                # Returns number of days in month.
> monthD="31 28 31 30 31 30 31 31 30 31 30 31"  # Declare as local?
> echo "$monthD" | awk '{ print $'"${1}"' }'    # Tricky.
> #                             ^^^^^^^^^
> # Parameter passed to function  ($1 -- month number), then to awk.
> # Awk sees this as "print $1 . . . print $12" (depending on month number)
> # Template for passing a parameter to embedded awk script:
> #                                 $'"${script_parameter}"'
> 
> #    Here's a slightly simpler awk construct:
> #    echo $monthD | awk -v month=$1 '{print $(month)}'
> #    Uses the -v awk option, which assigns a variable value
> #+   prior to execution of the awk program block.
> #    Thank you, Rich.
> 
> #  Needs error checking for correct parameter range (1-12)
> #+ and for February in leap year.
> }
> 
> # ----------------------------------------------
> # Usage example:
> month=4        # April, for example (4th month).
> days_in=$(month_length $month)
> echo $days_in  # 30
> # ----------------------------------------------
> ```
>
> See also [[Example A-7|Example A-7]] and [[Example A-37|Example A-37]].
>
> **Exercise:** Using what we have just learned, extend the previous [[complex-functions-and-function-complexities#^EX61|Roman numerals example]] to accept arbitrarily large input.

**Redirection**

_Redirecting the stdin of a function_

A function is essentially a [[special-characters#^CODEBLOCKREF|code block]], which means its stdin can be redirected (as in [[Example 3-1|Example 3-1]]).

![[Example 24-11|Example 24-11]]

There is an alternate, and perhaps less confusing method of redirecting a function's stdin. This involves redirecting the stdin to an embedded bracketed code block within the function.

```bash
# Instead of:
Function ()
{
 ...
 } < file

# Try this:
Function ()
{
  {
    ...
   } < file
}

# Similarly,

Function ()  # This works.
{
  {
   echo $*
  } | tr a b
}

Function ()  # This doesn't work.
{
  echo $*
} | tr a b   # A nested code block is mandatory here.


# Thanks, S.C.
```

> [!note]
> Emmanuel Rouat's [[sample-bashrc-and-bash-profile-files.html|sample bashrc file]] contains some instructive examples of functions.

[^1]: The **return** command is a Bash [[internal-commands-and-builtins|builtin]].
