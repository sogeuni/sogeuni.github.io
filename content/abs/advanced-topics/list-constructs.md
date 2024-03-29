---
title: 26. List Constructs
---


The _and list_ and _or list_ constructs provide a means of processing a number of commands consecutively. These can effectively replace complex nested [[testconstructs.html#TESTCONSTRUCTS1|if/then]] or even [[testbranch.html#CASEESAC1|case]] statements.

**Chaining together commands**

and list

```bash
command-1 && command-2 && command-3 && ... command-n
```

Each command executes in turn, provided that the previous command has given a return value of _true_ (zero). At the first _false_ (non-zero) return, the command chain terminates (the first command returning _false_ is the last one to execute).

An interesting use of a two-condition _and list_ from an early version of YongYe's [Tetris game script](http://bash.deta.in/Tetris_Game.sh):

```bash
equation()

{  # core algorithm used for doubling and halving the coordinates
   [[ ${cdx} ]] && ((y=cy+(ccy-cdy)${2}2))
   eval ${1}+=\"${x} ${y} \"
}
```

![[Example 26-1|Example 26-1]]

![[Example 26-2|Example 26-2]]

Of course, an _and list_ can also _set_ variables to a default value.

```bash
arg1=$@ && [ -z "$arg1" ] && arg1=DEFAULT
        
              # Set $arg1 to command-line arguments, if any.
              # But . . . set to DEFAULT if not specified on command-line.
```

or list

```bash
command-1 | command-2 | command-3 | ... command-n
```

Each command executes in turn for as long as the previous command returns false. At the first true return, the command chain terminates (the first command returning true is the last one to execute). This is obviously the inverse of the "and list".

![[Example 26-3|Example 26-3]]

> [!caution]
> If the first command in an _or list_ returns true, it _will_ execute.

```bash
# ==> The following snippets from the /etc/rc.d/init.d/single
#+==> script by Miquel van Smoorenburg
#+==> illustrate use of "and" and "or" lists.
# ==> "Arrowed" comments added by document author.

[ -x /usr/bin/clear ] && /usr/bin/clear
  # ==> If /usr/bin/clear exists, then invoke it.
  # ==> Checking for the existence of a command before calling it
  #+==> avoids error messages and other awkward consequences.

  # ==> . . .

# If they want to run something in single user mode, might as well run it...
for i in /etc/rc1.d/S[0-9][0-9]* ; do
        # Check if the script is there.
        [ -x "$i" ] | continue
  # ==> If corresponding file in $PWD *not* found,
  #+==> then "continue" by jumping to the top of the loop.

        # Reject backup files and files generated by rpm.
        case "$1" in
                *.rpmsave|*.rpmorig|*.rpmnew|*~|*.orig)
                        continue;;
        esac
        [ "$i" = "/etc/rc1.d/S00single" ] && continue
  # ==> Set script name, but don't execute it yet.
        $i start
done

  # ==> . . .
```

> [!important]
> The [[exit-and-exit-status#^EXITSTATUSREF|exit status]] of an **and list** or an **or list** is the exit status of the last command executed.

Clever combinations of _and_ and _or_ lists are possible, but the logic may easily become convoluted and require close attention to [[operations-and-related-topics#^OPPRECEDENCE1|operator precedence rules]], and possibly extensive debugging.

```bash
false && true | echo false         # false

# Same result as
( false && true ) | echo false     # false
# But NOT
false && ( true | echo false )     # (nothing echoed)

#  Note left-to-right grouping and evaluation of statements.

#  It's usually best to avoid such complexities.

#  Thanks, S.C.
```

See [contributed-scripts#^DAYSBETWEEN|Example A-7]] and [[Example 7-4|Example 7-4]] for illustrations of using **and / or list** constructs to test variables.
