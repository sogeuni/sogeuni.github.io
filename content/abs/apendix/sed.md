---
title: C.1. Sed
---


_Sed_ is a non-interactive [^1] **s**tream **ed**itor. It receives text input, whether from stdin or from a file, performs certain operations on specified lines of the input, one line at a time, then outputs the result to stdout or to a file. Within a shell script, _sed_ is usually one of several tool components in a pipe.

_Sed_ determines which lines of its input that it will operate on from the _address range_ passed to it. [^2] Specify this address range either by line number or by a pattern to match. For example, _3d_ signals _sed_ to delete line 3 of the input, and _/Windows/d_ tells sed that you want every line of the input containing a match to "Windows" deleted.

Of all the operations in the _sed_ toolkit, we will focus primarily on the three most commonly used ones. These are **p**rinting (to stdout), **d**eletion, and **s**ubstitution.

**Table C-1. Basic sed operators**

| Operator | Name | Effect |
| :--- | :--- | :--- |
| [address-range]/p | print | Print [specified address range] |
| [address-range]/d | delete | Delete [specified address range] |
| s/pattern1/pattern2/ | substitute | Substitute pattern2 for first instance of pattern1 in a line |
| [address-range]/s/pattern1/pattern2/ | substitute | Substitute pattern2 for first instance of pattern1 in a line, over _address-range_ |
| [address-range]/y/pattern1/pattern2/ | transform | replace any character in pattern1 with the corresponding character in pattern2, over _address-range_ (equivalent of **tr**) |
| [address] i pattern Filename | insert | Insert pattern at address indicated in file Filename. Usually used with -i _in-place_ option. |
| g | global | Operate on _every_ pattern match within each matched line of input |
|  |  |  |

> [!note]
> Unless the g (_global_) operator is appended to a _substitute_ command, the substitution operates only on the _first_ instance of a pattern match within each line.

From the command-line and in a shell script, a sed operation may require quoting and certain options.

```bash
sed -e '/^$/d' $filename
# The -e option causes the next string to be interpreted as an editing instruction.
#  (If passing only a single instruction to sed, the "-e" is optional.)
#  The "strong" quotes ('') protect the RE characters in the instruction
#+ from reinterpretation as special characters by the body of the script.
# (This reserves RE expansion of the instruction for sed.)
#
# Operates on the text contained in file $filename.
```

In certain cases, a _sed_ editing command will not work with single quotes.

```bash
filename=file1.txt
pattern=BEGIN

  sed "/^$pattern/d" "$filename"  # Works as specified.
# sed '/^$pattern/d' "$filename"    has unexpected results.
#        In this instance, with strong quoting (' ... '),
#+      "$pattern" will not expand to "BEGIN".
```

> [!note]
> _Sed_ uses the -e option to specify that the following string is an instruction or set of instructions. If there is only a single instruction contained in the string, then this may be omitted.

```bash
sed -n '/xzy/p' $filename
# The -n option tells sed to print only those lines matching the pattern.
# Otherwise all input lines would print.
# The -e option not necessary here since there is only a single editing instruction.
```

**Table C-2. Examples of sed operators**

|Notation|Effect|
|:--|:--|
|8d|Delete 8th line of input.|
|/^$/d|Delete all blank lines.|
|1,/^$/d|Delete from beginning of input up to, and including first blank line.|
|/Jones/p|Print only lines containing "Jones" (with -n option).|
|s/Windows/Linux/|Substitute "Linux" for first instance of "Windows" found in each input line.|
|s/BSOD/stability/g|Substitute "stability" for every instance of "BSOD" found in each input line.|
|s/ *$//|Delete all spaces at the end of every line.|
|s/00*/0/g|Compress all consecutive sequences of zeroes into a single zero.|
|echo "Working on it." \| sed -e '1i How far are you along?'|Prints "How far are you along?" as first line, "Working on it" as second.|
|5i 'Linux is great.' file.txt|Inserts 'Linux is great.' at line 5 of the file file.txt.|
|/GUI/d|Delete all lines containing "GUI".|
|s/GUI//g|Delete all instances of "GUI", leaving the remainder of each line intact.|

Substituting a zero-length string for another is equivalent to deleting that string within a line of input. This leaves the remainder of the line intact. Applying **s/GUI//** to the line

```bash
The most important parts of any application are its GUI and sound effects
```

results in

```bash
The most important parts of any application are its  and sound effects
```

A backslash forces the **sed** replacement command to continue on to the next line. This has the effect of using the _newline_ at the end of the first line as the _replacement string_.

```bash
s/^  */\
/g
```

This substitution replaces line-beginning spaces with a newline. The net result is to replace paragraph indents with a blank line between paragraphs.

An address range followed by one or more operations may require open and closed curly brackets, with appropriate newlines.

```bash
/[0-9A-Za-z]/,/^$/{
/^$/d
}
```

This deletes only the first of each set of consecutive blank lines. That might be useful for single-spacing a text file, but retaining the blank line(s) between paragraphs.

> [!note]
> The usual delimiter that _sed_ uses is /. However, _sed_ allows other delimiters, such as %. This is useful when / is part of a replacement string, as in a file pathname. See [[../beyond-the-basic/loops#^FINDSTRING|Example 11-10]] and [[../commands/file-and-archiving-commands#^STRIPC|Example 16-32]].

> [!tip]
> A quick way to double-space a text file is **sed G filename**.

For illustrative examples of sed within shell scripts, see:

1. [[../advanced-topics/shell-wrappers#^EX3|Example 36-1]]
2. [[../advanced-topics/shell-wrappers#^EX4|Example 36-2]]
3. [[../commands/complex-commands#^EX57|Example 16-3]]
4. [[./contributed-scripts#^RN|Example A-2]]
5. [[../commands/text-processing-commands#^GRP|Example 16-17]]
6. [[../commands/text-processing-commands#^COL|Example 16-27]]
7. [[./contributed-scripts#^BEHEAD|Example A-12]]
8. [[./contributed-scripts#^TREE|Example A-16]]
9. [[./contributed-scripts#^TREE2|Example A-17]]
10. [[../commands/file-and-archiving-commands#^STRIPC|Example 16-32]]
11. [[../beyond-the-basic/loops#^FINDSTRING|Example 11-10]]
12. [[../commands/math-commands#^BASE|Example 16-48]]
13. [[./contributed-scripts#^MAILFORMAT|Example A-1]]
14. [[../commands/text-processing-commands#^RND|Example 16-14]]
15. [[../commands/text-processing-commands#^WF|Example 16-12]]
16. [[./contributed-scripts#^LIFESLOW|Example A-10]]
17. [[../advanced-topics/here-documents#^SELFDOCUMENT|Example 19-12]]
18. [[../commands/text-processing-commands#^DICTLOOKUP|Example 16-19]]
19. [[./contributed-scripts#^WHX|Example A-29]]
20. [[./contributed-scripts#^BASHPODDER|Example A-31]]
21. [[./contributed-scripts#^TOHTML|Example A-24]]
22. [[./contributed-scripts#^STOPWATCH|Example A-43]]
23. [[./contributed-scripts#^SEDAPPEND|Example A-55]]

For a more extensive treatment of _sed_, refer to the [[../bibliography#^DGSEDREF|pertinent references]] in the [[../bibliography|_Bibliography_]].

[^1]: _Sed_ executes without user intervention.

[^2]: If no address range is specified, the default is _all_ lines.
