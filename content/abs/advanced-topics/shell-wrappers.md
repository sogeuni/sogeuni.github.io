---
title: 36.2. Shell Wrappers
---

A _wrapper_ is a shell script that embeds a system command or utility, that accepts and passes a set of parameters to that command. [^1] Wrapping a script around a complex command-line simplifies invoking it. This is expecially useful with [[a-sed-and-awk-micro-primer#^SEDREF|sed]] and [[awk#^AWKREF|awk]].

A **sed** or **awk** script would normally be invoked from the command-line by a **sed -e _'commands'_** or **awk _'commands'_**. Embedding such a script in a Bash script permits calling it more simply, and makes it _reusable_. This also enables combining the functionality of _sed_ and _awk_, for example [[special-characters#^PIPEREF|piping]] the output of a set of _sed_ commands to _awk_. As a saved executable file, you can then repeatedly invoke it in its original form or modified, without the inconvenience of retyping it on the command-line.

![[Example 36-1|Example 36-1]]

![[Example 36-2|Example 36-2]]

**Example 36-3.** A generic *shell wrapper* that writes to a logfile

```bash
#!/bin/bash
#  logging-wrapper.sh
#  Generic shell wrapper that performs an operation
#+ and logs it.

DEFAULT_LOGFILE=logfile.txt

# Set the following two variables.
OPERATION=
#         Can be a complex chain of commands,
#+        for example an awk script or a pipe . . .

LOGFILE=
if [ -z "$LOGFILE" ]
then     # If not set, default to ...
  LOGFILE="$DEFAULT_LOGFILE"
fi

#         Command-line arguments, if any, for the operation.
OPTIONS="$@"


# Log it.
echo "`date` + `whoami` + $OPERATION "$@"" >> $LOGFILE
# Now, do it.
exec $OPERATION "$@"

# It's necessary to do the logging before the operation.
# Why?|

**Example 36-4.** A *shell wrapper* around an awk script

|   |
|---|
|#!/bin/bash
# pr-ascii.sh: Prints a table of ASCII characters.

START=33   # Range of printable ASCII characters (decimal).
END=127    # Will not work for unprintable characters (> 127).

echo " Decimal   Hex     Character"   # Header.
echo " -------   ---     ---------"

for ((i=START; i<=END; i++))
do
  echo $i | awk '{printf("  %3d       %2x         %c\n", $1, $1, $1)}'
# The Bash printf builtin will not work in this context:
#     printf "%c" "$i"
done

exit 0


#  Decimal   Hex     Character
#  -------   ---     ---------
#    33       21         !
#    34       22         "
#    35       23         #
#    36       24         $
#
#    . . .
#
#   122       7a         z
#   123       7b         {
#   124       7c         |
#   125       7d         }


#  Redirect the output of this script to a file
#+ or pipe it to "more":  sh pr-asc.sh | more
```

![[Example 36-5|Example 36-5]]

For those scripts needing a single do-it-all tool, a Swiss army knife, there is _Perl_. Perl combines the capabilities of [[a-sed-and-awk-micro-primer#^SEDREF|sed]] and [[awk#^AWKREF|awk]], and throws in a large subset of **C**, to boot. It is modular and contains support for everything ranging from object-oriented programming up to and including the kitchen sink. Short Perl scripts lend themselves to embedding within shell scripts, and there may be some substance to the claim that Perl can totally replace shell scripting (though the author of the _ABS Guide_ remains skeptical). ^PERLREF

![[Example 36-6|Example 36-6]]

It is even possible to combine a Bash script and Perl script within the same file. Depending on how the script is invoked, either the Bash part or the Perl part will execute.

![[Example 36-7|Example 36-7]]

```bash
bash$ bash bashandperl.sh
Greetings from the Bash part of the script.

bash$ perl -x bashandperl.sh
Greetings from the Perl part of the script.
```

It is, of course, possible to embed even more exotic scripting languages within shell wrappers. _Python_, for example ...

![[Example 36-8|Example 36-8]]

Wrapping a script around _mplayer_ and the Google's translation server, you can create something that talks back to you.

![[Example 36-9|Example 36-9]]

One interesting example of a complex shell wrapper is Martin Matusiak's [_undvd_ script](http://sourceforge.net/projects/undvd/), which provides an easy-to-use command-line interface to the complex [mencoder](http://www.mplayerhq.hu/DOCS/HTML/en/mencoder.html) utility. Another example is Itzchak Rehberg's [Ext3Undel](http://projects.izzysoft.de/trac/ext3undel), a set of scripts to recover deleted file on an _ext3_ filesystem.

[^1]: Quite a number of Linux utilities are, in fact, shell wrappers. Some examples are /usr/bin/pdf2ps, /usr/bin/batch, and /usr/bin/xmkmf.
