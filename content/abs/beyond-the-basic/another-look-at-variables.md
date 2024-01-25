---
title: 9. Another Look at Variables
---


## Internal Variables

_[[internal-commands-and-builtins|Builtin]] variables:_

variables affecting bash script behavior

$BASH

The path to the _Bash_ binary itself

```bash
bash$ **echo $BASH**
/bin/bash
```

$BASH_ENV

An [[othertypesv#^ENVREF|environmental variable]] pointing to a Bash startup file to be read when a script is invoked

$BASH_SUBSHELL

A variable indicating the [[subshells#^SUBSHELLSREF|subshell]] level. This is a new addition to Bash, [[bashver3#^BASH3REF|version 3]].

See [[subshells#^SUBSHELL|Example 21-1]] for usage.

$BASHPID

_Process ID_ of the current instance of Bash. This is not the same as the [[another-look-at-variables#^PROCCID|$]] variable, but it often gives the same result.

```bash
bash4$ echo $$
11015


bash4$ echo $BASHPID
11015


bash4$ ps ax | grep bash4
11015 pts/2    R      0:00 bash4
	      
```

But ...

```bash
#!/bin/bash4

echo "\$\$ outside of subshell = $$"                              # 9602
echo "\$BASH_SUBSHELL  outside of subshell = $BASH_SUBSHELL"      # 0
echo "\$BASHPID outside of subshell = $BASHPID"                   # 9602

echo

( echo "\$\$ inside of subshell = $$"                             # 9602
  echo "\$BASH_SUBSHELL inside of subshell = $BASH_SUBSHELL"      # 1
  echo "\$BASHPID inside of subshell = $BASHPID" )                # 9603
  # Note that $$ returns PID of parent process.
```

$BASH_VERSINFO\[n]

A 6-element [[arrays#^ARRAYREF|array]] containing version information about the installed release of Bash. This is similar to $BASH_VERSION, below, but a bit more detailed.

```bash
# Bash version info:

for n in 0 1 2 3 4 5
do
  echo "BASH_VERSINFO[$n] = ${BASH_VERSINFO[$n]}"
done  

# BASH_VERSINFO[0] = 3                      # Major version no.
# BASH_VERSINFO[1] = 00                     # Minor version no.
# BASH_VERSINFO[2] = 14                     # Patch level.
# BASH_VERSINFO[3] = 1                      # Build version.
# BASH_VERSINFO[4] = release                # Release status.
# BASH_VERSINFO[5] = i386-redhat-linux-gnu  # Architecture
                                            # (same as $MACHTYPE).
```

$BASH_VERSION

The version of Bash installed on the system

```bash
bash$ echo $BASH_VERSION
3.2.25(1)-release
	      
```

```bash
tcsh% echo $BASH_VERSION
BASH_VERSION: Undefined variable.
	      
```

Checking `$BASH_VERSION` is a good method of determining which shell is running. [[another-look-at-variables#^SHELLVARREF|$SHELL]] does not necessarily give the correct answer.

$CDPATH

A colon-separated list of search paths available to the [[internal-commands-and-builtins#^CDREF|cd]] command, similar in function to the [[another-look-at-variables#^PATHREF|$PATH]] variable for binaries. The $CDPATH variable may be set in the local [[sample-bashrc-and-bash-profile-files#^BASHRC|~/.bashrc]] file.

```bash
bash$ cd bash-doc
bash: cd: bash-doc: No such file or directory


bash$ CDPATH=/usr/share/doc
bash$ cd bash-doc
/usr/share/doc/bash-doc


bash$ echo $PWD
/usr/share/doc/bash-doc
	      
```

$DIRSTACK

The top value in the directory stack [^1] (affected by [[internal-commands-and-builtins#^PUSHDREF|pushd]] and [[internal-commands-and-builtins#^POPDREF|popd]])

This builtin variable corresponds to the [[internal-commands-and-builtins#^DIRSD|dirs]] command, however **dirs** shows the entire contents of the directory stack.

$EDITOR

The default editor invoked by a script, usually **vi** or **emacs**.

$EUID

"effective" user ID number

Identification number of whatever identity the current user has assumed, perhaps by means of [[system-and-administrative-commands#^SUREF|su]].

> [!caution] The `$EUID` is not necessarily the same as the [[another-look-at-variables#^UIDREF|$UID]].

$FUNCNAME

Name of the current function

```bash
xyz23 ()
{
  echo "$FUNCNAME now executing."  # xyz23 now executing.
}

xyz23

echo "FUNCNAME = $FUNCNAME"        # FUNCNAME =
                                   # Null value outside a function.
```

See also [[contributed-scripts#^USEGETOPT|Example A-50]].

$GLOBIGNORE

A list of filename patterns to be excluded from matching in [[globbing.html|globbing]].

$GROUPS

Groups current user belongs to

This is a listing (array) of the group id numbers for current user, as recorded in [[important-files#^DATAFILESREF1|/etc/passwd]] and /etc/group.

```bash
root# echo $GROUPS
0


root# echo ${GROUPS[1]}
1


root# echo ${GROUPS[5]}
6
	      
```

$HOME

Home directory of the user, usually /home/username (see [[parameter-substitution#^EX6|Example 10-7]])

$HOSTNAME

The [[system-and-administrative-commands#^HNAMEREF|hostname]] command assigns the system host name at bootup in an init script. However, the gethostname() function sets the Bash internal variable $HOSTNAME. See also [[parameter-substitution#^EX6|Example 10-7]].

$HOSTTYPE

host type

Like [[another-look-at-variables#^MACHTYPEREF|$MACHTYPE]], identifies the system hardware.

```bash
bash$ echo $HOSTTYPE
i686
```

$IFS

internal field separator

This variable determines how Bash recognizes [[special-characters#^FIELDREF|fields]], or word boundaries, when it interprets character strings.

`$IFS` defaults to [[special-characters#Whitespace|whitespace]] (space, tab, and newline), but may be changed, for example, to parse a comma-separated data file. Note that [[another-look-at-variables#^APPREF|$*]] uses the first character held in $IFS. See [[quoting#^WEIRDVARS|Example 5-1]].

```bash
bash$ echo "$IFS"

(With $IFS set to default, a blank line displays.)
	      


bash$ echo "$IFS" | cat -vte
 ^I$
 $
(Show whitespace: here a single space, ^I [horizontal tab],
  and newline, and display "$" at end-of-line.)



bash$ bash -c 'set w x y z; IFS=":-;"; echo "$*"'
w:x:y:z
(Read commands from string and assign any arguments to pos params.)
	      
```

Set $IFS to eliminate whitespace in [[special-characters#^PATHNAMEREF|pathnames]].

```bash
IFS="$(printf '\n\t')"   # Per David Wheeler.
```

> [!caution] $IFS does not handle whitespace the same as it does other characters.
>
> **Example 9-1. $IFS and whitespace**
>
> ```bash
> #!/bin/bash
> # ifs.sh
> 
> 
> var1="a+b+c"
> var2="d-e-f"
> var3="g,h,i"
> 
> IFS=+
> # The plus sign will be interpreted as a separator.
> echo $var1     # a b c
> echo $var2     # d-e-f
> echo $var3     # g,h,i
> 
> echo
> 
> IFS="-"
> # The plus sign reverts to default interpretation.
> # The minus sign will be interpreted as a separator.
> echo $var1     # a+b+c
> echo $var2     # d e f
> echo $var3     # g,h,i
> 
> echo
> 
> IFS=","
> # The comma will be interpreted as a separator.
> # The minus sign reverts to default interpretation.
> echo $var1     # a+b+c
> echo $var2     # d-e-f
> echo $var3     # g h i
> 
> echo
> 
> IFS=" "
> # The space character will be interpreted as a separator.
> # The comma reverts to default interpretation.
> echo $var1     # a+b+c
> echo $var2     # d-e-f
> echo $var3     # g,h,i
> 
> # ======================================================== #
> 
> # However ...
> # $IFS treats whitespace differently than other characters.
> 
> output_args_one_per_line()
> {
>   for arg
>   do
>     echo "[$arg]"
>   done #  ^    ^   Embed within brackets, for your viewing pleasure.
> }
> 
> echo; echo "IFS=\" \""
> echo "-------"
> 
> IFS=" "
> var=" a  b c   "
> #    ^ ^^   ^^^
> output_args_one_per_line $var  # output_args_one_per_line `echo " a  b c   "`
> # [a]
> # [b]
> # [c]
> 
> 
> echo; echo "IFS=:"
> echo "-----"
> 
> IFS=:
> var=":a::b:c:::"               # Same pattern as above,
> #    ^ ^^   ^^^                #+ but substituting ":" for " "  ...
> output_args_one_per_line $var
> # []
> # [a]
> # []
> # [b]
> # [c]
> # []
> # []
> 
> # Note "empty" brackets.
> # The same thing happens with the "FS" field separator in awk.
> 
> 
> echo
> 
> exit
> ```
>
> (Many thanks, Stéphane Chazelas, for clarification and above examples.)
>
> See also [[communications-commands#^ISSPAMMER|Example 16-41]], [[loops#^BINGREP|Example 11-8]], and [[here-strings#^MAILBOXGREP|Example 19-14]] for instructive examples of using $IFS.

$IGNOREEOF

Ignore EOF: how many end-of-files (control-D) the shell will ignore before logging out.

$LC_COLLATE

Often set in the [[sample-bashrc-and-bash-profile-files|.bashrc]] or /etc/profile files, this variable controls collation order in filename expansion and pattern matching. If mishandled, LC_COLLATE can cause unexpected results in [[globbing|filename globbing]].

> [!note]
> As of version 2.05 of Bash, filename globbing no longer distinguishes between lowercase and uppercase letters in a character range between brackets. For example, **ls [A-M]*** would match both File1.txt and file1.txt. To revert to the customary behavior of bracket matching, set LC_COLLATE to C by an **export LC_COLLATE=C** in /etc/profile and/or ~/.bashrc.

$LC_CTYPE

This internal variable controls character interpretation in [[globbing.html|globbing]] and pattern matching.

$LINENO

This variable is the line number of the shell script in which this variable appears. It has significance only within the script in which it appears, and is chiefly useful for debugging purposes.

```bash
# *** BEGIN DEBUG BLOCK ***
last_cmd_arg=$_  # Save it.

echo "At line number $LINENO, variable \"v1\" = $v1"
echo "Last command argument processed = $last_cmd_arg"
# *** END DEBUG BLOCK ***
```

$MACHTYPE

machine type

Identifies the system hardware.

```bash
bash$ echo $MACHTYPE
i686
```

$OLDPWD

Old working directory ("OLD-Print-Working-Directory", previous directory you were in).

$OSTYPE

operating system type

```bash
bash$ echo $OSTYPE
linux
```

$PATH

Path to binaries, usually /usr/bin/, /usr/X11R6/bin/, /usr/local/bin, etc.

When given a command, the shell automatically does a hash table search on the directories listed in the _path_ for the executable. The path is stored in the [[othertypesv#^ENVREF|environmental variable]], $PATH, a list of directories, separated by colons. Normally, the system stores the $PATH definition in /etc/profile and/or [[sample-bashrc-and-bash-profile-files|~/.bashrc]] (see [[important-files|Appendix H]]).

```bash
bash$ echo $PATH
/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin:/sbin:/usr/sbin
```

**PATH=${PATH}:/opt/bin** appends the /opt/bin directory to the current path. In a script, it may be expedient to temporarily add a directory to the path in this way. When the script exits, this restores the original $PATH (a child process, such as a script, may not change the environment of the parent process, the shell).

> [!note] The current "working directory", ./, is usually omitted from the $PATH as a security measure.

$PIPESTATUS

[[arrays#^ARRAYREF|Array]] variable holding [[exit-and-exit-status#^EXITSTATUSREF|exit status]](es) of last executed _foreground_ [[special-characters#^PIPEREF|pipe]].

```bash
bash$ echo $PIPESTATUS
0

bash$ ls -al | bogus_command
bash: bogus_command: command not found
bash$ echo ${PIPESTATUS[1]}
127

bash$ ls -al | bogus_command
bash: bogus_command: command not found
bash$ echo $?
127
	      
```

The members of the $PIPESTATUS array hold the exit status of each respective command executed in a pipe. $PIPESTATUS[0] holds the exit status of the first command in the pipe, $PIPESTATUS[1] the exit status of the second command, and so on.

> [!caution]
> The $PIPESTATUS variable may contain an erroneous 0 value in a login shell (in releases prior to 3.0 of Bash).
>
> ```bash
> tcsh% bash
> 
> bash$ who | grep nobody | sort
> bash$ echo ${PIPESTATUS[*]}
> 0
> 	    
> ```
>
> The above lines contained in a script would produce the expected 0 1 0 output.
>
> Thank you, Wayne Pollock for pointing this out and supplying the above example.|

> [!note]
> The $PIPESTATUS variable gives unexpected results in some contexts.
>
> ```bash
> bash$ echo $BASH_VERSION
> 3.00.14(1)-release
> 
> bash$ $ ls | bogus_command | wc
> bash: bogus_command: command not found
>  0       0       0
> 
> bash$ echo ${PIPESTATUS[@]}
> 141 127 0
> 
> ```
>
> Chet Ramey attributes the above output to the behavior of [[basic-commands#^LSREF|ls]]. If _ls_ writes to a _pipe_ whose output is not read, then _SIGPIPE_ kills it, and its [[exit-and-exit-status#^EXITSTATUSREF|exit status]] is 141. Otherwise its exit status is 0, as expected. This likewise is the case for [[text-processing-commands#^TRREF|tr]].

> [!note]
> $PIPESTATUS is a "volatile" variable. It needs to be captured immediately after the pipe in question, before any other command intervenes.
>
> ```bash
> bash$ $ ls | bogus_command | wc
> bash: bogus_command: command not found
>  0       0       0
> 
> bash$ echo ${PIPESTATUS[@]}
> 0 127 0
> 
> bash$ echo ${PIPESTATUS[@]}
> 0
> 	      
> ```

> [!note] The [[bashver3#^PIPEFAILREF|pipefail option]] may be useful in cases where $PIPESTATUS does not give the desired information.

$PPID

The $PPID of a process is the process ID (pid) of its parent process. [^2]

Compare this with the [[system-and-administrative-commands#^PIDOFREF|pidof]] command.

$PROMPT_COMMAND

A variable holding a command to be executed just before the primary prompt, $PS1 is to be displayed.

$PS1

This is the main prompt, seen at the command-line.

$PS2

The secondary prompt, seen when additional input is expected. It displays as ">".

$PS3

The tertiary prompt, displayed in a [[testing-and-branching#^SELECTREF|select]] loop (see [[testing-and-branching#^EX31|Example 11-30]]).

$PS4

The quartenary prompt, shown at the beginning of each line of output when invoking a script with the -x _[[options#^OPTIONSREF|verbose trace]_ [option]]. It displays as "+".

As a debugging aid, it may be useful to embed diagnostic information in $PS4.

```bash
P4='$(read time junk < /proc/$$/schedstat; echo "@@@ $time @@@ " )'
# Per suggestion by Erik Brandsberg.
set -x
# Various commands follow ...
```

$PWD

Working directory (directory you are in at the time)

This is the analog to the [[internal-commands-and-builtins#^PWD2REF|pwd]] builtin command.

```bash
#!/bin/bash

E_WRONG_DIRECTORY=85

clear # Clear the screen.

TargetDirectory=/home/bozo/projects/GreatAmericanNovel

cd $TargetDirectory
echo "Deleting stale files in $TargetDirectory."

if [ "$PWD" != "$TargetDirectory" ]
then    # Keep from wiping out wrong directory by accident.
  echo "Wrong directory!"
  echo "In $PWD, rather than $TargetDirectory!"
  echo "Bailing out!"
  exit $E_WRONG_DIRECTORY
fi  

rm -rf *
rm .[A-Za-z0-9]*    # Delete dotfiles.
# rm -f .[^.]* ..?*   to remove filenames beginning with multiple dots.
# (shopt -s dotglob; rm -f *)   will also work.
# Thanks, S.C. for pointing this out.

#  A filename (`basename`) may contain all characters in the 0 - 255 range,
#+ except "/".
#  Deleting files beginning with weird characters, such as -
#+ is left as an exercise. (Hint: rm ./-weirdname or rm -- -weirdname)
result=$?   # Result of delete operations. If successful = 0.

echo
ls -al              # Any files left?
echo "Done."
echo "Old files deleted in $TargetDirectory."
echo

# Various other operations here, as necessary.

exit $result
```

$REPLY

The default value when a variable is not supplied to [[internal-commands-and-builtins#^READREF|read]]. Also applicable to [[testing-and-branching#^SELECTREF|select]] menus, but only supplies the item number of the variable chosen, not the value of the variable itself.

```bash
#!/bin/bash
# reply.sh

# REPLY is the default value for a 'read' command.

echo
echo -n "What is your favorite vegetable? "
read

echo "Your favorite vegetable is $REPLY."
#  REPLY holds the value of last "read" if and only if
#+ no variable supplied.

echo
echo -n "What is your favorite fruit? "
read fruit
echo "Your favorite fruit is $fruit."
echo "but..."
echo "Value of \$REPLY is still $REPLY."
#  $REPLY is still set to its previous value because
#+ the variable $fruit absorbed the new "read" value.

echo

exit 0
```

$SECONDS

The number of seconds the script has been running.

```bash
#!/bin/bash

TIME_LIMIT=10
INTERVAL=1

echo
echo "Hit Control-C to exit before $TIME_LIMIT seconds."
echo

while [ "$SECONDS" -le "$TIME_LIMIT" ]
do   #   $SECONDS is an internal shell variable.
  if [ "$SECONDS" -eq 1 ]
  then
    units=second
  else  
    units=seconds
  fi

  echo "This script has been running $SECONDS $units."
  #  On a slow or overburdened machine, the script may skip a count
  #+ every once in a while.
  sleep $INTERVAL
done

echo -e "\a"  # Beep!

exit 0
```

$SHELLOPTS

The list of enabled shell [[options#^OPTIONSREF|options]], a readonly variable.

```bash
bash$ echo $SHELLOPTS
braceexpand:hashall:histexpand:monitor:history:interactive-comments:emacs
	      
```

$SHLVL

Shell level, how deeply Bash is nested. [^3] If, at the command-line, $SHLVL is 1, then in a script it will increment to 2.

> [!note] This variable is [[subshells#^SUBSHNLEVREF|_not_ affected by subshells]]. Use [[another-look-at-variables#^BASHSUBSHELLREF|$BASH_SUBSHELL]] when you need an indication of subshell nesting.

$TMOUT

If the _$TMOUT_ environmental variable is set to a non-zero value time, then the shell prompt will time out after $time seconds. This will cause a logout.

As of version 2.05b of Bash, it is now possible to use _$TMOUT_ in a script in combination with [[internal-commands-and-builtins#^READREF|read]].

```bash
# Works in scripts for Bash, versions 2.05b and later.

TMOUT=3    # Prompt times out at three seconds.

echo "What is your favorite song?"
echo "Quickly now, you only have $TMOUT seconds to answer!"
read song

if [ -z "$song" ]
then
  song="(no answer)"
  # Default response.
fi

echo "Your favorite song is $song."
```

There are other, more complex, ways of implementing timed input in a script. One alternative is to set up a timing loop to signal the script when it times out. This also requires a signal handling routine to [[debugging#^TRAPREF1|trap]] (see [[debugging#^EX76|Example 32-5]]) the interrupt generated by the timing loop (whew!).

###### Example 9-2. Timed Input

```bash
#!/bin/bash
# timed-input.sh

# TMOUT=3    Also works, as of newer versions of Bash.

TIMER_INTERRUPT=14
TIMELIMIT=3  # Three seconds in this instance.
             # May be set to different value.

PrintAnswer()
{
  if [ "$answer" = TIMEOUT ]
  then
    echo $answer
  else       # Don't want to mix up the two instances. 
    echo "Your favorite veggie is $answer"
    kill $!  #  Kills no-longer-needed TimerOn function
             #+ running in background.
             #  $! is PID of last job running in background.
  fi

}  


TimerOn()
{
  sleep $TIMELIMIT && kill -s 14 $$ &
  # Waits 3 seconds, then sends sigalarm to script.
}  


Int14Vector()
{
  answer="TIMEOUT"
  PrintAnswer
  exit $TIMER_INTERRUPT
}  

trap Int14Vector $TIMER_INTERRUPT
# Timer interrupt (14) subverted for our purposes.

echo "What is your favorite vegetable "
TimerOn
read answer
PrintAnswer


#  Admittedly, this is a kludgy implementation of timed input.
#  However, the "-t" option to "read" simplifies this task.
#  See the "t-out.sh" script.
#  However, what about timing not just single user input,
#+ but an entire script?

#  If you need something really elegant ...
#+ consider writing the application in C or C++,
#+ using appropriate library functions, such as 'alarm' and 'setitimer.'

exit 0
```

An alternative is using [[system-and-administrative-commands#^STTYREF|stty]].

###### Example 9-3. Once more, timed input

```bash
#!/bin/bash
# timeout.sh

#  Written by Stephane Chazelas,
#+ and modified by the document author.

INTERVAL=5                # timeout interval

timedout_read() {
  timeout=$1
  varname=$2
  old_tty_settings=`stty -g`
  stty -icanon min 0 time ${timeout}0
  eval read $varname      # or just  read $varname
  stty "$old_tty_settings"
  # See man page for "stty."
}

echo; echo -n "What's your name? Quick! "
timedout_read $INTERVAL your_name

#  This may not work on every terminal type.
#  The maximum timeout depends on the terminal.
#+ (it is often 25.5 seconds).

echo

if [ ! -z "$your_name" ]  # If name input before timeout ...
then
  echo "Your name is $your_name."
else
  echo "Timed out."
fi

echo

# The behavior of this script differs somewhat from "timed-input.sh."
# At each keystroke, the counter resets.

exit 0
```

Perhaps the simplest method is using the -t option to [[internal-commands-and-builtins#^READREF|read]].

###### Example 9-4. Timed *read*

```bash
#!/bin/bash
# t-out.sh [time-out]
# Inspired by a suggestion from "syngin seven" (thanks).


TIMELIMIT=4         # 4 seconds

read -t $TIMELIMIT variable <&1
#                           ^^^
#  In this instance, "<&1" is needed for Bash 1.x and 2.x,
#  but unnecessary for Bash 3+.

echo

if [ -z "$variable" ]  # Is null?
then
  echo "Timed out, variable still unset."
else  
  echo "variable = $variable"
fi  

exit 0
```

$UID

User ID number

Current user's user identification number, as recorded in [[important-files#^DATAFILESREF1|/etc/passwd]]

This is the current user's real id, even if she has temporarily assumed another identity through [[system-and-administrative-commands#^SUREF|su]]. $UID is a readonly variable, not subject to change from the command line or within a script, and is the counterpart to the [[system-and-administrative-commands#^IDREF|id]] builtin.

###### Example 9-5. Am I root?

```bash
#!/bin/bash
# am-i-root.sh:   Am I root or not?

ROOT_UID=0   # Root has $UID 0.

if [ "$UID" -eq "$ROOT_UID" ]  # Will the real "root" please stand up?
then
  echo "You are root."
else
  echo "You are just an ordinary user (but mom loves you just the same)."
fi

exit 0


# ============================================================= #
# Code below will not execute, because the script already exited.

# An alternate method of getting to the root of matters:

ROOTUSER_NAME=root

username=`id -nu`              # Or...   username=`whoami`
if [ "$username" = "$ROOTUSER_NAME" ]
then
  echo "Rooty, toot, toot. You are root."
else
  echo "You are just a regular fella."
fi
```

See also [[example 2-3|Example 2-3]].

> [!note]
> The variables $ENV, $LOGNAME, $MAIL, $TERM, $USER, and $USERNAME are _not_ Bash [[internal-commands-and-builtins|builtins]]. These are, however, often set as [[othertypesv#^ENVREF|environmental variables]] in one of the [[important-files#^FILESREF1|Bash]] or _login_ startup files. $SHELL, the name of the user's login shell, may be set from /etc/passwd or in an "init" script, and it is likewise not a Bash builtin.
>
> ```bash
> tcsh% echo $LOGNAME
> bozo
> tcsh% echo $SHELL
> /bin/tcsh
> tcsh% echo $TERM
> rxvt
> 
> bash$ echo $LOGNAME
> bozo
> bash$ echo $SHELL
> /bin/tcsh
> bash$ echo $TERM
> rxvt
> 	      
> ```

**Positional Parameters**

$0, $1, $2, etc.

Positional parameters, passed from command line to script, passed to a function, or [[internal-commands-and-builtins#^SETREF|set]] to a variable (see [[othertypesv#^EX17|Example 4-5]] and [[internal-commands-and-builtins#^EX34|Example 15-16]])

$#

Number of command-line arguments [[shell-wrappers#^EX4|^4] or positional parameters (see [Example 36-2]])

$*

All of the positional parameters, seen as a single word

> [!note] "$*" must be quoted.

$@

Same as $*, but each parameter is a quoted string, that is, the parameters are passed on intact, without interpretation or expansion. This means, among other things, that each parameter in the argument list is seen as a separate word.

> [!note] Of course, "$@" should be quoted.

###### Example 9-6. *arglist*: Listing arguments with $* and $@

```bash
#!/bin/bash
# arglist.sh
# Invoke this script with several arguments, such as "one two three" ...

E_BADARGS=85

if [ ! -n "$1" ]
then
  echo "Usage: `basename $0` argument1 argument2 etc."
  exit $E_BADARGS
fi  

echo

index=1          # Initialize count.

echo "Listing args with \"\$*\":"
for arg in "$*"  # Doesn't work properly if "$*" isn't quoted.
do
  echo "Arg #$index = $arg"
  let "index+=1"
done             # $* sees all arguments as single word. 
echo "Entire arg list seen as single word."

echo

index=1          # Reset count.
                 # What happens if you forget to do this?

echo "Listing args with \"\$@\":"
for arg in "$@"
do
  echo "Arg #$index = $arg"
  let "index+=1"
done             # $@ sees arguments as separate words. 
echo "Arg list seen as separate words."

echo

index=1          # Reset count.

echo "Listing args with \$* (unquoted):"
for arg in $*
do
  echo "Arg #$index = $arg"
  let "index+=1"
done             # Unquoted $* sees arguments as separate words. 
echo "Arg list seen as separate words."

exit 0
```

Following a **shift**, the $@ holds the remaining command-line parameters, lacking the previous $1, which was lost.

```bash
#!/bin/bash
# Invoke with ./scriptname 1 2 3 4 5

echo "$@"    # 1 2 3 4 5
shift
echo "$@"    # 2 3 4 5
shift
echo "$@"    # 3 4 5

# Each "shift" loses parameter $1.
# "$@" then contains the remaining parameters.
```

The $@ special parameter finds use as a tool for filtering input into shell scripts. The **cat "$@"** construction accepts input to a script either from stdin or from files given as parameters to the script. See [[text-processing-commands#^ROT13|Example 16-24]] and [[text-processing-commands#^CRYPTOQUOTE|Example 16-25]].

> [!caution] The $* and $@ parameters sometimes display inconsistent and puzzling behavior, depending on the setting of [[another-look-at-variables#^IFSREF|$IFS]].

###### Example 9-7. Inconsistent $* and $@ behavior

```bash
#!/bin/bash

#  Erratic behavior of the "$*" and "$@" internal Bash variables,
#+ depending on whether or not they are quoted.
#  Demonstrates inconsistent handling of word splitting and linefeeds.


set -- "First one" "second" "third:one" "" "Fifth: :one"
# Setting the script arguments, $1, $2, $3, etc.

echo

echo 'IFS unchanged, using "$*"'
c=0
for i in "$*"               # quoted
do echo "$((c+=1)): [$i]"   # This line remains the same in every instance.
                            # Echo args.
done
echo ---

echo 'IFS unchanged, using $*'
c=0
for i in $*                 # unquoted
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS unchanged, using "$@"'
c=0
for i in "$@"
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS unchanged, using $@'
c=0
for i in $@
do echo "$((c+=1)): [$i]"
done
echo ---

IFS=:
echo 'IFS=":", using "$*"'
c=0
for i in "$*"
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using $*'
c=0
for i in $*
do echo "$((c+=1)): [$i]"
done
echo ---

var=$*
echo 'IFS=":", using "$var" (var=$*)'
c=0
for i in "$var"
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using $var (var=$*)'
c=0
for i in $var
do echo "$((c+=1)): [$i]"
done
echo ---

var="$*"
echo 'IFS=":", using $var (var="$*")'
c=0
for i in $var
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using "$var" (var="$*")'
c=0
for i in "$var"
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using "$@"'
c=0
for i in "$@"
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using $@'
c=0
for i in $@
do echo "$((c+=1)): [$i]"
done
echo ---

var=$@
echo 'IFS=":", using $var (var=$@)'
c=0
for i in $var
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using "$var" (var=$@)'
c=0
for i in "$var"
do echo "$((c+=1)): [$i]"
done
echo ---

var="$@"
echo 'IFS=":", using "$var" (var="$@")'
c=0
for i in "$var"
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using $var (var="$@")'
c=0
for i in $var
do echo "$((c+=1)): [$i]"
done

echo

# Try this script with ksh or zsh -y.

exit 0

#  This example script written by Stephane Chazelas,
#+ and slightly modified by the document author.
```

> [!note] The **$@** and **$*** parameters differ only when between double quotes.

###### Example 9-8. $* and $@ when $IFS is empty

```bash
#!/bin/bash

#  If $IFS set, but empty,
#+ then "$*" and "$@" do not echo positional params as expected.

mecho ()       # Echo positional parameters.
{
echo "$1,$2,$3";
}


IFS=""         # Set, but empty.
set a b c      # Positional parameters.

mecho "$*"     # abc,,
#                   ^^
mecho $*       # a,b,c

mecho $@       # a,b,c
mecho "$@"     # a,b,c

#  The behavior of $* and $@ when $IFS is empty depends
#+ on which Bash or sh version being run.
#  It is therefore inadvisable to depend on this "feature" in a script.


# Thanks, Stephane Chazelas.

exit
```

**Other Special Parameters**

$-

Flags passed to script (using [[internal-commands-and-builtins#^SETREF|set]]). See [[internal-commands-and-builtins#^EX34|Example 15-16]].

> [!caution] This was originally a _ksh_ construct adopted into Bash, and unfortunately it does not seem to work reliably in Bash scripts. One possible use for it is to have a script [[interactive-and-non-interactive-shell-and-scripts#^IITEST|self-test whether it is interactive]].

$!

[[special-characters#^PROCESSIDDEF|PID]] (process ID) of last job run in background

```bash
LOG=$0.log

COMMAND1="sleep 100"

echo "Logging PIDs background commands for script: $0" >> "$LOG"
# So they can be monitored, and killed as necessary.
echo >> "$LOG"

# Logging commands.

echo -n "PID of \"$COMMAND1\":  " >> "$LOG"
${COMMAND1} &
echo $! >> "$LOG"
# PID of "sleep 100":  1506

# Thank you, Jacques Lederer, for suggesting this.
```

Using $! for job control:

```bash
possibly_hanging_job & { sleep ${TIMEOUT}; eval 'kill -9 $!' &> /dev/null; }
# Forces completion of an ill-behaved program.
# Useful, for example, in init scripts.

# Thank you, Sylvain Fourmanoit, for this creative use of the "!" variable.
```

Or, alternately:

```bash
# This example by Matthew Sage.
# Used with permission.

TIMEOUT=30   # Timeout value in seconds
count=0

possibly_hanging_job & {
        while ((count < TIMEOUT )); do
                eval '[ ! -d "/proc/$!" ] && ((count = TIMEOUT))'
                # /proc is where information about running processes is found.
                # "-d" tests whether it exists (whether directory exists).
                # So, we're waiting for the job in question to show up.
                ((count++))
                sleep 1
        done
        eval '[ -d "/proc/$!" ] && kill -15 $!'
        # If the hanging job is running, kill it.
}

#  -------------------------------------------------------------- #

#  However, this may not not work as specified if another process
#+ begins to run after the "hanging_job" . . .
#  In such a case, the wrong job may be killed.
#  Ariel Meragelman suggests the following fix.

TIMEOUT=30
count=0
# Timeout value in seconds
possibly_hanging_job & {

while ((count < TIMEOUT )); do
  eval '[ ! -d "/proc/$lastjob" ] && ((count = TIMEOUT))'
  lastjob=$!
  ((count++))
  sleep 1
done
eval '[ -d "/proc/$lastjob" ] && kill -15 $lastjob'

}

exit
```

$_

Special variable set to final argument of previous command executed.

###### Example 9-9. Underscore variable

```bash
#!/bin/bash

echo $_              #  /bin/bash
                     #  Just called /bin/bash to run the script.
                     #  Note that this will vary according to
                     #+ how the script is invoked.

du >/dev/null        #  So no output from command.
echo $_              #  du

ls -al >/dev/null    #  So no output from command.
echo $_              #  -al  (last argument)

:
echo $_              #  :
```

$?

[[exit-and-exit-status#^EXITSTATUSREF|Exit status]] of a command, [[functions|function]], or the script itself (see [[complex-functions-and-function-complexities#^MAX|Example 24-7]])

`$$`

Process ID (_PID_) of the script itself. [[debugging#^ONLINE|^5] The $ variable often finds use in scripts to construct "unique" temp file names (see [Example 32-6]], [[file-and-archiving-commands#^DERPM|Example 16-31]], and [[job-control-commands#^SELFDESTRUCT|Example 15-27]]). This is usually simpler than invoking [[file-and-archiving-commands#^MKTEMPREF|mktemp]].

## Typing variables: **declare** or **typeset**

The _declare_ or _typeset_ [[internal-commands-and-builtins|builtins]], which are exact synonyms, permit modifying the properties of variables. This is a very weak form of the _typing_ [^6] available in certain programming languages. The _declare_ command is specific to version 2 or later of Bash. The _typeset_ command also works in ksh scripts.

**declare/typeset options**

-r _readonly_

(**declare -r var1** works the same as **readonly var1**)

This is the rough equivalent of the **C** _const_ type qualifier. An attempt to change the value of a _readonly_ variable fails with an error message.

```bash
declare -r var1=1
echo "var1 = $var1"   # var1 = 1

(( var1++ ))          # x.sh: line 4: var1: readonly variable
```

-i _integer_

```bash
declare -i number
# The script will treat subsequent occurrences of "number" as an integer.		

number=3
echo "Number = $number"     # Number = 3

number=three
echo "Number = $number"     # Number = 0
# Tries to evaluate the string "three" as an integer.
```

Certain arithmetic operations are permitted for declared integer variables without the need for [[complex-commands#^EXPRREF|expr]] or [[internal-commands-and-builtins#^LETREF|let]].

```bash
n=6/3
echo "n = $n"       # n = 6/3

declare -i n
n=6/3
echo "n = $n"       # n = 2
```

-a _array_

```bash
declare -a indices
```

The variable _indices_ will be treated as an [[arrays#^ARRAYREF|array]].

-f _function(s)_

```bash
declare -f
```

A **declare -f** line with no arguments in a script causes a listing of all the [[functions|functions]] previously defined in that script.

```bash
declare -f function_name
```

A **declare -f function_name** in a script lists just the function named.

-x [[internal-commands-and-builtins#^EXPORTREF|export]]

```bash
declare -x var3
```

This declares a variable as available for exporting outside the environment of the script itself.

-x var=$value

```bash
declare -x var3=373
```

The **declare** command permits assigning a value to a variable in the same statement as setting its properties.

###### Example 9-10. Using *declare* to type variables

```bash
#!/bin/bash

func1 ()
{
  echo This is a function.
}

declare -f        # Lists the function above.

echo

declare -i var1   # var1 is an integer.
var1=2367
echo "var1 declared as $var1"
var1=var1+1       # Integer declaration eliminates the need for 'let'.
echo "var1 incremented by 1 is $var1."
# Attempt to change variable declared as integer.
echo "Attempting to change var1 to floating point value, 2367.1."
var1=2367.1       # Results in error message, with no change to variable.
echo "var1 is still $var1"

echo

declare -r var2=13.36         # 'declare' permits setting a variable property
                              #+ and simultaneously assigning it a value.
echo "var2 declared as $var2" # Attempt to change readonly variable.
var2=13.37                    # Generates error message, and exit from script.

echo "var2 is still $var2"    # This line will not execute.

exit 0                        # Script will not exit here.
```

> [!caution]
> Using the _declare_ builtin restricts the [[subshells#^SCOPEREF|scope]] of a variable.
>
> ```bash
> foo ()
> {
> FOO="bar"
> }
> 
> bar ()
> {
> foo
> echo $FOO
> }
> 
> bar   # Prints bar.
> ```
>
> However . . .
>
> ```bash
> foo (){
> declare FOO="bar"
> }
> 
> bar ()
> {
> foo
> echo $FOO
> }
> 
> bar  # Prints nothing.
> 
> 
> # Thank you, Michael Iatrou, for pointing this out.
> ```

### 9.2.1. Another use for _declare_

The _declare_ command can be helpful in identifying variables, [[othertypesv#^ENVREF|environmental]] or otherwise. This can be especially useful with [[arrays#^ARRAYREF|arrays]].

```bash
bash$ declare | grep HOME
HOME=/home/bozo


bash$ zzy=68
bash$ declare | grep zzy
zzy=68


bash$ Colors=([0]="purple" [1]="reddish-orange" [2]="light green")
bash$ echo ${Colors[@]}
purple reddish-orange light green
bash$ declare | grep Colors
Colors=([0]="purple" [1]="reddish-orange" [2]="light green")
	     
```

## $RANDOM: generate random integer

> Anyone who attempts to generate random numbers by deterministic means is, of course, living in a state of sin.
>
> --<cite>John von Neumann</cite>

$RANDOM is an internal Bash [[functions|function]] (not a constant) that returns a _pseudorandom_ [^7] integer in the range 0 - 32767. It should _not_ be used to generate an encryption key.

###### Example 9-11. Generating random numbers

```bash
#!/bin/bash

# $RANDOM returns a different random integer at each invocation.
# Nominal range: 0 - 32767 (signed 16-bit integer).

MAXCOUNT=10
count=1

echo
echo "$MAXCOUNT random numbers:"
echo "-----------------"
while [ "$count" -le $MAXCOUNT ]      # Generate 10 ($MAXCOUNT) random integers.
do
  number=$RANDOM
  echo $number
  let "count += 1"  # Increment count.
done
echo "-----------------"

# If you need a random int within a certain range, use the 'modulo' operator.
# This returns the remainder of a division operation.

RANGE=500

echo

number=$RANDOM
let "number %= $RANGE"
#           ^^
echo "Random number less than $RANGE  ---  $number"

echo



#  If you need a random integer greater than a lower bound,
#+ then set up a test to discard all numbers below that.

FLOOR=200

number=0   #initialize
while [ "$number" -le $FLOOR ]
do
  number=$RANDOM
done
echo "Random number greater than $FLOOR ---  $number"
echo

   # Let's examine a simple alternative to the above loop, namely
   #       let "number = $RANDOM + $FLOOR"
   # That would eliminate the while-loop and run faster.
   # But, there might be a problem with that. What is it?



# Combine above two techniques to retrieve random number between two limits.
number=0   #initialize
while [ "$number" -le $FLOOR ]
do
  number=$RANDOM
  let "number %= $RANGE"  # Scales $number down within $RANGE.
done
echo "Random number between $FLOOR and $RANGE ---  $number"
echo



# Generate binary choice, that is, "true" or "false" value.
BINARY=2
T=1
number=$RANDOM

let "number %= $BINARY"
#  Note that    let "number >>= 14"    gives a better random distribution
#+ (right shifts out everything except last binary digit).
if [ "$number" -eq $T ]
then
  echo "TRUE"
else
  echo "FALSE"
fi  

echo


# Generate a toss of the dice.
SPOTS=6   # Modulo 6 gives range 0 - 5.
          # Incrementing by 1 gives desired range of 1 - 6.
          # Thanks, Paulo Marcel Coelho Aragao, for the simplification.
die1=0
die2=0
# Would it be better to just set SPOTS=7 and not add 1? Why or why not?

# Tosses each die separately, and so gives correct odds.

    let "die1 = $RANDOM % $SPOTS +1" # Roll first one.
    let "die2 = $RANDOM % $SPOTS +1" # Roll second one.
    #  Which arithmetic operation, above, has greater precedence --
    #+ modulo (%) or addition (+)?


let "throw = $die1 + $die2"
echo "Throw of the dice = $throw"
echo


exit 0
```

###### Example 9-12. Picking a random card from a deck

```bash
#!/bin/bash
# pick-card.sh

# This is an example of choosing random elements of an array.


# Pick a card, any card.

Suites="Clubs
Diamonds
Hearts
Spades"

Denominations="2
3
4
5
6
7
8
9
10
Jack
Queen
King
Ace"

# Note variables spread over multiple lines.


suite=($Suites)                # Read into array variable.
denomination=($Denominations)

num_suites=${#suite[*]}        # Count how many elements.
num_denominations=${#denomination[*]}

echo -n "${denomination[$((RANDOM%num_denominations))]} of "
echo ${suite[$((RANDOM%num_suites))]}


# $bozo sh pick-cards.sh
# Jack of Clubs


# Thank you, "jipe," for pointing out this use of $RANDOM.
exit 0
```

###### Example 9-13. Brownian Motion Simulation

```bash
#!/bin/bash
# brownian.sh
# Author: Mendel Cooper
# Reldate: 10/26/07
# License: GPL3

#  ----------------------------------------------------------------
#  This script models Brownian motion:
#+ the random wanderings of tiny particles in a fluid,
#+ as they are buffeted by random currents and collisions.
#+ This is colloquially known as the "Drunkard's Walk."

#  It can also be considered as a stripped-down simulation of a
#+ Galton Board, a slanted board with a pattern of pegs,
#+ down which rolls a succession of marbles, one at a time.
#+ At the bottom is a row of slots or catch basins in which
#+ the marbles come to rest at the end of their journey.
#  Think of it as a kind of bare-bones Pachinko game.
#  As you see by running the script,
#+ most of the marbles cluster around the center slot.
#+ This is consistent with the expected binomial distribution.
#  As a Galton Board simulation, the script
#+ disregards such parameters as
#+ board tilt-angle, rolling friction of the marbles,
#+ angles of impact, and elasticity of the pegs.
#  To what extent does this affect the accuracy of the simulation?
#  ----------------------------------------------------------------

PASSES=500            #  Number of particle interactions / marbles.
ROWS=10               #  Number of "collisions" (or horiz. peg rows).
RANGE=3               #  0 - 2 output range from $RANDOM.
POS=0                 #  Left/right position.
RANDOM=$$             #  Seeds the random number generator from PID
                      #+ of script.

declare -a Slots      # Array holding cumulative results of passes.
NUMSLOTS=21           # Number of slots at bottom of board.


Initialize_Slots () { # Zero out all elements of the array.
for i in $( seq $NUMSLOTS )
do
  Slots[$i]=0
done

echo                  # Blank line at beginning of run.
  }


Show_Slots () {
echo; echo
echo -n " "
for i in $( seq $NUMSLOTS )   # Pretty-print array elements.
do
  printf "%3d" ${Slots[$i]}   # Allot three spaces per result.
done

echo # Row of slots:
echo " |__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|"
echo "                                ||"
echo #  Note that if the count within any particular slot exceeds 99,
     #+ it messes up the display.
     #  Running only(!) 500 passes usually avoids this.
  }


Move () {              # Move one unit right / left, or stay put.
  Move=$RANDOM         # How random is $RANDOM? Well, let's see ...
  let "Move %= RANGE"  # Normalize into range of 0 - 2.
  case "$Move" in
    0 ) ;;                   # Do nothing, i.e., stay in place.
    1 ) ((POS--));;          # Left.
    2 ) ((POS++));;          # Right.
    * ) echo -n "Error ";;   # Anomaly! (Should never occur.)
  esac
  }


Play () {                    # Single pass (inner loop).
i=0
while [ "$i" -lt "$ROWS" ]   # One event per row.
do
  Move
  ((i++));
done

SHIFT=11                     # Why 11, and not 10?
let "POS += $SHIFT"          # Shift "zero position" to center.
(( Slots[$POS]++ ))          # DEBUG: echo $POS

# echo -n "$POS "

  }


Run () {                     # Outer loop.
p=0
while [ "$p" -lt "$PASSES" ]
do
  Play
  (( p++ ))
  POS=0                      # Reset to zero. Why?
done
  }


# --------------
# main ()
Initialize_Slots
Run
Show_Slots
# --------------

exit $?

#  Exercises:
#  ---------
#  1) Show the results in a vertical bar graph, or as an alternative,
#+    a scattergram.
#  2) Alter the script to use /dev/urandom instead of $RANDOM.
#     Will this make the results more random?
#  3) Provide some sort of "animation" or graphic output
#     for each marble played.
```

_Jipe_ points out a set of techniques for generating random numbers within a range.

```bash
#  Generate random number between 6 and 30.
   rnumber=$((RANDOM%25+6))	

#  Generate random number in the same 6 - 30 range,
#+ but the number must be evenly divisible by 3.
   rnumber=$(((RANDOM%30/3+1)*3))

#  Note that this will not work all the time.
#  It fails if $RANDOM%30 returns 0.

#  Frank Wang suggests the following alternative:
   rnumber=$(( RANDOM%27/3*3+6 ))
```

_Bill Gradwohl_ came up with an improved formula that works for positive numbers.

```bash
rnumber=$(((RANDOM%(max-min+divisibleBy))/divisibleBy*divisibleBy+min))
```

Here Bill presents a versatile function that returns a random number between two specified values.

###### Example 9-14. Random between values

```bash
#!/bin/bash
# random-between.sh
# Random number between two specified values. 
# Script by Bill Gradwohl, with minor modifications by the document author.
# Corrections in lines 187 and 189 by Anthony Le Clezio.
# Used with permission.


randomBetween() {
   #  Generates a positive or negative random number
   #+ between $min and $max
   #+ and divisible by $divisibleBy.
   #  Gives a "reasonably random" distribution of return values.
   #
   #  Bill Gradwohl - Oct 1, 2003

   syntax() {
   # Function embedded within function.
      echo
      echo    "Syntax: randomBetween [min] [max] [multiple]"
      echo
      echo -n "Expects up to 3 passed parameters, "
      echo    "but all are completely optional."
      echo    "min is the minimum value"
      echo    "max is the maximum value"
      echo -n "multiple specifies that the answer must be "
      echo     "a multiple of this value."
      echo    "    i.e. answer must be evenly divisible by this number."
      echo    
      echo    "If any value is missing, defaults area supplied as: 0 32767 1"
      echo -n "Successful completion returns 0, "
      echo     "unsuccessful completion returns"
      echo    "function syntax and 1."
      echo -n "The answer is returned in the global variable "
      echo    "randomBetweenAnswer"
      echo -n "Negative values for any passed parameter are "
      echo    "handled correctly."
   }

   local min=${1:-0}
   local max=${2:-32767}
   local divisibleBy=${3:-1}
   # Default values assigned, in case parameters not passed to function.

   local x
   local spread

   # Let's make sure the divisibleBy value is positive.
   [ ${divisibleBy} -lt 0 ] && divisibleBy=$((0-divisibleBy))

   # Sanity check.
   if [ $# -gt 3 -o ${divisibleBy} -eq 0 -o  ${min} -eq ${max} ]; then 
      syntax
      return 1
   fi

   # See if the min and max are reversed.
   if [ ${min} -gt ${max} ]; then
      # Swap them.
      x=${min}
      min=${max}
      max=${x}
   fi

   #  If min is itself not evenly divisible by $divisibleBy,
   #+ then fix the min to be within range.
   if [ $((min/divisibleBy*divisibleBy)) -ne ${min} ]; then 
      if [ ${min} -lt 0 ]; then
         min=$((min/divisibleBy*divisibleBy))
      else
         min=$((((min/divisibleBy)+1)*divisibleBy))
      fi
   fi

   #  If max is itself not evenly divisible by $divisibleBy,
   #+ then fix the max to be within range.
   if [ $((max/divisibleBy*divisibleBy)) -ne ${max} ]; then 
      if [ ${max} -lt 0 ]; then
         max=$((((max/divisibleBy)-1)*divisibleBy))
      else
         max=$((max/divisibleBy*divisibleBy))
      fi
   fi

   #  ---------------------------------------------------------------------
   #  Now, to do the real work.

   #  Note that to get a proper distribution for the end points,
   #+ the range of random values has to be allowed to go between
   #+ 0 and abs(max-min)+divisibleBy, not just abs(max-min)+1.

   #  The slight increase will produce the proper distribution for the
   #+ end points.

   #  Changing the formula to use abs(max-min)+1 will still produce
   #+ correct answers, but the randomness of those answers is faulty in
   #+ that the number of times the end points ($min and $max) are returned
   #+ is considerably lower than when the correct formula is used.
   #  ---------------------------------------------------------------------

   spread=$((max-min))
   #  Omair Eshkenazi points out that this test is unnecessary,
   #+ since max and min have already been switched around.
   [ ${spread} -lt 0 ] && spread=$((0-spread))
   let spread+=divisibleBy
   randomBetweenAnswer=$(((RANDOM%spread)/divisibleBy*divisibleBy+min))   

   return 0

   #  However, Paulo Marcel Coelho Aragao points out that
   #+ when $max and $min are not divisible by $divisibleBy,
   #+ the formula fails.
   #
   #  He suggests instead the following formula:
   #    rnumber = $(((RANDOM%(max-min+1)+min)/divisibleBy*divisibleBy))

}

# Let's test the function.
min=-14
max=20
divisibleBy=3


#  Generate an array of expected answers and check to make sure we get
#+ at least one of each answer if we loop long enough.

declare -a answer
minimum=${min}
maximum=${max}
   if [ $((minimum/divisibleBy*divisibleBy)) -ne ${minimum} ]; then 
      if [ ${minimum} -lt 0 ]; then
         minimum=$((minimum/divisibleBy*divisibleBy))
      else
         minimum=$((((minimum/divisibleBy)+1)*divisibleBy))
      fi
   fi


   #  If max is itself not evenly divisible by $divisibleBy,
   #+ then fix the max to be within range.

   if [ $((maximum/divisibleBy*divisibleBy)) -ne ${maximum} ]; then 
      if [ ${maximum} -lt 0 ]; then
         maximum=$((((maximum/divisibleBy)-1)*divisibleBy))
      else
         maximum=$((maximum/divisibleBy*divisibleBy))
      fi
   fi


#  We need to generate only positive array subscripts,
#+ so we need a displacement that that will guarantee
#+ positive results.

disp=$((0-minimum))
for ((i=${minimum}; i<=${maximum}; i+=divisibleBy)); do
   answer[i+disp]=0
done


# Now loop a large number of times to see what we get.
loopIt=1000   #  The script author suggests 100000,
              #+ but that takes a good long while.

for ((i=0; i<${loopIt}; ++i)); do

   #  Note that we are specifying min and max in reversed order here to
   #+ make the function correct for this case.

   randomBetween ${max} ${min} ${divisibleBy}

   # Report an error if an answer is unexpected.
   [ ${randomBetweenAnswer} -lt ${min} -o ${randomBetweenAnswer} -gt ${max} ] \
   && echo MIN or MAX error - ${randomBetweenAnswer}!
   [ $((randomBetweenAnswer%${divisibleBy})) -ne 0 ] \
   && echo DIVISIBLE BY error - ${randomBetweenAnswer}!

   # Store the answer away statistically.
   answer[randomBetweenAnswer+disp]=$((answer[randomBetweenAnswer+disp]+1))
done



# Let's check the results

for ((i=${minimum}; i<=${maximum}; i+=divisibleBy)); do
   [ ${answer[i+disp]} -eq 0 ] \
   && echo "We never got an answer of $i." \
   || echo "${i} occurred ${answer[i+disp]} times."
done


exit 0
```

Just how random is $RANDOM? The best way to test this is to write a script that tracks the distribution of "random" numbers generated by $RANDOM. Let's roll a $RANDOM die a few times . . .

###### Example 9-15. Rolling a single die with RANDOM

```bash
#!/bin/bash
# How random is RANDOM?

RANDOM=$$       # Reseed the random number generator using script process ID.

PIPS=6          # A die has 6 pips.
MAXTHROWS=600   # Increase this if you have nothing better to do with your time.
throw=0         # Number of times the dice have been cast.

ones=0          #  Must initialize counts to zero,
twos=0          #+ since an uninitialized variable is null, NOT zero.
threes=0
fours=0
fives=0
sixes=0

print_result ()
{
echo
echo "ones =   $ones"
echo "twos =   $twos"
echo "threes = $threes"
echo "fours =  $fours"
echo "fives =  $fives"
echo "sixes =  $sixes"
echo
}

update_count()
{
case "$1" in
  0) ((ones++));;   # Since a die has no "zero", this corresponds to 1.
  1) ((twos++));;   # And this to 2.
  2) ((threes++));; # And so forth.
  3) ((fours++));;
  4) ((fives++));;
  5) ((sixes++));;
esac
}

echo


while [ "$throw" -lt "$MAXTHROWS" ]
do
  let "die1 = RANDOM % $PIPS"
  update_count $die1
  let "throw += 1"
done  

print_result

exit $?

#  The scores should distribute evenly, assuming RANDOM is random.
#  With $MAXTHROWS at 600, all should cluster around 100,
#+ plus-or-minus 20 or so.
#
#  Keep in mind that RANDOM is a ***pseudorandom*** generator,
#+ and not a spectacularly good one at that.

#  Randomness is a deep and complex subject.
#  Sufficiently long "random" sequences may exhibit
#+ chaotic and other "non-random" behavior.

# Exercise (easy):
# ---------------
# Rewrite this script to flip a coin 1000 times.
# Choices are "HEADS" and "TAILS."
```

As we have seen in the last example, it is best to _reseed_ the _RANDOM_ generator each time it is invoked. Using the same seed for _RANDOM_ repeats the same series of numbers. [^8] (This mirrors the behavior of the _random()_ function in _C_.)

###### Example 9-16. Reseeding RANDOM

```bash
#!/bin/bash
# seeding-random.sh: Seeding the RANDOM variable.
# v 1.1, reldate 09 Feb 2013

MAXCOUNT=25       # How many numbers to generate.
SEED=

random_numbers ()
{
local count=0
local number

while [ "$count" -lt "$MAXCOUNT" ]
do
  number=$RANDOM
  echo -n "$number "
  let "count++"
done  
}

echo; echo

SEED=1
RANDOM=$SEED      # Setting RANDOM seeds the random number generator.
echo "Random seed = $SEED"
random_numbers


RANDOM=$SEED      # Same seed for RANDOM . . .
echo; echo "Again, with same random seed ..."
echo "Random seed = $SEED"
random_numbers    # . . . reproduces the exact same number series.
                  #
                  # When is it useful to duplicate a "random" series?

echo; echo

SEED=2
RANDOM=$SEED      # Trying again, but with a different seed . . .
echo "Random seed = $SEED"
random_numbers    # . . . gives a different number series.

echo; echo

# RANDOM=$$  seeds RANDOM from process id of script.
# It is also possible to seed RANDOM from 'time' or 'date' commands.

# Getting fancy...
SEED=$(head -1 /dev/urandom | od -N 1 | awk '{ print $2 }'| sed s/^0*//)
#  Pseudo-random output fetched
#+ from /dev/urandom (system pseudo-random device-file),
#+ then converted to line of printable (octal) numbers by "od",
#+ then "awk" retrieves just one number for SEED,
#+ finally "sed" removes any leading zeros.
RANDOM=$SEED
echo "Random seed = $SEED"
random_numbers

echo; echo

exit 0
```

> [!note]
> The /dev/urandom pseudo-device file provides a method of generating much more "random" pseudorandom numbers than the $RANDOM variable. **dd if=/dev/urandom of=targetfile bs=1 count=XX** creates a file of well-scattered pseudorandom numbers. However, assigning these numbers to a variable in a script requires a workaround, such as filtering through [[miscellaneous-commands#^ODREF|od]] (as in above example, [[text-processing-commands#^RND|Example 16-14]], and [[contributed-scripts#^INSERTIONSORT|Example A-36]]), or even piping to [[file-and-archiving-commands#^MD5SUMREF|md5sum]] (see [[colorizing-scripts#^HORSERACE|Example 36-16]]).
>
> There are also other ways to generate pseudorandom numbers in a script. **Awk** provides a convenient means of doing this.
>
> **Example 9-17. Pseudorandom numbers, using [[awk#^AWKREF|awk]]**
>
> ```bash
> #!/bin/bash
> #  random2.sh: Returns a pseudorandom number in the range 0 - 1,
> #+ to 6 decimal places. For example: 0.822725
> #  Uses the awk rand() function.
> 
> AWKSCRIPT=' { srand(); print rand() } '
> #           Command(s)/parameters passed to awk
> # Note that srand() reseeds awk's random number generator.
> 
> 
> echo -n "Random number between 0 and 1 = "
> 
> echo | awk "$AWKSCRIPT"
> # What happens if you leave out the 'echo'?
> 
> exit 0
> 
> 
> # Exercises:
> # ---------
> 
> # 1) Using a loop construct, print out 10 different random numbers.
> #      (Hint: you must reseed the srand() function with a different seed
> #+     in each pass through the loop. What happens if you omit this?)
> 
> # 2) Using an integer multiplier as a scaling factor, generate random numbers 
> #+   in the range of 10 to 100.
> 
> # 3) Same as exercise #2, above, but generate random integers this time.
> ```
>
> The [[time-date-commands#^DATEREF|date]] command also lends itself to [[time-date-commands#^DATERANDREF|generating pseudorandom integer sequences]].

[^1]: A _stack register_ is a set of consecutive memory locations, such that the values stored (_pushed_) are retrieved (_popped_) in _reverse_ order. The last value stored is the first retrieved. This is sometimes called a _LIFO_ (_last-in-first-out_) or _pushdown_ stack.

[^2]: The PID of the currently running script is `$$`, of course.

[^3]: Somewhat analogous to [[local-variables#^RECURSIONREF|recursion]], in this context _nesting_ refers to a pattern embedded within a larger pattern. One of the definitions of _nest_, according to the 1913 edition of _Webster's Dictionary_, illustrates this beautifully: "_A collection of boxes, cases, or the like, of graduated size, each put within the one next larger._"

[^4]: The words "argument" and "parameter" are often used interchangeably. In the context of this document, they have the same precise meaning: _a variable passed to a script or function._

[^5]: Within a script, inside a subshell, `$$` [[another-look-at-variables#^BASHPIDREF|returns the PID of the script]], not the subshell.

[^6]: In this context, _typing_ a variable means to classify it and restrict its properties. For example, a variable _declared_ or _typed_ as an integer is no longer available for [[reference-cards#^STRINGOPSTAB|string operations]].

    ```bash
    declare -i intvar

    intvar=23
    echo "$intvar"   # 23
    intvar=stringval
    echo "$intvar"   # 0
    ```

[^7]: True "randomness," insofar as it exists at all, can only be found in certain incompletely understood natural phenomena, such as radioactive decay. Computers only _simulate_ randomness, and computer-generated sequences of "random" numbers are therefore referred to as _pseudorandom_.

[^8]: The _seed_ of a computer-generated pseudorandom number series can be considered an identification label. For example, think of the pseudorandom series with a seed of _23_ as _Series #23_.

    A property of a pseurandom number series is the length of the cycle before it starts repeating itself. A good pseurandom generator will produce series with very long cycles.
