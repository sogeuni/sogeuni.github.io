---
title: 36.5. "Colorizing" Scripts
---


The ANSI [^1] escape sequences set screen attributes, such as bold text, and color of foreground and background. [[converting-dos-batch-files-to-shell-scripts#^DOSBATCH1|DOS batch files]] commonly used ANSI escape codes for _color_ output, and so can Bash scripts.

![[Example 36-13|Example 36-13]]

![[Example 36-14|Example 36-14]]

The simplest, and perhaps most useful ANSI escape sequence is bold text, **\033[1m ... \033[0m**. The \033 represents an [[quoting#^ESCP|escape]], the "[1" turns on the bold attribute, while the "[0" switches it off. The "m" terminates each term of the escape sequence.

```bash
bash$ echo -e "\033[1mThis is bold text.\033[0m"
```

A similar escape sequence switches on the underline attribute (on an _rxvt_ and an _aterm_).

```bash
bash$ echo -e "\033[4mThis is underlined text.\033[0m"
```

> [!note]
> With an **echo**, the -e option enables the escape sequences.

Other escape sequences change the text and/or background color.

```bash
bash$ echo -e '\E[34;47mThis prints in blue.'; tput sgr0


bash$ echo -e '\E[33;44m'"yellow text on blue background"; tput sgr0


bash$ echo -e '\E[1;33;44m'"BOLD yellow text on blue background"; tput sgr0
	      

```

> [!note]
> It's usually advisable to set the _bold_ attribute for light-colored foreground text.

The **tput sgr0** restores the terminal settings to normal. Omitting this lets all subsequent output from that particular terminal remain blue.

> [!note]
> Since **tput sgr0** fails to restore terminal settings under certain circumstances, **echo -ne \E[0m** may be a better choice.

> Use the following template for writing colored text on a colored background.
>
> **echo -e '\E[COLOR1;COLOR2mSome text goes here.'**
>
>The "\E[" begins the escape sequence. The semicolon-separated numbers "COLOR1" and "COLOR2" specify a foreground and a background color, according to the table below. (The order of the numbers does not matter, since the foreground and background numbers fall in non-overlapping ranges.) The "m" terminates the escape sequence, and the text begins immediately after that.
>
> Note also that [[varsubn#^SNGLQUO|single quotes]] enclose the remainder of the command sequence following the **echo -e**.

The numbers in the following table work for an _rxvt_ terminal. Results may vary for other terminal emulators.

**Table 36-1. Numbers representing colors in Escape Sequences**

|Color|Foreground|Background|
|:--|:--|:--|
|black|30|40|
|red|31|41|
|green|32|42|
|yellow|33|43|
|blue|34|44|
|magenta|35|45|
|cyan|36|46|
|white|37|47|

![[Example 36-15|Example 36-15]]

See also [[Example A-21|Example A-21]], [[Example A-44|Example A-44]], [[Example A-52|Example A-52]], and [[Example A-40|Example A-40]].

> [!caution]
> There is, however, a major problem with all this. _ANSI escape sequences are emphatically [[portability-issues|non-portable]]._ What works fine on some terminal emulators (or the console) may work differently, or not at all, on others. A "colorized" script that looks stunning on the script author's machine may produce unreadable output on someone else's. This somewhat compromises the usefulness of colorizing scripts, and possibly relegates this technique to the status of a gimmick. Colorized scripts are probably inappropriate in a commercial setting, i.e., your supervisor might disapprove.

Alister's [ansi-color](http://code.google.com/p/ansi-color/) utility (based on [Moshe Jacobson's color utility](http://bash.deta.in/color-1.1.tar.gz) considerably simplifies using ANSI escape sequences. It substitutes a clean and logical syntax for the clumsy constructs just discussed.

Henry/teikedvl has likewise created a utility ([http://scriptechocolor.sourceforge.net/](http://scriptechocolor.sourceforge.net/)) to simplify creation of colorized scripts.

[^1]: ANSI is, of course, the acronym for the American National Standards Institute. This august body establishes and maintains various technical and industrial standards.
