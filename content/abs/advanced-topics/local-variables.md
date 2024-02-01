---
title: 24.2. Local Variables
---


**What makes a variable _local_?**

local variables

A variable declared as _local_ is one that is visible only within the [[special-chars.html#CODEBLOCKREF|block of code]] in which it appears. It has local [[subshells.html#SCOPEREF|scope]]. In a function, a _local variable_ has meaning only within that function block. [^1]

![[Example 24-12|Example 24-12]]

> [!caution]
> Before a function is called, _all_ variables declared within the function are invisible outside the body of the function, not just those explicitly declared as _local_.
>
> ```bash
> #!/bin/bash
> 
> func ()
> {
> global_var=37    #  Visible only within the function block
>                  #+ before the function has been called. 
> }                #  END OF FUNCTION
> 
> echo "global_var = $global_var"  # global_var =
>                                  #  Function "func" has not yet been called,
>                                  #+ so $global_var is not visible here.
> 
> func
> echo "global_var = $global_var"  # global_var = 37
>                                  # Has been set by function call.
> ```

> [!note]
> As Evgeniy Ivanov points out, when declaring and setting a local variable in a single command, apparently the order of operations is to _first set the variable, and only afterwards restrict it to local scope_. This is reflected in the [[exit-and-exit-status.html#EXITSTATUSREF|return value]].
>
> ```bash
> #!/bin/bash
> 
> echo "==OUTSIDE Function (global)=="
> t=$(exit 1)
> echo $?      # 1
>              # As expected.
> echo
> 
> function0 ()
> {
> 
> echo "==INSIDE Function=="
> echo "Global"
> t0=$(exit 1)
> echo $?      # 1
>              # As expected.
> 
> echo
> echo "Local declared & assigned in same command."
> local t1=$(exit 1)
> echo $?      # 0
>              # Unexpected!
> #  Apparently, the variable assignment takes place before
> #+ the local declaration.
> #+ The return value is for the latter.
> 
> echo
> echo "Local declared, then assigned (separate commands)."
> local t2
> t2=$(exit 1)
> echo $?      # 1
>              # As expected.
> 
> }
> 
> function0
> ```

## 24.2.1. Local variables and recursion.

> _Recursion_ is an interesting and sometimes useful form of _self-reference_. [Herbert Mayer](biblio.html#MAYERREF) defines it as ". . . expressing an algorithm by using a simpler version of that same algorithm . . ."
>
> Consider a definition defined in terms of itself, [^2] an expression implicit in its own expression, [^3] _a snake swallowing its own tail_, [^4] or . . . a function that calls itself. [^5]
>
> **Example 24-13. Demonstration of a simple recursive function**
>
> ```bash
> #!/bin/bash
> # recursion-demo.sh
> # Demonstration of recursion.
> 
> RECURSIONS=9   # How many times to recurse.
> r_count=0      # Must be global. Why?
> 
> recurse ()
> {
>   var="$1"
> 
>   while [ "$var" -ge 0 ]
>   do
>     echo "Recursion count = "$r_count"  +-+  \$var = "$var""
>     (( var-- )); (( r_count++ ))
>     recurse "$var"  #  Function calls itself (recurses)
>   done              #+ until what condition is met?
> }
> 
> recurse $RECURSIONS
> 
> exit $?
> ```
>
> **Example 24-14. Another simple demonstration**
>
> ```bash
> #!/bin/bash
> # recursion-def.sh
> # A script that defines "recursion" in a rather graphic way.
> 
> RECURSIONS=10
> r_count=0
> sp=" "
> 
> define_recursion ()
> {
>   ((r_count++))
>   sp="$sp"" "
>   echo -n "$sp"
>   echo "\"The act of recurring ... \""   # Per 1913 Webster's dictionary.
> 
>   while [ $r_count -le $RECURSIONS ]
>   do
>     define_recursion
>   done
> }
> 
> echo
> echo "Recursion: "
> define_recursion
> echo
> 
> exit $?
> ```
^RECURSIONREF0

Local variables are a useful tool for writing recursive code, but this practice generally involves a great deal of computational overhead and is definitely _not_ recommended in a shell script. [^6]

**Example 24-15.** Recursion, using a local variable

```bash
#!/bin/bash

#               factorial
#               ---------


# Does bash permit recursion?
# Well, yes, but...
# It's so slow that you gotta have rocks in your head to try it.


MAX_ARG=5
E_WRONG_ARGS=85
E_RANGE_ERR=86


if [ -z "$1" ]
then
  echo "Usage: `basename $0` number"
  exit $E_WRONG_ARGS
fi

if [ "$1" -gt $MAX_ARG ]
then
  echo "Out of range ($MAX_ARG is maximum)."
  #  Let's get real now.
  #  If you want greater range than this,
  #+ rewrite it in a Real Programming Language.
  exit $E_RANGE_ERR
fi  

fact ()
{
  local number=$1
  #  Variable "number" must be declared as local,
  #+ otherwise this doesn't work.
  if [ "$number" -eq 0 ]
  then
    factorial=1    # Factorial of 0 = 1.
  else
    let "decrnum = number - 1"
    fact $decrnum  # Recursive function call (the function calls itself).
    let "factorial = $number * $?"
  fi

  return $factorial
}

fact $1
echo "Factorial of $1 is $?."

exit 0
```

Also see [[contributed-scripts.html#PRIMES|Example A-15]] for an example of recursion in a script. Be aware that recursion is resource-intensive and executes slowly, and is therefore generally not appropriate in a script.

[^1]: However, as Thomas Braunberger points out, a local variable declared in a function _is also visible to functions called by the parent function._

    ```bash
    #!/bin/bash

    function1 ()
    {
    local func1var=20

    echo "Within function1, \$func1var = $func1var."

    function2
    }

    function2 ()
    {
    echo "Within function2, \$func1var = $func1var."
    }

    function1

    exit 0


    # Output of the script:

    # Within function1, $func1var = 20.
    # Within function2, $func1var = 20.
    ```
    This is documented in the Bash manual:

    "Local can only be used within a function; it makes the variable name have a visible scope restricted to that function _and its children_." [emphasis added] _The ABS Guide author considers this behavior to be a bug._

[^2]: Otherwise known as _redundancy_.

[^3]: Otherwise known as _tautology_.

[^4]: Otherwise known as a _metaphor_.

[^5]: Otherwise known as a _recursive function_.

[^6]: Too many levels of recursion may crash a script with a segfault.

    ```bash
    #!/bin/bash

    #  Warning: Running this script could possibly lock up your system!
    #  If you're lucky, it will segfault before using up all available memory.

    recursive_function ()		   
    {
    echo "$1"     # Makes the function do something, and hastens the segfault.
    (( $1 < $2 )) && recursive_function $(( $1 + 1 )) $2;
    #  As long as 1st parameter is less than 2nd,
    #+ increment 1st and recurse.
    }

    recursive_function 1 50000  # Recurse 50,000 levels!
    #  Most likely segfaults (depending on stack size, set by ulimit -m).

    #  Recursion this deep might cause even a C program to segfault,
    #+ by using up all the memory allotted to the stack.


    echo "This will probably not print."
    exit 0  # This script will not exit normally.

    #  Thanks, Stéphane Chazelas.
    ```
