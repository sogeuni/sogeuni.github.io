---
title: Appendix N. Converting DOS Batch Files to Shell Scripts
---


Quite a number of programmers learned scripting on a PC running DOS. Even the crippled DOS batch file language allowed writing some fairly powerful scripts and applications, though they often required extensive kludges and workarounds. Occasionally, the need still arises to convert an old DOS batch file to a UNIX shell script. This is generally not difficult, as DOS batch file operators are only a limited subset of the equivalent shell scripting ones.

**Table N-1. Batch file keywords / variables / operators, and their shell equivalents**

|Batch File Operator|Shell Script Equivalent|Meaning|
|:--|:--|:--|
|%|$|command-line parameter prefix|
|/|-|command option flag|
|\|/|directory path separator|
|==|=|(equal-to) string comparison test|
|!==!|!=|(not equal-to) string comparison test|
|\||\||pipe|
|@|set +v|do not echo current command|
|*|*|filename "wild card"|
|>|>|file redirection (overwrite)|
|>>|>>|file redirection (append)|
|<|<|redirect stdin|
|%VAR%|$VAR|environmental variable|
|REM|#|comment|
|NOT|!|negate following test|
|NUL|/dev/null|"black hole" for burying command output|
|ECHO|echo|echo (many more option in Bash)|
|ECHO.|echo|echo blank line|
|ECHO OFF|set +v|do not echo command(s) following|
|FOR %%VAR IN (LIST) DO|for var in [list]; do|"for" loop|
|:LABEL|none (unnecessary)|label|
|GOTO|none (use a function)|jump to another location in the script|
|PAUSE|sleep|pause or wait an interval|
|CHOICE|case or select|menu choice|
|IF|if|if-test|
|IF EXIST _FILENAME_|if [ -e filename ]|test if file exists|
|IF !%N==!|if [ -z "$N" ]|if replaceable parameter "N" not present|
|CALL|source or . (dot operator)|"include" another script|
|COMMAND /C|source or . (dot operator)|"include" another script (same as CALL)|
|SET|export|set an environmental variable|
|SHIFT|shift|left shift command-line argument list|
|SGN|-lt or -gt|sign (of integer)|
|ERRORLEVEL|$?|exit status|
|CON|stdin|"console" (stdin)|
|PRN|/dev/lp0|(generic) printer device|
|LPT1|/dev/lp0|first printer device|
|COM1|/dev/ttyS0|first serial port|

Batch files usually contain DOS commands. These must be translated into their UNIX equivalents in order to convert a batch file into a shell script.

**Table N-2. DOS commands and their UNIX equivalents**

|DOS Command|UNIX Equivalent|Effect|
|:--|:--|:--|
|ASSIGN|ln|link file or directory|
|ATTRIB|chmod|change file permissions|
|CD|cd|change directory|
|CHDIR|cd|change directory|
|CLS|clear|clear screen|
|COMP|diff, comm, cmp|file compare|
|COPY|cp|file copy|
|Ctl-C|Ctl-C|break (signal)|
|Ctl-Z|Ctl-D|EOF (end-of-file)|
|DEL|rm|delete file(s)|
|DELTREE|rm -rf|delete directory recursively|
|DIR|ls -l|directory listing|
|ERASE|rm|delete file(s)|
|EXIT|exit|exit current process|
|FC|comm, cmp|file compare|
|FIND|grep|find strings in files|
|MD|mkdir|make directory|
|MKDIR|mkdir|make directory|
|MORE|more|text file paging filter|
|MOVE|mv|move|
|PATH|$PATH|path to executables|
|REN|mv|rename (move)|
|RENAME|mv|rename (move)|
|RD|rmdir|remove directory|
|RMDIR|rmdir|remove directory|
|SORT|sort|sort file|
|TIME|date|display system time|
|TYPE|cat|output file to stdout|
|XCOPY|cp|(extended) file copy|

> [!note]
> Virtually all UNIX and shell operators and commands have many more options and enhancements than their DOS and batch file counterparts. Many DOS batch files rely on auxiliary utilities, such as **ask.com**, a crippled counterpart to [[internal-commands-and-builtins#^READREF|read]].
>
> DOS supports only a very limited and incompatible subset of filename [[globbing|wild-card expansion]], recognizing just the * and ? characters.

Converting a DOS batch file into a shell script is generally straightforward, and the result ofttimes reads better than the original.

###### Example N-1. VIEWDATA.BAT: DOS Batch File

```batch
REM VIEWDATA

REM INSPIRED BY AN EXAMPLE IN "DOS POWERTOOLS"
REM                           BY PAUL SOMERSON


@ECHO OFF

IF !%1==! GOTO VIEWDATA
REM  IF NO COMMAND-LINE ARG...
FIND "%1" C:\BOZO\BOOKLIST.TXT
GOTO EXIT0
REM  PRINT LINE WITH STRING MATCH, THEN EXIT.

:VIEWDATA
TYPE C:\BOZO\BOOKLIST.TXT | MORE
REM  SHOW ENTIRE FILE, 1 PAGE AT A TIME.

:EXIT0
```

The script conversion is somewhat of an improvement. [^1]

###### Example N-2. *viewdata.sh*: Shell Script Conversion of VIEWDATA.BAT

```bash
#!/bin/bash
# viewdata.sh
# Conversion of VIEWDATA.BAT to shell script.

DATAFILE=/home/bozo/datafiles/book-collection.data
ARGNO=1

# @ECHO OFF                 Command unnecessary here.

if [ $# -lt "$ARGNO" ]    # IF !%1==! GOTO VIEWDATA
then
  less $DATAFILE          # TYPE C:\MYDIR\BOOKLIST.TXT | MORE
else
  grep "$1" $DATAFILE     # FIND "%1" C:\MYDIR\BOOKLIST.TXT
fi  

exit 0                    # :EXIT0

#  GOTOs, labels, smoke-and-mirrors, and flimflam unnecessary.
#  The converted script is short, sweet, and clean,
#+ which is more than can be said for the original.
```

Ted Davis' [Shell Scripts on the PC](http://www.maem.umr.edu/batch/) site had a set of comprehensive tutorials on the old-fashioned art of batch file programming. Unfortunately the page has vanished without a trace.

[^1]: Various readers have suggested modifications of the above batch file to prettify it and make it more compact and efficient. In the opinion of the _ABS Guide_ author, this is wasted effort. A Bash script can access a DOS filesystem, or even an NTFS partition (with the help of [[http://www.ntfs-3g.org|ntfs-3g]]) to do batch or scripted operations.
