---
title: 16.8. Math Commands
---


**"Doing the numbers"**

**factor**

Decompose an integer into prime factors.

```bash
bash$ factor 27417
27417: 3 13 19 37
```

![[Example 16-46|Example 16-46]]

**bc**

Bash can't handle floating point calculations, and it lacks operators for certain important mathematical functions. Fortunately, **bc** gallops to the rescue.

Not just a versatile, arbitrary precision calculation utility, **bc** offers many of the facilities of a programming language. It has a syntax vaguely resembling **C**.

Since it is a fairly well-behaved UNIX utility, and may therefore be used in a [[special-chars#^PIPEREF|pipe]], **bc** comes in handy in scripts.

Here is a simple template for using **bc** to calculate a script variable. This uses [[commandsub#^COMMANDSUBREF|command substitution]].

```bash
variable=$(echo "OPTIONS; OPERATIONS" | bc)
```

![[Example 16-47|Example 16-47]]

![[Example 16-48|Example 16-48]]

An alternate method of invoking **bc** involves using a [[here-documents#^HEREDOCREF|here document]] embedded within a [[commandsub#^COMMANDSUBREF|command substitution]] block. This is especially appropriate when a script needs to pass a list of options and commands to **bc**.

```bash
variable=`bc << LIMIT_STRING
options
statements
operations
LIMIT_STRING
`

...or...


variable=$(bc << LIMIT_STRING
options
statements
operations
LIMIT_STRING
)
```

![[Example 16-49|Example 16-49]]

![[Example 16-50|Example 16-50]]

See also [[Example A-37|Example A-37]].

**dc**

The **dc** (**d**esk **c**alculator) utility is [[internalvariables#^STACKDEFREF|stack-oriented]] and uses RPN (_Reverse Polish Notation_). Like **bc**, it has much of the power of a programming language.

Similar to the procedure with **bc**, [[internal#^ECHOREF|echo]] a command-string to **dc**.

```bash
echo "[Printing a string ... ]P" | dc
# The P command prints the string between the preceding brackets.

# And now for some simple arithmetic.
echo "7 8 * p" | dc     # 56
#  Pushes 7, then 8 onto the stack,
#+ multiplies ("*" operator), then prints the result ("p" operator).
```

Most persons avoid **dc**, because of its non-intuitive input and rather cryptic operators. Yet, it has its uses.

![[Example 16-51|Example 16-51]]

Studying the [[basic#^INFOREF|info]] page for **dc** is a painful path to understanding its intricacies. There seems to be a small, select group of _dc wizards_ who delight in showing off their mastery of this powerful, but arcane utility.

```bash
bash$ echo "16i[q]sa[ln0=aln100%Pln100/snlbx]sbA0D68736142snlbxq" | dc
Bash
```

```bash
dc <<< 10k5v1+2/p # 1.6180339887
#  ^^^            Feed operations to dc using a Here String.
#      ^^^        Pushes 10 and sets that as the precision (10k).
#         ^^      Pushes 5 and takes its square root
#                 (5v, v = square root).
#           ^^    Pushes 1 and adds it to the running total (1+).
#             ^^  Pushes 2 and divides the running total by that (2/).
#               ^ Pops and prints the result (p)
#  The result is  1.6180339887 ...
#  ... which happens to be the Pythagorean Golden Ratio, to 10 places.
```

![[Example 16-52|Example 16-52]]

**awk**

Yet another way of doing floating point math in a script is using [[awk#^AWKREF|awk's]] built-in math functions in a [[wrapper#^SHWRAPPER|shell wrapper]].

![[Example 16-53|Example 16-53]]
