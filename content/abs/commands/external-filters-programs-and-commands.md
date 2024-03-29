---
title: 16. External Filters, Programs and Commands
---


Standard UNIX commands make shell scripts more versatile. The power of scripts comes from coupling system commands and shell directives with simple programming constructs.

16.5. File and Archiving Commands
16.6. Communications Commands
16.7. Terminal Control Commands
16.8. Math Commands
16.9. Miscellaneous Commands

## Basic Commands

**The first commands a novice learns**

## ls

The basic file "list" command. It is all too easy to underestimate the power of this humble command. For example, using the -R, recursive option, **ls** provides a tree-like listing of a directory structure. Other useful options are -S, sort listing by file size, -t, sort by file modification time, -v, sort by (numerical) version numbers embedded in the filenames, [^1] -b, show escape characters, and -i, show file inodes (see [[Example 16-4|Example 16-4]]).

```bash
bash$ ls -l
-rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter10.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter11.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter12.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter1.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter2.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter3.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:49 Chapter_headings.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:49 Preface.txt


bash$ ls -lv
 total 0
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:49 Chapter_headings.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:49 Preface.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter1.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter2.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter3.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter10.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter11.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter12.txt
```

> [!tip]
> The _ls_ command returns a non-zero [[exit-and-exit-status#^EXITSTATUSREF|exit status]] when attempting to list a non-existent file.
>
> ```bash
> bash$ ls abc
> ls: abc: No such file or directory
> 
> 
> bash$ echo $?
> 2
> ```

![[Example 16-1|Example 16-1]]

## cat, tac

**cat**, an acronym for _concatenate_, lists a file to stdout. When combined with redirection (> or >>), it is commonly used to concatenate files.

```bash
# Uses of 'cat'
cat filename                          # Lists the file.

cat file.1 file.2 file.3 > file.123   # Combines three files into one.
```

The -n option to **cat** inserts consecutive numbers before all lines of the target file(s). The -b option numbers only the non-blank lines. The -v option echoes nonprintable characters, using ^ notation. The -s option squeezes multiple consecutive blank lines into a single blank line.

See also [[Example 16-28|Example 16-28]] and [[Example 16-24|Example 16-24]].

> [!note]
> In a [[special-characters#^PIPEREF|pipe]], it may be more efficient to [[io-redirection|redirect]] the stdin to a file, rather than to **cat** the file.
>
> ```bash
> cat filename | tr a-z A-Z
> 
> tr a-z A-Z < filename   #  Same effect, but starts one less process,
>                         #+ and also dispenses with the pipe.
> ```

**tac**, is the inverse of _cat_, listing a file backwards from its end.

## rev

reverses each line of a file, and outputs to stdout. This does not have the same effect as **tac**, as it preserves the order of the lines, but flips each one around (mirror image).

```bash
bash$ cat file1.txt
This is line 1.
 This is line 2.


bash$ tac file1.txt
This is line 2.
 This is line 1.


bash$ rev file1.txt
.1 enil si sihT
 .2 enil si sihT
	      
```

## cp

This is the file copy command. **cp file1 file2** copies file1 to file2, overwriting file2 if it already exists (see [[Example 16-6|Example 16-6]]).

> [!tip]
> Particularly useful are the -a archive flag (for copying an entire directory tree), the -u update flag (which prevents overwriting identically-named newer files), and the -r and -R recursive flags.
>
> ```bash
> cp -u source_dir/* dest_dir
> #  "Synchronize" dest_dir to source_dir
> #+  by copying over all newer and not previously existing files.
> ```

## mv

This is the file _move_ command. It is equivalent to a combination of **cp** and **rm**. It may be used to move multiple files to a directory, or even to rename a directory. For some examples of using **mv** in a script, see [[Example 10-11|Example 10-11]] and [[Example A-2|Example A-2]].

> [!note]
> When used in a non-interactive script, **mv** takes the -f (_force_) option to bypass user input.
>
> When a directory is moved to a preexisting directory, it becomes a subdirectory of the destination directory.
>
> ```bash
> bash$ mv source_directory target_directory
> 
> bash$ ls -lF target_directory
> total 1
>  drwxrwxr-x    2 bozo  bozo      1024 May 28 19:20 source_directory/
> 	      
> ```

## rm

Delete (remove) a file or files. The -f option forces removal of even readonly files, and is useful for bypassing user input in a script.

> [!note]
> The _rm_ command will, by itself, fail to remove filenames beginning with a dash. Why? Because _rm_ sees a dash-prefixed filename as an _option_.
>
> ```bash
> bash$ rm -badname
> rm: invalid option -- b
>  Try `rm --help' for more information.
> ```
>
> One clever workaround is to precede the filename with a " -- " (the _end-of-options_ flag).
>
> ```bash
> bash$ rm -- -badname
> ```
>
> Another method to is to preface the filename to be removed with a dot-slash .
>
> ```bash
> bash$ rm ./-badname
> ```

> [!warning]
> When used with the recursive flag -r, this command removes files all the way down the directory tree from the current directory. A careless **rm -rf *** can wipe out a big chunk of a directory structure.

## rmdir

Remove directory. The directory must be empty of all files -- including "invisible" _dotfiles_ [^2] -- for this command to succeed.

## mkdir

Make directory, creates a new directory. For example, **mkdir -p project/programs/December** creates the named directory. The _-p_ option automatically creates any necessary parent directories.

## chmod

Changes the attributes of an existing file or directory (see [[Example 15-14|Example 15-14]]).

```bash
chmod +x filename
# Makes "filename" executable for all users.

chmod u+s filename
# Sets "suid" bit on "filename" permissions.
# An ordinary user may execute "filename" with same privileges as the file's owner.
# (This does not apply to shell scripts.)
```

```bash
chmod 644 filename
#  Makes "filename" readable/writable to owner, readable to others
#+ (octal mode).

chmod 444 filename
#  Makes "filename" read-only for all.
#  Modifying the file (for example, with a text editor)
#+ not allowed for a user who does not own the file (except for root),
#+ and even the file owner must force a file-save
#+ if she modifies the file.
#  Same restrictions apply for deleting the file.
```

```bash
chmod 1777 directory-name
#  Gives everyone read, write, and execute permission in directory,
#+ however also sets the "sticky bit".
#  This means that only the owner of the directory,
#+ owner of the file, and, of course, root
#+ can delete any particular file in that directory.

chmod 111 directory-name
#  Gives everyone execute-only permission in a directory.
#  This means that you can execute and READ the files in that directory
#+ (execute permission necessarily includes read permission
#+ because you can't execute a file without being able to read it).
#  But you can't list the files or search for them with the "find" command.
#  These restrictions do not apply to root.

chmod 000 directory-name
#  No permissions at all for that directory.
#  Can't read, write, or execute files in it.
#  Can't even list files in it or "cd" to it.
#  But, you can rename (mv) the directory
#+ or delete it (rmdir) if it is empty.
#  You can even symlink to files in the directory,
#+ but you can't read, write, or execute the symlinks.
#  These restrictions do not apply to root.
```

## chattr

**Ch**ange file **attr**ibutes. This is analogous to **chmod** above, but with different options and a different invocation syntax, and it works only on _ext2/ext3_ filesystems.

One particularly interesting **chattr** option is i. A **chattr +i filename** marks the file as immutable. The file cannot be modified, linked to, or deleted, _not even by root_. This file attribute can be set or removed only by _root_. In a similar fashion, the a option marks the file as append only.

```bash
root# chattr +i file1.txt


root# rm file1.txt

rm: remove write-protected regular file `file1.txt'? y
 rm: cannot remove `file1.txt': Operation not permitted
	      
```

If a file has the s (secure) attribute set, then when it is deleted its block is overwritten with binary zeroes. [^3]

If a file has the u (undelete) attribute set, then when it is deleted, its contents can still be retrieved (undeleted).

If a file has the c (compress) attribute set, then it will automatically be compressed on writes to disk, and uncompressed on reads.

> [!note]
> The file attributes set with **chattr** do not show in a file listing (**ls -l**).

## ln

Creates links to pre-existings files. A "link" is a reference to a file, an alternate name for it. The **ln** command permits referencing the linked file by more than one name and is a superior alternative to aliasing (see [[Example 4-6|Example 4-6]]).

The **ln** creates only a reference, a pointer to the file only a few bytes in size.

The **ln** command is most often used with the -s, symbolic or "soft" link flag. Advantages of using the -s flag are that it permits linking across file systems or to directories.

The syntax of the command is a bit tricky. For example: **ln -s oldfile newfile** links the previously existing oldfile to the newly created link, newfile.

> [!caution]
> If a file named newfile has previously existed, an error message will result.

> **Which type of link to use?**
>
> As John Macdonald explains it:
>
> Both of these [types of links] provide a certain measure of dual reference -- if you edit the contents of the file using any name, your changes will affect both the original name and either a hard or soft new name. The differences between them occurs when you work at a higher level. The advantage of a hard link is that the new name is totally independent of the old name -- if you remove or rename the old name, that does not affect the hard link, which continues to point to the data while it would leave a soft link hanging pointing to the old name which is no longer there. The advantage of a soft link is that it can refer to a different file system (since it is just a reference to a file name, not to actual data). And, unlike a hard link, a symbolic link can refer to a directory.

Links give the ability to invoke a script (or any other type of executable) with multiple names, and having that script behave according to how it was invoked.

![[Example 16-2|Example 16-2]]

## man, info

These commands access the manual and information pages on system commands and installed utilities. When available, the _info_ pages usually contain more detailed descriptions than do the _man_ pages.

There have been various attempts at "automating" the writing of _man pages_. For a script that makes a tentative first step in that direction, see [[Example A-39|Example A-39]].

## Complex Commands

**Commands for more advanced users**

**find**

-exec _COMMAND_ \;

Carries out _COMMAND_ on each file that **find** matches. The command sequence terminates with ; (the ";" is [[escapingsection#^ESCP|escaped]] to make certain the shell passes it to **find** literally, without interpreting it as a special character).

```bash
bash$ find ~/ -name '*.txt'
/home/bozo/.kde/share/apps/karm/karmdata.txt
 /home/bozo/misc/irmeyc.txt
 /home/bozo/test-scripts/1.txt
	      
```

If _COMMAND_ contains {}, then **find** substitutes the full path name of the selected file for "{}".

```bash
find ~/ -name 'core*' -exec rm {} \;
# Removes all core dump files from user's home directory.
```

```bash
find /home/bozo/projects -mtime -1
#                               ^   Note minus sign!
#  Lists all files in /home/bozo/projects directory tree
#+ that were modified within the last day (current_day - 1).
#
find /home/bozo/projects -mtime 1
#  Same as above, but modified *exactly* one day ago.
#
#  mtime = last modification time of the target file
#  ctime = last status change time (via 'chmod' or otherwise)
#  atime = last access time

DIR=/home/bozo/junk_files
find "$DIR" -type f -atime +5 -exec rm {} \;
#                          ^           ^^
#  Curly brackets are placeholder for the path name output by "find."
#
#  Deletes all files in "/home/bozo/junk_files"
#+ that have not been accessed in *at least* 5 days (plus sign ... +5).
#
#  "-type filetype", where
#  f = regular file
#  d = directory
#  l = symbolic link, etc.
#
#  (The 'find' manpage and info page have complete option listings.)
```

```bash
find /etc -exec grep '[0-9][0-9]*[.][0-9][0-9]*[.][0-9][0-9]*[.][0-9][0-9]*' {} \;

# Finds all IP addresses (xxx.xxx.xxx.xxx) in /etc directory files.
# There a few extraneous hits. Can they be filtered out?

# Possibly by:

find /etc -type f -exec cat '{}' \; | tr -c '.[:digit:]' '\n' \
| grep '^[^.][^.]*\.[^.][^.]*\.[^.][^.]*\.[^.][^.]*$'
#
#  [:digit:] is one of the character classes
#+ introduced with the POSIX 1003.2 standard. 

# Thanks, Stéphane Chazelas. 
```

> [!note]
> The -exec option to **find** should not be confused with the [[internal#^EXECREF|exec]] shell builtin.

![[Example 16-3|Example 16-3]]

![[Example 16-4|Example 16-4]]

The **find** command also works without the -exec option.

```bash
#!/bin/bash
#  Find suid root files.
#  A strange suid file might indicate a security hole,
#+ or even a system intrusion.

directory="/usr/sbin"
# Might also try /sbin, /bin, /usr/bin, /usr/local/bin, etc.
permissions="+4000"  # suid root (dangerous!)


for file in $( find "$directory" -perm "$permissions" )
do
  ls -ltF --author "$file"
done
```

See [[Example 16-30|Example 16-30]], [[Example 3-4|Example 3-4]], and [[Example 11-10|Example 11-10]] for scripts using **find**. Its [[basic#^MANREF|manpage]] provides more detail on this complex and powerful command.

**xargs**

A filter for feeding arguments to a command, and also a tool for assembling the commands themselves. It breaks a data stream into small enough chunks for filters and commands to process. Consider it as a powerful replacement for [[commandsub#^BACKQUOTESREF|backquotes]]. In situations where [[commandsub#^COMMANDSUBREF|command substitution]] fails with a too many arguments error, substituting **xargs** often works. [^4] Normally, **xargs** reads from stdin or from a pipe, but it can also be given the output of a file.

The default command for **xargs** is [[internal#^ECHOREF|echo]]. This means that input piped to **xargs** may have linefeeds and other whitespace characters stripped out.

```bash
bash$ ls -l
total 0
 -rw-rw-r--    1 bozo  bozo         0 Jan 29 23:58 file1
 -rw-rw-r--    1 bozo  bozo         0 Jan 29 23:58 file2



bash$ ls -l | xargs
total 0 -rw-rw-r-- 1 bozo bozo 0 Jan 29 23:58 file1 -rw-rw-r-- 1 bozo bozo 0 Jan...



bash$ find ~/mail -type f | xargs grep "Linux"
./misc:User-Agent: slrn/0.9.8.1 (Linux)
 ./sent-mail-jul-2005: hosted by the Linux Documentation Project.
 ./sent-mail-jul-2005: (Linux Documentation Project Site, rtf version)
 ./sent-mail-jul-2005: Subject: Criticism of Bozo's Windows/Linux article
 ./sent-mail-jul-2005: while mentioning that the Linux ext2/ext3 filesystem
 . . .
	      
```

**ls | xargs -p -l gzip** [[file-and-archiving-commands#^GZIPREF|gzips]] every file in current directory, one at a time, prompting before each operation.

> [!note]
> Note that _xargs_ processes the arguments passed to it sequentially, _one at a time_.
>
> ```bash
> bash$ find /usr/bin | xargs file
> /usr/bin:          directory
>  /usr/bin/foomatic-ppd-options:          perl script text executable
>  . . .
> 	      
> ```

> [!tip] An interesting _xargs_ option is -n _NN_, which limits to _NN_ the number of arguments passed.
>
> **ls \| xargs -n 8 echo** lists the files in the current directory in 8 columns.

> [!tip]
> Another useful option is -0, in combination with **find -print0** or **grep -lZ**. This allows handling arguments containing whitespace or quotes.
>
> **find / -type f -print0 \| xargs -0 grep -liwZ GUI \| xargs -0 rm -f**
>
> **grep -rliwZ GUI / \| xargs -0 rm -f**
>
> Either of the above will remove any file containing "GUI". _(Thanks, S.C.)_
>
> Or:
>
> ```bash
> cat /proc/"$pid"/"$OPTION" | xargs -0 echo
> #  Formats output:         ^^^^^^^^^^^^^^^
> #  From Han Holl's fixup of "get-commandline.sh"
> #+ script in "/dev and /proc" chapter.
> ```

> [!tip]
> The -P option to _xargs_ permits running processes in parallel. This speeds up execution in a machine with a multicore CPU.
>
> ```bash
> #!/bin/bash
> 
> ls *gif | xargs -t -n1 -P2 gif2png
> # Converts all the gif images in current directory to png.
> 
> # Options:
> # =======
> # -t    Print command to stderr.
> # -n1   At most 1 argument per command line.
> # -P2   Run up to 2 processes simultaneously.
> 
> # Thank you, Roberto Polli, for the inspiration.
> ```

![[Example 16-5|Example 16-5]]

[[moreadv#^CURLYBRACKETSREF|As in **find**]], a curly bracket pair serves as a placeholder for replacement text.

![[Example 16-6|Example 16-6]]

![[Example 16-7|Example 16-7]]

![[Example 16-8|Example 16-8]]

**expr**

All-purpose expression evaluator: Concatenates and evaluates the arguments according to the operation given (arguments must be separated by spaces). Operations may be arithmetic, comparison, string, or logical.

**expr 3 + 5**

returns 8

**expr 5 % 3**

returns 2

**expr 1 / 0**

returns the error message, expr: division by zero

Illegal arithmetic operations not allowed.

**expr 5 \* 3**

returns 15

The multiplication operator must be escaped when used in an arithmetic expression with **expr**.

**y=`expr $y + 1`**

Increment a variable, with the same effect as **let y=y+1** and **y=$(($y+1))**. This is an example of [[arithmetic-expansion#^ARITHEXPREF|arithmetic expansion]].

**z=`expr substr $string $position $length`**

Extract substring of $length characters, starting at $position.

![[Example 16-9|Example 16-9]]

> [!important]
> The [[special-chars#^NULLREF|: (_null_)]] operator can substitute for **match**. For example, **b=`expr $a : [0-9]*`** is the exact equivalent of **b=`expr match $a [0-9]*`** in the above listing.
>
> ```bash
> #!/bin/bash
> 
> echo
> echo "String operations using \"expr \$string : \" construct"
> echo "==================================================="
> echo
> 
> a=1234zipper5FLIPPER43231
> 
> echo "The string being operated upon is \"`expr "$a" : '\(.*\)'`\"."
> #     Escaped parentheses grouping operator.            ==  ==
> 
> #       ***************************
> #+          Escaped parentheses
> #+           match a substring
> #       ***************************
> 
> 
> #  If no escaped parentheses ...
> #+ then 'expr' converts the string operand to an integer.
> 
> echo "Length of \"$a\" is `expr "$a" : '.*'`."   # Length of string
> 
> echo "Number of digits at the beginning of \"$a\" is `expr "$a" : '[0-9]*'`."
> 
> # ------------------------------------------------------------------------- #
> 
> echo
> 
> echo "The digits at the beginning of \"$a\" are `expr "$a" : '\([0-9]*\)'`."
> #                                                             ==      ==
> echo "The first 7 characters of \"$a\" are `expr "$a" : '\(.......\)'`."
> #         =====                                          ==       ==
> # Again, escaped parentheses force a substring match.
> #
> echo "The last 7 characters of \"$a\" are `expr "$a" : '.*\(.......\)'`."
> #         ====                  end of string operator  ^^
> #  (In fact, means skip over one or more of any characters until specified
> #+  substring found.)
> 
> echo
> 
> exit 0
> ```

The above script illustrates how **expr** uses the _escaped parentheses -- \( ... \) --_ grouping operator in tandem with [[regexp#^REGEXREF|regular expression]] parsing to match a substring. Here is a another example, this time from "real life."

```bash
# Strip the whitespace from the beginning and end.
LRFDATE=`expr "$LRFDATE" : '[[:space:]]*\(.*\)[[:space:]]*$'`

#  From Peter Knowles' "booklistgen.sh" script
#+ for converting files to Sony Librie/PRS-50X format.
#  (http://booklistgensh.peterknowles.com)
```

[[wrapper#^PERLREF|Perl]], [[sedawk#^SEDREF|sed]], and [[awk#^AWKREF|awk]] have far superior string parsing facilities. A short **sed** or **awk** "subroutine" within a script (see [[wrapper|Section 36.2]]) is an attractive alternative to **expr**.

See [[string-manipulation|Section 10.1]] for more on using **expr** in string operations.

## Time / Date Commands

**Time/date and timing**

## date

Simply invoked, **date** prints the date and time to stdout. Where this command gets interesting is in its formatting and parsing options.

![[Example 16-10|Example 16-10]]

The -u option gives the UTC (Universal Coordinated Time).

```bash
bash$ date
Fri Mar 29 21:07:39 MST 2002



bash$ date -u
Sat Mar 30 04:07:42 UTC 2002
	      
```

This option facilitates calculating the time between different dates.

![[Example 16-11|Example 16-11]]

The _date_ command has quite a number of _output_ options. For example %N gives the nanosecond portion of the current time. One interesting use for this is to generate random integers.

```bash
date +%N | sed -e 's/000$//' -e 's/^0//'
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#  Strip off leading and trailing zeroes, if present.
#  Length of generated integer depends on
#+ how many zeroes stripped off.

# 115281032
# 63408725
# 394504284
```

There are many more options (try **man date**).

```bash
date +%j
# Echoes day of the year (days elapsed since January 1).

date +%k%M
# Echoes hour and minute in 24-hour format, as a single digit string.



# The 'TZ' parameter permits overriding the default time zone.
date                 # Mon Mar 28 21:42:16 MST 2005
TZ=EST date          # Mon Mar 28 23:42:16 EST 2005
# Thanks, Frank Kannemann and Pete Sjoberg, for the tip.


SixDaysAgo=$(date --date='6 days ago')
OneMonthAgo=$(date --date='1 month ago')  # Four weeks back (not a month!)
OneYearAgo=$(date --date='1 year ago')
```

See also [[Example 3-4|Example 3-4]] and [[Example A-43|Example A-43]].

## zdump

Time zone dump: echoes the time in a specified time zone.

```bash
bash$ zdump EST
EST  Tue Sep 18 22:09:22 2001 EST
	    
```

## time

Outputs verbose timing statistics for executing a command.

**time ls -l /** gives something like this:

```bash
real    0m0.067s
 user    0m0.004s
 sys     0m0.005s
```

See also the very similar [[x9644#^TIMESREF|times]] command in the previous section.

> [!note]
> As of [[bashver2^#BASH2REF|version 2.0]] of Bash, **time** became a shell reserved word, with slightly altered behavior in a pipeline.

## touch

Utility for updating access/modification times of a file to current system time or other specified time, but also useful for creating a new file. The command **touch zzz** will create a new file of zero length, named zzz, assuming that zzz did not previously exist. Time-stamping empty files in this way is useful for storing date information, for example in keeping track of modification times on a project.

> [!note]
> The **touch** command is equivalent to **: >> newfile** or **>> newfile** (for ordinary files).

> [!tip]
> Before doing a [[basic#^CPREF|cp -u]] (_copy/update_), use **touch** to update the time stamp of files you don't wish overwritten.
>
> As an example, if the directory /home/bozo/tax_audit contains the files spreadsheet-051606.data, spreadsheet-051706.data, and spreadsheet-051806.data, then doing a **touch spreadsheet*.data** will protect these files from being overwritten by files with the same names during a **cp -u /home/bozo/financial_info/spreadsheet*data /home/bozo/tax_audit**.

## at

The **at** job control command executes a given set of commands at a specified time. Superficially, it resembles [[system-and-administrative-commands#^CRONREF|cron]], however, **at** is chiefly useful for one-time execution of a command set.

**at 2pm January 15** prompts for a set of commands to execute at that time. These commands should be shell-script compatible, since, for all practical purposes, the user is typing in an executable shell script a line at a time. Input terminates with a [[special-chars#^CTLDREF|Ctl-D]].

Using either the -f option or input redirection (<), **at** reads a command list from a file. This file is an executable shell script, though it should, of course, be non-interactive. Particularly clever is including the [[miscellaneous-commands#^RUNPARTSREF|run-parts]] command in the file to execute a different set of scripts.

```bash
bash$ at 2:30 am Friday < at-jobs.list
job 2 at 2000-10-27 02:30
	     
```

## batch

The **batch** job control command is similar to **at**, but it runs a command list when the system load drops below .8. Like **at**, it can read commands from a file with the -f option.

> The concept of _batch processing_ dates back to the era of mainframe computers. It means running a set of commands without user intervention.

## cal

Prints a neatly formatted monthly calendar to stdout. Will do current year or a large range of past and future years.

## sleep

This is the shell equivalent of a _wait loop_. It pauses for a specified number of seconds, doing nothing. It can be useful for timing or in processes running in the background, checking for a specific event every so often (polling), as in [[Example 32-6|Example 32-6]].

```bash
sleep 3     # Pauses 3 seconds.
```

> [!note]
> The **sleep** command defaults to seconds, but minute, hours, or days may also be specified.
>
> ```bash
> sleep 3 h   # Pauses 3 hours!
> ```

> [!note]
> The [[system-and-administrative-commands#^WATCHREF|watch]] command may be a better choice than **sleep** for running commands at timed intervals.

## usleep

_Microsleep_ (the _u_ may be read as the Greek _mu_, or _micro-_ prefix). This is the same as **sleep**, above, but "sleeps" in microsecond intervals. It can be used for fine-grained timing, or for polling an ongoing process at very frequent intervals.

```bash
usleep 30     # Pauses 30 microseconds.
```

This command is part of the Red Hat _initscripts / rc-scripts_ package.

> [!caution]
> The **usleep** command does not provide particularly accurate timing, and is therefore unsuitable for critical timing loops.

## hwclock, clock

The **hwclock** command accesses or adjusts the machine's hardware clock. Some options require _root_ privileges. The /etc/rc.d/rc.sysinit startup file uses **hwclock** to set the system time from the hardware clock at bootup.

The **clock** command is a synonym for **hwclock**.

## Text Processing Commands

**Commands affecting text and text files**

**sort**

File sort utility, often used as a filter in a pipe. This command sorts a _text stream_ or file forwards or backwards, or according to various keys or character positions. Using the -m option, it merges presorted input files. The _info page_ lists its many capabilities and options. See [[Example 11-10|Example 11-10]], [[Example 11-11|Example 11-11]], and [[Example A-8|Example A-8]].

**tsort**

_Topological sort_, reading in pairs of whitespace-separated strings and sorting according to input patterns. The original purpose of **tsort** was to sort a list of dependencies for an obsolete version of the _ld_ linker in an "ancient" version of UNIX.

The results of a _tsort_ will usually differ markedly from those of the standard **sort** command, above.

**uniq**

This filter removes duplicate lines from a sorted file. It is often seen in a pipe coupled with [[external-filters-programs-and-commands#^SORTREF|sort]].

```bash
cat list-1 list-2 list-3 | sort | uniq > final.list
# Concatenates the list files,
# sorts them,
# removes duplicate lines,
# and finally writes the result to an output file.
```

The useful -c option prefixes each line of the input file with its number of occurrences.

```bash
bash$ cat testfile
This line occurs only once.
 This line occurs twice.
 This line occurs twice.
 This line occurs three times.
 This line occurs three times.
 This line occurs three times.


bash$ uniq -c testfile
      1 This line occurs only once.
       2 This line occurs twice.
       3 This line occurs three times.


bash$ sort testfile | uniq -c | sort -nr
      3 This line occurs three times.
       2 This line occurs twice.
       1 This line occurs only once.
	      
```

The **sort INPUTFILE | uniq -c | sort -nr** command string produces a _frequency of occurrence_ listing on the INPUTFILE file (the -nr options to **sort** cause a reverse numerical sort). This template finds use in analysis of log files and dictionary lists, and wherever the lexical structure of a document needs to be examined.

![[Example 16-12|Example 16-12]]

```bash
bash$ cat testfile
This line occurs only once.
 This line occurs twice.
 This line occurs twice.
 This line occurs three times.
 This line occurs three times.
 This line occurs three times.


bash$ ./wf.sh testfile
      6 this
       6 occurs
       6 line
       3 times
       3 three
       2 twice
       1 only
       1 once
	       
```

**expand**, **unexpand**

The **expand** filter converts tabs to spaces. It is often used in a [[special-characters#^PIPEREF|pipe]].

The **unexpand** filter converts spaces to tabs. This reverses the effect of **expand**.

**cut**

A tool for extracting [[special-characters#^FIELDREF|fields]] from files. It is similar to the **print $N** command set in [[awk#^AWKREF|awk]], but more limited. It may be simpler to use _cut_ in a script than _awk_. Particularly important are the -d (delimiter) and -f (field specifier) options.

Using **cut** to obtain a listing of the mounted filesystems:

```bash
cut -d ' ' -f1,2 /etc/mtab
```

Using **cut** to list the OS and kernel version:

```bash
uname -a \| cut -d" " -f1,3,11,12
```

Using **cut** to extract message headers from an e-mail folder:

```bash
bash$ grep '^Subject:' read-messages | cut -c10-80
Re: Linux suitable for mission-critical apps?
 MAKE MILLIONS WORKING AT HOME!!!
 Spam complaint
 Re: Spam complaint
```

Using **cut** to parse a file:

```bash
# List all the users in /etc/passwd.

FILENAME=/etc/passwd

for user in $(cut -d: -f1 $FILENAME)
do
  echo $user
done

# Thanks, Oleg Philon for suggesting this.
```

**cut -d ' ' -f2,3 filename** is equivalent to **awk -F'[ ]' '{ print $2, $3 }' filename**

> [!note]
> It is even possible to specify a linefeed as a delimiter. The trick is to actually embed a linefeed (**RETURN**) in the command sequence.
>
> ```bash
> bash$ cut -d'
>  ' -f3,7,19 testfile
> This is line 3 of testfile.
>  This is line 7 of testfile.
>  This is line 19 of testfile.
> 	   
> ```
>
> Thank you, Jaka Kranjc, for pointing this out.|

See also [[Example 16-48|Example 16-48]].

**paste**

Tool for merging together different files into a single, multi-column file. In combination with [[external-filters-programs-and-commands#^CUTREF|cut]], useful for creating system log files.

```bash
bash$ cat items
alphabet blocks
 building blocks
 cables

bash$ cat prices
$1.00/dozen
 $2.50 ea.
 $3.75

bash$ paste items prices
alphabet blocks $1.00/dozen
 building blocks $2.50 ea.
 cables  $3.75
```

**join**

Consider this a special-purpose cousin of **paste**. This powerful utility allows merging two files in a meaningful fashion, which essentially creates a simple version of a relational database.

The **join** command operates on exactly two files, but pastes together only those lines with a common tagged [[special-characters#^FIELDREF|field]] (usually a numerical label), and writes the result to stdout. The files to be joined should be sorted according to the tagged field for the matchups to work properly.

```bash
File: 1.data

100 Shoes
200 Laces
300 Socks
```

```bash
File: 2.data

100 $40.00
200 $1.00
300 $2.00
```

```bash
bash$ join 1.data 2.data
File: 1.data 2.data

 100 Shoes $40.00
 200 Laces $1.00
 300 Socks $2.00
	      
```

> [!note]
> The tagged field appears only once in the output.

**head**

lists the beginning of a file to stdout. The default is 10 lines, but a different number can be specified. The command has a number of interesting options.

![[Example 16-13|Example 16-13]]

![[Example 16-14|Example 16-14]]

See also [[Example 16-39|Example 16-39]].

**tail**

lists the (tail) end of a file to stdout. The default is 10 lines, but this can be changed with the -n option. Commonly used to keep track of changes to a system logfile, using the -f option, which outputs lines appended to the file.

![[Example 16-15|Example 16-15]]

> [!tip]
> To list a specific line of a text file, [[special-characters#^PIPEREF|pipe]] the output of **head** to **tail -n 1**. For example **head -n 8 database.txt \| tail -n 1** lists the 8th line of the file database.txt.
>
> To set a variable to a given block of a text file:
>
> ```bash
> var=$(head -n $m $filename | tail -n $n)
> 
> # filename = name of file
> # m = from beginning of file, number of lines to end of block
> # n = number of lines to set variable to (trim from end of block)
> ```

> [!note]
> Newer implementations of **tail** deprecate the older **tail -$LINES filename** usage. The standard **tail -n $LINES filename** is correct.

See also [[Example 16-5|Example 16-5]], [[Example 16-39|Example 16-39]] and [[Example 32-6|Example 32-6]].

**grep**

A multi-purpose file search tool that uses [[regexp#^REGEXREF|Regular Expressions]]. It was originally a command/filter in the venerable **ed** line editor: **g/re/p** -- _global - regular expression - print_.

**grep** _pattern_ [_file_...]

Search the target file(s) for occurrences of _pattern_, where _pattern_ may be literal text or a Regular Expression.

```bash
bash$ grep '[rst]ystem.$' osinfo.txt
The GPL governs the distribution of the Linux operating system.
	      
```

If no target file(s) specified, **grep** works as a filter on stdout, as in a [[special-characters#^PIPEREF|pipe]].

```bash
bash$ ps ax | grep clock
765 tty1     S      0:00 xclock
 901 pts/1    S      0:00 grep clock
	      
```

The -i option causes a case-insensitive search.

The -w option matches only whole words.

The -l option lists only the files in which matches were found, but not the matching lines.

The -r (recursive) option searches files in the current working directory and all subdirectories below it.

The -n option lists the matching lines, together with line numbers.

```bash
bash$ grep -n Linux osinfo.txt
2:This is a file containing information about Linux.
 6:The GPL governs the distribution of the Linux operating system.
	      
```

The -v (or --invert-match) option _filters out_ matches.

```bash
grep pattern1 *.txt \| grep -v pattern2

# Matches all lines in "*.txt" files containing "pattern1",
# but ***not*** "pattern2".
```

The -c (--count) option gives a numerical count of matches, rather than actually listing the matches.

```bash
grep -c txt *.sgml   # (number of occurrences of "txt" in "*.sgml" files)


#   grep -cz .
#            ^ dot
# means count (-c) zero-separated (-z) items matching "."
# that is, non-empty ones (containing at least 1 character).
# 
printf 'a b\nc  d\n\n\n\n\n\000\n\000e\000\000\nf' \| grep -cz .     # 3
printf 'a b\nc  d\n\n\n\n\n\000\n\000e\000\000\nf' \| grep -cz '$'   # 5
printf 'a b\nc  d\n\n\n\n\n\000\n\000e\000\000\nf' \| grep -cz '^'   # 5
#
printf 'a b\nc  d\n\n\n\n\n\000\n\000e\000\000\nf' \| grep -c '$'    # 9
# By default, newline chars (\n) separate items to match. 

# Note that the -z option is GNU "grep" specific.


# Thanks, S.C.
```

The --color (or --colour) option marks the matching string in color (on the console or in an _xterm_ window). Since _grep_ prints out each entire line containing the matching pattern, this lets you see exactly _what_ is being matched. See also the -o option, which shows only the matching portion of the line(s).

![[Example 16-16|Example 16-16]]

When invoked with more than one target file given, **grep** specifies which file contains matches.

```bash
bash$ grep Linux osinfo.txt misc.txt
osinfo.txt:This is a file containing information about Linux.
 osinfo.txt:The GPL governs the distribution of the Linux operating system.
 misc.txt:The Linux operating system is steadily gaining in popularity.
```

> [!tip]
> To force **grep** to show the filename when searching only one target file, simply give /dev/null as the second file.
>
> ```bash
> bash$ grep Linux osinfo.txt /dev/null
> osinfo.txt:This is a file containing information about Linux.
>  osinfo.txt:The GPL governs the distribution of the Linux operating system.
> ```

If there is a successful match, **grep** returns an [[exit-and-exit-status#^EXITSTATUSREF|exit status]] of 0, which makes it useful in a condition test in a script, especially in combination with the -q option to suppress output.

```bash
SUCCESS=0                      # if grep lookup succeeds
word=Linux
filename=data.file

grep -q "$word" "$filename"    #  The "-q" option
                               #+ causes nothing to echo to stdout.
if [ $? -eq $SUCCESS ]
# if grep -q "$word" "$filename"   can replace lines 5 - 7.
then
  echo "$word found in $filename"
else
  echo "$word not found in $filename"
fi
```

[[Example 32-6|Example 32-6]] demonstrates how to use **grep** to search for a word pattern in a system logfile.

![[Example 16-17|Example 16-17]]

How can **grep** search for two (or more) separate patterns? What if you want **grep** to display all lines in a file or files that contain both "pattern1" _and_ "pattern2"?

One method is to [[special-characters#^PIPEREF|pipe]] the result of **grep pattern1** to **grep pattern2**.

For example, given the following file:

```bash
# Filename: tstfile

This is a sample file.
This is an ordinary text file.
This file does not contain any unusual text.
This file is not unusual.
Here is some text.
```

Now, let's search this file for lines containing _both_ "file" and "text" . . .

```bash
bash$ grep file tstfile
# Filename: tstfile
 This is a sample file.
 This is an ordinary text file.
 This file does not contain any unusual text.
 This file is not unusual.

bash$ grep file tstfile | grep text
This is an ordinary text file.
 This file does not contain any unusual text.
```

Now, for an interesting recreational use of _grep_ . . .

![[Example 16-18|Example 16-18]]

**egrep** -- _extended grep_ -- is the same as **grep -E**. This uses a somewhat different, extended set of [[regexp#^REGEXREF|Regular Expressions]], which can make the search a bit more flexible. It also allows the boolean | (_or_) operator.

```bash
bash $ egrep 'matches|Matches' file.txt
Line 1 matches.
 Line 3 Matches.
 Line 4 contains matches, but also Matches
              
```

**fgrep** -- _fast grep_ -- is the same as **grep -F**. It does a literal string search (no [[regexp#^REGEXREF|Regular Expressions]]), which generally speeds things up a bit.

> [!note]
> On some Linux distros, **egrep** and **fgrep** are symbolic links to, or aliases for **grep**, but invoked with the -E and -F options, respectively.

![[Example 16-19|Example 16-19]]

> [!note]
> See also [[Example A-41|Example A-41]] for an example of speedy _fgrep_ lookup on a large text file.

**agrep** (_approximate grep_) extends the capabilities of **grep** to approximate matching. The search string may differ by a specified number of characters from the resulting matches. This utility is not part of the core Linux distribution.

> [!tip]
> To search compressed files, use **zgrep**, **zegrep**, or **zfgrep**. These also work on non-compressed files, though slower than plain **grep**, **egrep**, **fgrep**. They are handy for searching through a mixed set of files, some compressed, some not.
>
> To search [[file-and-archiving-commands#^BZIPREF|bzipped]] files, use **bzgrep**.

**look**

The command **look** works like **grep**, but does a lookup on a "dictionary," a sorted word list. By default, **look** searches for a match in /usr/dict/words, but a different dictionary file may be specified.

![[Example 16-20|Example 16-20]]

**sed**, **awk**

Scripting languages especially suited for parsing text files and command output. May be embedded singly or in combination in pipes and shell scripts.

**[[a-sed-and-awk-micro-primer#^SEDREF|sed]]**

Non-interactive "stream editor", permits using many **ex** commands in [[external-filters-programs-and-commands#^BATCHPROCREF|batch]] mode. It finds many uses in shell scripts.

**[[awk#^AWKREF|awk]]**

Programmable file extractor and formatter, good for manipulating and/or extracting [[special-characters#^FIELDREF|fields]] (columns) in structured text files. Its syntax is similar to C.

**wc**

_wc_ gives a "word count" on a file or I/O stream:

```bash
bash $ wc /usr/share/doc/sed-4.1.2/README
13  70  447 README
[13 lines  70 words  447 characters]
```

**wc -w** gives only the word count.

**wc -l** gives only the line count.

**wc -c** gives only the byte count.

**wc -m** gives only the character count.

**wc -L** gives only the length of the longest line.

Using **wc** to count how many .txt files are in current working directory:

```bash
$ ls *.txt | wc -l
#  Will work as long as none of the "*.txt" files
#+ have a linefeed embedded in their name.

#  Alternative ways of doing this are:
#      find . -maxdepth 1 -name \*.txt -print0 | grep -cz .
#      (shopt -s nullglob; set -- *.txt; echo $#)

#  Thanks, S.C.
```

Using **wc** to total up the size of all the files whose names begin with letters in the range d - h

```bash
bash$ wc [d-h]* | grep total | awk '{print $3}'
71832
```

Using **wc** to count the instances of the word "Linux" in the main source file for this book.

```bash
bash$ grep Linux abs-book.sgml | wc -l
138
```

See also [[Example 16-39|Example 16-39]] and [[Example 20-8|Example 20-8]].

Certain commands include some of the functionality of **wc** as options.

```bash
... | grep foo | wc -l
# This frequently used construct can be more concisely rendered.

... | grep -c foo
# Just use the "-c" (or "--count") option of grep.

# Thanks, S.C.
```

**tr**

character translation filter.

> [!caution]
> [[special-characters#^UCREF|Must use quoting and/or brackets]], as appropriate. Quotes prevent the shell from reinterpreting the special characters in **tr** command sequences. Brackets should be quoted to prevent expansion by the shell.

Either **tr "A-Z" "*" <filename** or **tr A-Z \* <filename** changes all the uppercase letters in filename to asterisks (writes to stdout). On some systems this may not work, but **tr A-Z '[**]'** will.

The -d option deletes a range of characters.

```bash
echo "abcdef"                 # abcdef
echo "abcdef" | tr -d b-d     # aef


tr -d 0-9 <filename
# Deletes all digits from the file "filename".
```

The --squeeze-repeats (or -s) option deletes all but the first instance of a string of consecutive characters. This option is useful for removing excess [[special-characters#Whitespace|whitespace]].

```bash
bash$ echo "XXXXX" | tr --squeeze-repeats 'X'
X
```

The -c "complement" option _inverts_ the character set to match. With this option, **tr** acts only upon those characters _not_ matching the specified set.

```bash
bash$ echo "acfdeb123" | tr -c b-d +
+c+d+b++++
```

Note that **tr** recognizes [[brief-introduction-to-regular-expressions#^POSIXREF|POSIX character classes]]. [^5]

```bash
bash$ echo "abcd2ef1" | tr '[:alpha:]' -
----2--1
```

![[Example 16-21|Example 16-21]]

![[Example 16-22|Example 16-22]]

![[Example 16-23|Example 16-23]]

![[Example 16-24|Example 16-24]]

![[Example 16-25|Example 16-25]]

Of course, _tr_ lends itself to _code obfuscation_.

```bash
#!/bin/bash
# jabh.sh

x="wftedskaebjgdBstbdbsmnjgz"
echo $x | tr "a-z" 'oh, turtleneck Phrase Jar!'

# Based on the Wikipedia "Just another Perl hacker" article.
```

> **_tr_ variants**
>
> The **tr** utility has two historic variants. The BSD version does not use brackets (**tr a-z A-Z**), but the SysV one does (**tr '[a-z]' '[A-Z]'**). The GNU version of **tr** resembles the BSD one.

**fold**

A filter that wraps lines of input to a specified width. This is especially useful with the -s option, which breaks lines at word spaces (see [[Example 16-26|Example 16-26]] and [[Example A-1|Example A-1]]).

**fmt**

Simple-minded file formatter, used as a filter in a pipe to "wrap" long lines of text output.

![[Example 16-26|Example 16-26]]

See also [[Example 16-5|Example 16-5]].

> [!tip]
> A powerful alternative to **fmt** is Kamil Toman's **par** utility, available from [http://www.cs.berkeley.edu/~amc/Par/](http://www.cs.berkeley.edu/~amc/Par/).

**col**

This deceptively named filter removes reverse line feeds from an input stream. It also attempts to replace whitespace with equivalent tabs. The chief use of **col** is in filtering the output from certain text processing utilities, such as **groff** and **tbl**.

**column**

Column formatter. This filter transforms list-type text output into a "pretty-printed" table by inserting tabs at appropriate places.

![[Example 16-27|Example 16-27]]

**colrm**

Column removal filter. This removes columns (characters) from a file and writes the file, lacking the range of specified columns, back to stdout. **colrm 2 4 <filename** removes the second through fourth characters from each line of the text file filename.

> [!caution]
> If the file contains tabs or nonprintable characters, this may cause unpredictable behavior. In such cases, consider using [[external-filters-programs-and-commands#^EXPANDREF|expand]] and **unexpand** in a pipe preceding **colrm**.

**nl**

Line numbering filter: **nl filename** lists filename to stdout, but inserts consecutive numbers at the beginning of each non-blank line. If filename omitted, operates on stdin.

The output of **nl** is very similar to **cat -b**, since, by default **nl** does not list blank lines.

![[Example 16-28|Example 16-28]]

**pr**

Print formatting filter. This will paginate files (or stdout) into sections suitable for hard copy printing or viewing on screen. Various options permit row and column manipulation, joining lines, setting margins, numbering lines, adding page headers, and merging files, among other things. The **pr** command combines much of the functionality of **nl**, **paste**, **fold**, **column**, and **expand**.

**pr -o 5 --width=65 fileZZZ | more** gives a nice paginated listing to screen of fileZZZ with margins set at 5 and 65.

A particularly useful option is -d, forcing double-spacing (same effect as **sed -G**).

**gettext**

The GNU **gettext** package is a set of utilities for [[localization|localizing]] and translating the text output of programs into foreign languages. While originally intended for C programs, it now supports quite a number of programming and scripting languages.

The **gettext** _program_ works on shell scripts. See the _info page_.

**msgfmt**

A program for generating binary message catalogs. It is used for [[localization|localization]].

**iconv**

A utility for converting file(s) to a different encoding (character set). Its chief use is for [[localization|localization]].

```bash
# Convert a string from UTF-8 to UTF-16 and print to the BookList
function write_utf8_string {
    STRING=$1
    BOOKLIST=$2
    echo -n "$STRING" | iconv -f UTF8 -t UTF16 | \
    cut -b 3- | tr -d \\n >> "$BOOKLIST"
}

#  From Peter Knowles' "booklistgen.sh" script
#+ for converting files to Sony Librie/PRS-50X format.
#  (http://booklistgensh.peterknowles.com)
```

**recode**

Consider this a fancier version of **iconv**, above. This very versatile utility for converting a file to a different encoding scheme. Note that _recode_ is not part of the standard Linux installation.

**TeX**, **gs**

**TeX** and **Postscript** are text markup languages used for preparing copy for printing or formatted video display.

**TeX** is Donald Knuth's elaborate typsetting system. It is often convenient to write a shell script encapsulating all the options and arguments passed to one of these markup languages.

_Ghostscript_ (**gs**) is a GPL-ed Postscript interpreter.

**texexec**

Utility for processing _TeX_ and _pdf_ files. Found in /usr/bin on many Linux distros, it is actually a [[shell-wrappers#^SHWRAPPER|shell wrapper]] that calls [[shell-wrappers#^PERLREF|Perl]] to invoke _Tex_.

```bash
texexec --pdfarrange --result=Concatenated.pdf *pdf

#  Concatenates all the pdf files in the current working directory
#+ into the merged file, Concatenated.pdf . . .
#  (The --pdfarrange option repaginates a pdf file. See also --pdfcombine.)
#  The above command-line could be parameterized and put into a shell script.
```

**enscript**

Utility for converting plain text file to PostScript

For example, **enscript filename.txt -p filename.ps** produces the PostScript output file filename.ps.

**groff**, **tbl**, **eqn**

Yet another text markup and display formatting language is **groff**. This is the enhanced GNU version of the venerable UNIX **roff/troff** display and typesetting package. [[external-filters-programs-and-commands#^MANREF|Manpages]] use **groff**.

The **tbl** table processing utility is considered part of **groff**, as its function is to convert table markup into **groff** commands.

The **eqn** equation processing utility is likewise part of **groff**, and its function is to convert equation markup into **groff** commands.

![[Example 16-29|Example 16-29]]

See also [[Example A-39|Example A-39]].

**lex**, **yacc**

The **lex** lexical analyzer produces programs for pattern matching. This has been replaced by the nonproprietary **flex** on Linux systems.

The **yacc** utility creates a parser based on a set of specifications. This has been replaced by the nonproprietary **bison** on Linux systems.

[^1]: The -v option also orders the sort by _upper- and lowercase prefixed_ filenames.

[^2]: _Dotfiles_ are files whose names begin with a _dot_, such as ~/.Xdefaults. Such filenames do not appear in a normal **ls** listing (although an **ls -a** will show them), and they cannot be deleted by an accidental **rm -rf ***. Dotfiles are generally used as setup and configuration files in a user's home directory.

[^3]: This particular feature may not yet be implemented in the version of the ext2/ext3 filesystem installed on your system. Check the documentation for your Linux distro.

[^4]: And even when _xargs_ is not strictly necessary, it can speed up execution of a command involving [[external-filters-programs-and-commands#^BATCHPROCREF|batch-processing]] of multiple files.

[^5]: This is only true of the GNU version of **tr**, not the generic version often found on commercial UNIX systems.
