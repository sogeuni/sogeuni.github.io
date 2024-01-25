---
title: 5. Quoting
---

Quoting means just that, bracketing a string in quotes. This has the effect of protecting [[special-characters|special characters]] in the string from reinterpretation or expansion by the shell or shell script. (A character is "special" if it has an interpretation other than its literal meaning. For example, the [[special-characters#^ASTERISKREF|asterisk *]] represents a *wild card* character in [[globbing|globbing]] and [[regexp#^REGEXREF|Regular Expressions]]).

```bash
bash$ ls -l [Vv]*
-rw-rw-r--    1 bozo  bozo       324 Apr  2 15:05 VIEWDATA.BAT
 -rw-rw-r--    1 bozo  bozo       507 May  4 14:25 vartrace.sh
 -rw-rw-r--    1 bozo  bozo       539 Apr 14 17:11 viewdata.sh

bash$ ls -l '[Vv]*'
ls: [Vv]*: No such file or directory
```

> In everyday speech or writing, when we "quote" a phrase, we set it apart and give it special meaning. In a Bash script, when we *quote* a string, we set it apart and protect its *literal* meaning.|

Certain programs and utilities reinterpret or expand special characters in a quoted string. An important use of quoting is protecting a command-line parameter from the shell, but still letting the calling program expand it.

```bash
bash$ grep '[Ff]irst' *.txt
file1.txt:This is the first line of file1.txt.
 file2.txt:This is the First line of file2.txt.
```

Note that the unquoted **grep \[Ff]irst \*.txt** works under the Bash shell. [^1]

Quoting can also suppress [[internal-commands-and-builtins#^ECHOREF|echo's]] "appetite" for newlines.

```bash
bash$ echo $(ls -l)
total 8 -rw-rw-r-- 1 bo bo 13 Aug 21 12:57 t.sh -rw-rw-r-- 1 bo bo 78 Aug 21 12:57 u.sh


bash$ echo "$(ls -l)"
total 8
 -rw-rw-r--  1 bo bo  13 Aug 21 12:57 t.sh
 -rw-rw-r--  1 bo bo  78 Aug 21 12:57 u.sh
```

## Quoting Variables

When referencing a variable, it is generally advisable to enclose its name in double quotes. This prevents reinterpretation of all special characters within the quoted string -- except `$`, ` `` (backquote), and `\`(escape). [^2] Keeping $ as a special character within double quotes permits referencing a quoted variable (*"$variable"*), that is, replacing the variable with its value (see [[example 4-1|Example 4-1]], above).

Use double quotes to prevent word splitting. [^3] An argument enclosed in double quotes presents itself as a single word, even if it contains [[special-characters#Whitespace|whitespace]] separators.

```bash
List="one two three"

for a in $List     # Splits the variable in parts at whitespace.
do
  echo "$a"
done
# one
# two
# three

echo "---"

for a in "$List"   # Preserves whitespace in a single variable.
do #     ^     ^
  echo "$a"
done
# one two three
```

A more elaborate example:

```bash
variable1="a variable containing five words"
COMMAND This is $variable1    # Executes COMMAND with 7 arguments:
# "This" "is" "a" "variable" "containing" "five" "words"

COMMAND "This is $variable1"  # Executes COMMAND with 1 argument:
# "This is a variable containing five words"


variable2=""    # Empty.

COMMAND $variable2 $variable2 $variable2
                # Executes COMMAND with no arguments. 
COMMAND "$variable2" "$variable2" "$variable2"
                # Executes COMMAND with 3 empty arguments. 
COMMAND "$variable2 $variable2 $variable2"
                # Executes COMMAND with 1 argument (2 spaces). 

# Thanks, Stéphane Chazelas.
```

> [!tip] Enclosing the arguments to an **echo** statement in double quotes is necessary only when word splitting or preservation of [[special-characters#Whitespace|whitespace]] is an issue.

![[example 5-1]]

Single quotes (' ') operate similarly to double quotes, but do not permit referencing variables, since the special meaning of $ is turned off. Within single quotes, *every* special character except ' gets interpreted literally. Consider single quotes ("full quoting") to be a stricter method of quoting than double quotes ("partial quoting").

> [!note]
> Since even the escape character (\) gets a literal interpretation within single quotes, trying to enclose a single quote within single quotes will not yield the expected result.
>
> ```bash
> echo "Why can't I write 's between single quotes"
> 
> echo
> 
> # The roundabout method.
> echo 'Why can'\''t I write '"'"'s between single quotes'
> #    |-------|  |----------|   |-----------------------|
> # Three single-quoted strings, with escaped and quoted single quotes between.
> 
> # This example courtesy of Stéphane Chazelas.
> ```

## Escaping

*Escaping* is a method of quoting single characters. The escape (\) preceding a character tells the shell to interpret that character literally.

> [!caution] With certain commands and utilities, such as [[internal-commands-and-builtins#^ECHOREF|echo]] and [[sedawk#^SEDREF|sed]], escaping a character may have the opposite effect - it can toggle on a special meaning for that character.

**Special meanings of certain escaped characters**

used with **echo** and **sed**

\n

means newline

\r

means return

\t

means tab

\v

means vertical tab

\b

means backspace

\a

means *alert* (beep or flash)

\0xx

translates to the octal [[special-characters#^ASCIIDEF|ASCII]] equivalent of *0nn*, where *nn* is a string of digits

> [!important] The `$' ... '` [[Chapter%205.%20Quoting.md#^QUOTINGREF|quoted]] string-expansion construct is a mechanism that uses escaped octal or hex values to assign ASCII characters to variables, e.g., `quote=$'\042'`.

![[example 5-2]]

A more elaborate example:

![[example 5-3]]

See also [[bash-version-2#^EX77|Example 37-1]].

\"

gives the quote its literal meaning

```bash
echo "Hello"                     # Hello
echo "\"Hello\" ... he said."    # "Hello" ... he said.
```

\$

gives the dollar sign its literal meaning (variable name following \$ will not be referenced)

```bash
echo "\$variable01"           # $variable01
echo "The book cost \$7.98."  # The book cost $7.98.
```

\\

gives the backslash its literal meaning

```bash
echo "\\"  # Results in \

# Whereas . . .

echo "\"   # Invokes secondary prompt from the command-line.
           # In a script, gives an error message.

# However . . .

echo '\'   # Results in \
```

> [!note] 
> The behavior of \ depends on whether it is escaped, [[varsubn#^SNGLQUO|strong-quoted]], [[varsubn#^DBLQUO|weak-quoted]], or appearing within [[command-substitution#^COMMANDSUBREF|command substitution]] or a [[here-documents#^HEREDOCREF|here document]].
>
> ```bash
>                       #  Simple escaping and quoting
> echo \z               #  z
> echo \\z              # \z
> echo '\z'             # \z
> echo '\\z'            # \\z
> echo "\z"             # \z
> echo "\\z"            # \z
> 
>                       #  Command substitution
> echo `echo \z`        #  z
> echo `echo \\z`       #  z
> echo `echo \\\z`      # \z
> echo `echo \\\\z`     # \z
> echo `echo \\\\\\z`   # \z
> echo `echo \\\\\\\z`  # \\z
> echo `echo "\z"`      # \z
> echo `echo "\\z"`     # \z
> 
>                       # Here document
> cat <<EOF              
> \z                      
> EOF                   # \z
> 
> cat <<EOF              
> \\z                     
> EOF                   # \z
> 
> # These examples supplied by Stéphane Chazelas.
> ```
>
> Elements of a string assigned to a variable may be escaped, but the escape character alone may not be assigned to a variable.
> 
> ```bash
> variable=\
> echo "$variable"
> # Will not work - gives an error message:
> # test.sh: : command not found
> # A "naked" escape cannot safely be assigned to a variable.
> #
> #  What actually happens here is that the "\" escapes the newline and
> #+ the effect is        variable=echo "$variable"
> #+                      invalid variable assignment
> 
> variable=\
> 23skidoo
> echo "$variable"        #  23skidoo
>                         #  This works, since the second line
>                         #+ is a valid variable assignment.
> 
> variable=\ 
> #        \^    escape followed by space
> echo "$variable"        # space
> 
> variable=\\
> echo "$variable"        # \
> 
> variable=\\\
> echo "$variable"
> # Will not work - gives an error message:
> # test.sh: \: command not found
> #
> #  First escape escapes second one, but the third one is left "naked",
> #+ with same result as first instance, above.
> 
> variable=\\\\
> echo "$variable"        # \\
>                         # Second and fourth escapes escaped.
>                         # This is o.k.
> ```

Escaping a space can prevent word splitting in a command's argument list.

```bash
file_list="/bin/cat /bin/gzip /bin/more /usr/bin/less /usr/bin/emacs-20.7"
# List of files as argument(s) to a command.

# Add two files to the list, and list all.
ls -l /usr/X11R6/bin/xsetroot /sbin/dump $file_list

echo "-------------------------------------------------------------------------"

# What happens if we escape a couple of spaces?
ls -l /usr/X11R6/bin/xsetroot\ /sbin/dump\ $file_list
# Error: the first three files concatenated into a single argument to 'ls -l'
#        because the two escaped spaces prevent argument (word) splitting.
```

The escape also provides a means of writing a multi-line command. Normally, each separate line constitutes a different command, but an escape at the end of a line *escapes the newline character*, and the command sequence continues on to the next line.

```bash
(cd /source/directory && tar cf - . ) | \
(cd /dest/directory && tar xpvf -)
# Repeating Alan Cox's directory tree copy command,
# but split into two lines for increased legibility.

# As an alternative:
tar cf - -C /source/directory . |
tar xpvf - -C /dest/directory
# See note below.
# (Thanks, Stéphane Chazelas.)
```

> [!note] If a script line ends with a \|, a pipe character, then a \, an escape, is not strictly necessary. It is, however, good programming practice to always escape the end of a line of code that continues to the following line.

```bash
echo "foo
bar" 
#foo
#bar

echo

echo 'foo
bar'    # No difference yet.
#foo
#bar

echo

echo foo\
bar     # Newline escaped.
#foobar

echo

echo "foo\
bar"     # Same here, as \ still interpreted as escape within weak quotes.
#foobar

echo

echo 'foo\
bar'     # Escape character \ taken literally because of strong quoting.
#foo\
#bar

# Examples suggested by Stéphane Chazelas.
```

[^1]: Unless there is a file named first in the current working directory. Yet another reason to *quote*. (Thank you, Harald Koenig, for pointing this out.)

[^2]: Encapsulating "!" within double quotes gives an error when used *from the command line*. This is interpreted as a [[history-commands|history command]]. Within a script, though, this problem does not occur, since the Bash history mechanism is disabled then.
    
    Of more concern is the *apparently* inconsistent behavior of *\* within double quotes, and especially following an **echo -e** command.
    
    ```bash
    bash$ echo hello\!
    hello!
    bash$ echo "hello\!"
    hello\!
    
    
    bash$ echo \
    >
    bash$ echo "\"
    >
    bash$ echo \a
    a
    bash$ echo "\a"
    \a
    
    
    bash$ echo x\ty
    xty
    bash$ echo "x\ty"
    x\ty
    
    bash$ echo -e x\ty
    xty
    bash$ echo -e "x\ty"
    x       y
    
    ```
    
    Double quotes following an *echo* *sometimes* escape *\*. Moreover, the -e option to *echo* causes the "\t" to be interpreted as a *tab*.
    
    (Thank you, Wayne Pollock, for pointing this out, and Geoff Lee and Daniel Barclay for explaining it.)

[^3]: "Word splitting," in this context, means dividing a character string into separate and discrete arguments.
