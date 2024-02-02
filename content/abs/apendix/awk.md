---
title: C.2. Awk
---


_Awk_ [^1] is a full-featured text processing language with a syntax reminiscent of _C_. While it possesses an extensive set of operators and capabilities, we will cover only a few of these here - the ones most useful in shell scripts.

Awk breaks each line of input passed to it into [[special-characters#^FIELDREF|fields]]. By default, a field is a string of consecutive characters delimited by [[special-characters#Whitespace|whitespace]], though there are options for changing this. Awk parses and operates on each separate field. This makes it ideal for handling structured text files -- especially tables -- data organized into consistent chunks, such as rows and columns.

[[varsubn#^SNGLQUO|Strong quoting]] and [[special-characters#^CODEBLOCKREF|curly brackets]] enclose blocks of awk code within a shell script.

```bash
# $1 is field #1, $2 is field #2, etc.

echo one two | awk '{print $1}'
# one

echo one two | awk '{print $2}'
# two

# But what is field #0 ($0)?
echo one two | awk '{print $0}'
# one two
# All the fields!


awk '{print $3}' $filename
# Prints field #3 of file $filename to stdout.

awk '{print $1 $5 $6}' $filename
# Prints fields #1, #5, and #6 of file $filename.

awk '{print $0}' $filename
# Prints the entire file!
# Same effect as:   cat $filename . . . or . . . sed '' $filename
```

We have just seen the awk _print_ command in action. The only other feature of awk we need to deal with here is variables. Awk handles variables similarly to shell scripts, though a bit more flexibly.

```bash
{ total += ${column_number} }
```

This adds the value of _column_number_ to the running total of _total_>. Finally, to print "total", there is an **END** command block, executed after the script has processed all its input.

```bash
END { print total }
```

Corresponding to the **END**, there is a **BEGIN**, for a code block to be performed before awk starts processing its input.

The following example illustrates how **awk** can add text-parsing tools to a shell script.

![[Example C-1|Example C-1]]

For simpler examples of awk within shell scripts, see:

1. [[Example 15-14|Example 15-14]]
2. [[Example 20-8|Example 20-8]]
3. [[Example 16-32|Example 16-32]]
4. [[Example 36-5|Example 36-5]]
5. [[Example 28-2|Example 28-2]]
6. [[Example 15-20|Example 15-20]]
7. [[Example 29-3|Example 29-3]]
8. [[Example 29-4|Example 29-4]]
9. [[Example 11-3|Example 11-3]]
10. [[Example 16-61|Example 16-61]]
11. [[Example 9-16|Example 9-16]]
12. [[Example 16-4|Example 16-4]]
13. [[Example 10-6|Example 10-6]]
14. [[Example 36-19|Example 36-19]]
15. [[Example 11-9|Example 11-9]]
16. [[Example 36-4|Example 36-4]]
17. [[Example 16-53|Example 16-53]]
18. [[Example T-3|Example T-3]]

That's all the awk we'll cover here, folks, but there's lots more to learn. See the appropriate references in the [[bibliography|_Bibliography_]].

[^1]: Its name derives from the initials of its authors, **A**ho, **W**einberg, and **K**ernighan.
