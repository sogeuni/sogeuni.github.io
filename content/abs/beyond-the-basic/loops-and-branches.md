---
title: 11. Loops and Branches
---


> What needs this iteration, woman?
>
> --<cite>Shakespeare, _Othello_</cite>

Operations on code blocks are the key to structured and organized shell scripts. Looping and branching constructs provide the tools for accomplishing this.

## Loops

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

![[Example 11-1|Example 11-1]]

Each **[list]** element may contain multiple parameters. This is useful when processing parameters in groups. In such cases, use the [[internal-commands-and-builtins#SETREF|set]] command (see [[Example 15-16|Example 15-16]]) to force parsing of each **[list]** element and assignment of each component to the positional parameters.

![[Example 11-2|Example 11-2]]

A variable may supply the **[list]** in a _for loop_.

![[Example 11-3|Example 11-3]]

The **[list]** in a _for loop_ may be parameterized.

![[Example 11-4|Example 11-4]]

If the **[list]** in a _for loop_ contains wild cards (* and ?) used in filename expansion, then [[globbing|globbing]] takes place.

![[Example 11-5|Example 11-5]]

Omitting the **in [list]** part of a _for loop_ causes the loop to operate on $@ -- the [[internalvariables#POSPARAMREF|positional parameters]]. A particularly clever illustration of this is [[Example A-15|Example A-15]]. See also [[Example 15-17|Example 15-17]].

![[Example 11-6|Example 11-6]]

It is possible to use [[command-substitution#COMMANDSUBREF|command substitution]] to generate the **[list]** in a _for loop_. See also [[Example 16-54|Example 16-54]], [[Example 11-11|Example 11-11]] and [[Example 16-48|Example 16-48]].

![[Example 11-7|Example 11-7]]

Here is a somewhat more complex example of using command substitution to create the **[list]**.

![[Example 11-8|Example 11-8]]

More of the same.

![[Example 11-9|Example 11-9]]

Yet another example of the **[list]** resulting from command substitution.

![[Example 11-10|Example 11-10]]

A final example of **[list]** / command substitution, but this time the "command" is a [[functions|function]].

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

![[Example 11-11|Example 11-11]]

The stdout of a loop may be [[io-redirection|redirected]] to a file, as this slight modification to the previous example shows.

![[Example 11-12|Example 11-12]]

There is an alternative syntax to a _for loop_ that will look very familiar to C programmers. This requires [[dblparens#DBLPARENSREF|double parentheses]].

![[Example 11-13|Example 11-13]]

See also [[Example 27-16|Example 27-16]], [[Example 27-17|Example 27-17]], and [[Example A-6|Example A-6]].

---

Now, a _for loop_ used in a "real-life" context.

![[Example 11-14|Example 11-14]]

> [!note]
> The [[internal-commands-and-builtins#keywordref|keywords]] **do** and **done** delineate the _for-loop_ command block. However, these may, in certain contexts, be omitted by framing the command block within [[special-chars#CODEBLOCKREF|curly brackets]]
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

Note that the _test brackets_ [[loops1#WHILENOBRACKETS|are _not_ mandatory]] in a _while_ loop. See, for example, the [[internal-commands-and-builtins#GETOPTSX|getopts construct]].

![[Example 11-15|Example 11-15]]

![[Example 11-16|Example 11-16]]

A _while loop_ may have multiple conditions. Only the final condition determines when the loop terminates. This necessitates a slightly different loop syntax, however.

![[Example 11-17|Example 11-17]]

As with a _for loop_, a _while loop_ may employ C-style syntax by using the double-parentheses construct (see also [[Example 8-5|Example 8-5]]).

![[Example 11-18|Example 11-18]]

Inside its test brackets, a _while loop_ can call a [[functions|function]].

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

By coupling the power of the [[internal-commands-and-builtins#READREF|read]] command with a _while loop_, we get the handy [[internal-commands-and-builtins#WHILEREADREF|while read]] construct, useful for reading and parsing files.

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
> A _while loop_ may have its stdin [[redirecting-code-blocks#REDIRREF|redirected to a file]] by a < at its end.
>
> A _while loop_ may have its stdin [[internal-commands-and-builtins#READPIPEREF|supplied by a pipe]].

**until**

This construct tests for a condition at the top of a loop, and keeps looping as long as that condition is _false_ (opposite of _while loop_).

**until** [ _condition-is-true_ ]  
do
 command(s)...
done

Note that an _until loop_ tests for the terminating condition at the _top_ of the loop, differing from a similar construct in some programming languages.

As is the case with _for loops_, placing the _do_ on the same line as the condition test requires a semicolon.

**until** [ _condition-is-true_ ] ; do

![[Example 11-19|Example 11-19]]

How to choose between a _for_ loop or a _while_ loop or _until_ loop? In **C**, you would typically use a _for_ loop when the number of loop iterations is known beforehand. With _Bash_, however, the situation is fuzzier. The Bash _for_ loop is more loosely structured and more flexible than its equivalent in other languages. Therefore, feel free to use whatever type of loop gets the job done in the simplest way.

## Nested Loops

A _nested loop_ is a loop within a loop, an inner loop within the body of an outer one. How this works is that the first pass of the outer loop triggers the inner loop, which executes to completion. Then the second pass of the outer loop triggers the inner loop again. This repeats until the outer loop finishes. Of course, a _break_ within either the inner or outer loop would interrupt this process.

![[Example 11-20|Example 11-20]]

See [[Example 27-11|Example 27-11]] for an illustration of nested [[loops-and-branches#^WHILELOOPREF|while loops]], and [[Example 27-13|Example 27-13]] to see a while loop nested inside an [[loops-and-branches#^UNTILLOOPREF|until loop]].

## Loop Control

> Tournez cent tours, tournez mille tours,
>
> Tournez souvent et tournez toujours . . .
>
> --<cite>Verlaine, "Chevaux de bois"</cite>

**Commands affecting loop behavior**

**break**, **continue**

The **break** and **continue** loop control commands [^2] correspond exactly to their counterparts in other programming languages. The **break** command terminates the loop (_breaks_ out of it), while **continue** causes a jump to the next [[loops-and-branches#^ITERATIONREF|iteration]] of the loop, skipping all the remaining commands in that particular loop cycle.

![[Example 11-21|Example 11-21]]

The **break** command may optionally take a parameter. A plain **break** terminates only the innermost loop in which it is embedded, but a **break N** breaks out of _N_ levels of loop.

![[Example 11-22|Example 11-22]]

The **continue** command, similar to **break**, optionally takes a parameter. A plain **continue** cuts short the current iteration within its loop and begins the next. A **continue N** terminates all remaining iterations at its loop level and continues with the next iteration at the loop, N levels above.

![[Example 11-23|Example 11-23]]

![[Example 11-24|Example 11-24]]

> [!caution] The **continue N** construct is difficult to understand and tricky to use in any meaningful context. It is probably best avoided.

## Testing and Branching

The **case** and **select** constructs are technically not loops, since they do not iterate the execution of a code block. Like loops, however, they direct program flow according to conditions at the top or bottom of the block.

**Controlling program flow in a code block**

**case (in) / esac**

The **case** construct is the shell scripting analog to _switch_ in **C/C++**. It permits branching to one of a number of code blocks, depending on condition tests. It serves as a kind of shorthand for multiple if/then/else statements and is an appropriate tool for creating menus.

case "$variable" in

 "$condition1" )
 command...
 ;;

 "$condition2" )
 command...
 ;;
  
  
**esac**

> [!note]
> - Quoting the variables is not mandatory, since word splitting does not take place.
> - Each test line ends with a right paren **)**. [^3]
> - Each condition block ends with a _double_ semicolon ;;.
> - If a condition tests _true_, then the associated commands execute and the **case** block terminates.
> - The entire **case** block ends with an **esac** (_case_ spelled backwards).|

![[Example 11-25|Example 11-25]]

![[Example 11-26|Example 11-26]]

An exceptionally clever use of **case** involves testing for command-line parameters.

```bash
#! /bin/bash

case "$1" in
  "") echo "Usage: ${0##*/} <filename>"; exit $E_PARAM;;
                      # No command-line parameters,
                      # or first parameter empty.
# Note that ${0##*/} is ${var##pattern} param substitution.
                      # Net result is $0.

  -*) FILENAME=./$1;;   #  If filename passed as argument ($1)
                      #+ starts with a dash,
                      #+ replace it with ./$1
                      #+ so further commands don't interpret it
                      #+ as an option.

  * ) FILENAME=$1;;     # Otherwise, $1.
esac
```

Here is a more straightforward example of command-line parameter handling:

```bash
#! /bin/bash


while [ $# -gt 0 ]; do    # Until you run out of parameters . . .
  case "$1" in
    -d|--debug)
              # "-d" or "--debug" parameter?
              DEBUG=1
              ;;
    -c|--conf)
              CONFFILE="$2"
              shift
              if [ ! -f $CONFFILE ]; then
                echo "Error: Supplied file doesn't exist!"
                exit $E_CONFFILE     # File not found error.
              fi
              ;;
  esac
  shift       # Check next set of parameters.
done

#  From Stefano Falsetto's "Log2Rot" script,
#+ part of his "rottlog" package.
#  Used with permission.
```

![[Example 11-27|Example 11-27]]

A **case** construct can filter strings for [[globbing.html|globbing]] patterns.

![[Example 11-28|Example 11-28]]

![[Example 11-29|Example 11-29]]

**select**

The **select** construct, adopted from the Korn Shell, is yet another tool for building menus.

select variable [in list]
do
 command...
 break
done

This prompts the user to enter one of the choices presented in the variable list. Note that **select** uses the $PS3 prompt (#? ) by default, but this may be changed.

![[Example 11-30|Example 11-30]]

If **in _list_** is omitted, then **select** uses the list of command line arguments ($@) passed to the script or the function containing the **select** construct.

Compare this to the behavior of a

**for** _variable_ [in _list_]

construct with the **in _list_** omitted.

![[Example 11-31.md|Example 11-31.md]]

See also [[Example 37-3|Example 37-3]].

[^1]: _Iteration_: Repeated execution of a command or group of commands, usually -- but not always, _while_ a given condition holds, or _until_ a given condition is met.

[^2]: These are shell [[internal-commands-and-builtins|builtins]], whereas other loop commands, such as [[loops1#^WHILELOOPREF|while]] and [[loops-and-branches#^CASEESAC1|case]], are [[internal-commands-and-builtins#^keywordref|keywords]].

[^3]: Pattern-match lines may also _start_ with a **(** left paren to give the layout a more structured appearance.

    ```bash
    case $( arch ) in   # $( arch ) returns machine architecture.
      ( i386 ) echo "80386-based machine";;
    # ^      ^
      ( i486 ) echo "80486-based machine";;
      ( i586 ) echo "Pentium-based machine";;
      ( i686 ) echo "Pentium2+-based machine";;
      (    * ) echo "Other type of machine";;
    esac
    ```
