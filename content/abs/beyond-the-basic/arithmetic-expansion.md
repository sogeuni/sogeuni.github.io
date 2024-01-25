---
title: 13. Arithmetic Expansion
---


Arithmetic expansion provides a powerful tool for performing (integer) arithmetic operations in scripts. Translating a string into a numerical expression is relatively straightforward using _backticks_, _double parentheses_, or _let_. ^ARITHEXPREF

**Variations**

Arithmetic expansion with [[command-substitution#^BACKQUOTESREF|backticks]] (often used in conjunction with [[complex-commands#^EXPRREF|expr]])

```bash
z=`expr $z + 3`          # The 'expr' command performs the expansion.
```

Arithmetic expansion with [[operations-and-related-topics|double parentheses]], and using [[internal-commands-and-builtins#^LETREF|let]]

The use of _backticks_ (_backquotes_) in arithmetic expansion has been superseded by _double parentheses_ -- **((...))** and **$((...))** -- and also by the very convenient [[internal-commands-and-builtins#^LETREF|let]] construction.

```bash
z=$(($z+3))
z=$((z+3))                                  #  Also correct.
                                            #  Within double parentheses,
                                            #+ parameter dereferencing
                                            #+ is optional.

# $((EXPRESSION)) is arithmetic expansion.  #  Not to be confused with
                                            #+ command substitution.



# You may also use operations within double parentheses without assignment.

  n=0
  echo "n = $n"                             # n = 0

  (( n += 1 ))                              # Increment.
# (( $n += 1 )) is incorrect!
  echo "n = $n"                             # n = 1


let z=z+3
let "z += 3"  #  Quotes permit the use of spaces in variable assignment.
              #  The 'let' operator actually performs arithmetic evaluation,
              #+ rather than expansion.
```

Examples of arithmetic expansion in scripts:

1. [[complex-commands#^EX45|Example 16-9]]
2. [[loops#^EX25|Example 11-15]]
3. [[arrays#^EX66|Example 27-1]]
4. [[arrays#^BUBBLE|Example 27-11]]
5. [[contributed-scripts#^TREE|Example A-16]]
