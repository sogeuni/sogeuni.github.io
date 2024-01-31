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

See [[Example 21-1|Example 21-1]] for usage.

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

See also [[Example A-50|Example A-50]].

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

Home directory of the user, usually /home/username (see [[Example 10-7|Example 10-7]])

$HOSTNAME

The [[system-and-administrative-commands#^HNAMEREF|hostname]] command assigns the system host name at bootup in an init script. However, the gethostname() function sets the Bash internal variable $HOSTNAME. See also [[Example 10-7|Example 10-7]].

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

`$IFS` defaults to [[special-characters#Whitespace|whitespace]] (space, tab, and newline), but may be changed, for example, to parse a comma-separated data file. Note that [[another-look-at-variables#^APPREF|$*]] uses the first character held in $IFS. See [[Example 5-1|Example 5-1]].

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
> ![[Example 9-1|Example 9-1]]
>
> (Many thanks, Stéphane Chazelas, for clarification and above examples.)
>
> See also [[Example 16-41|Example 16-41]], [[Example 11-8|Example 11-8]], and [[Example 19-14|Example 19-14]] for instructive examples of using $IFS.

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
> Chet Ramey attributes the above output to the behavior of [[external-filters-programs-and-commands#^LSREF|ls]]. If _ls_ writes to a _pipe_ whose output is not read, then _SIGPIPE_ kills it, and its [[exit-and-exit-status#^EXITSTATUSREF|exit status]] is 141. Otherwise its exit status is 0, as expected. This likewise is the case for [[external-filters-programs-and-commands#^TRREF|tr]].

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

The tertiary prompt, displayed in a [[loops-and-branches#^SELECTREF|select]] loop (see [[Example 11-30|Example 11-30]]).

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

The default value when a variable is not supplied to [[internal-commands-and-builtins#^READREF|read]]. Also applicable to [[loops-and-branches#^SELECTREF|select]] menus, but only supplies the item number of the variable chosen, not the value of the variable itself.

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

There are other, more complex, ways of implementing timed input in a script. One alternative is to set up a timing loop to signal the script when it times out. This also requires a signal handling routine to [[debugging#^TRAPREF1|trap]] (see [[Example 32-5|Example 32-5]]) the interrupt generated by the timing loop (whew!).

![[Example 9-2|Example 9-2]]

An alternative is using [[system-and-administrative-commands#^STTYREF|stty]].

![[Example 9-3|Example 9-3]]

Perhaps the simplest method is using the -t option to [[internal-commands-and-builtins#^READREF|read]].

![[Example 9-4|Example 9-4]]

$UID

User ID number

Current user's user identification number, as recorded in [[important-files#^DATAFILESREF1|/etc/passwd]]

This is the current user's real id, even if she has temporarily assumed another identity through [[system-and-administrative-commands#^SUREF|su]]. $UID is a readonly variable, not subject to change from the command line or within a script, and is the counterpart to the [[system-and-administrative-commands#^IDREF|id]] builtin.

![[Example 9-5|Example 9-5]]

See also [[Example 2-3|Example 2-3]].

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

Positional parameters, passed from command line to script, passed to a function, or [[internal-commands-and-builtins#^SETREF|set]] to a variable (see [[Example 4-5|Example 4-5]] and [[Example 15-16|Example 15-16]])

$#

Number of command-line arguments [[shell-wrappers#^EX4|^4] or positional parameters (see [Example 36-2]])

$*

All of the positional parameters, seen as a single word

> [!note] "$*" must be quoted.

$@

Same as $*, but each parameter is a quoted string, that is, the parameters are passed on intact, without interpretation or expansion. This means, among other things, that each parameter in the argument list is seen as a separate word.

> [!note] Of course, "$@" should be quoted.

![[Example 9-6|Example 9-6]]

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

The `$@` special parameter finds use as a tool for filtering input into shell scripts. The **`cat "$@"`** construction accepts input to a script either from stdin or from files given as parameters to the script. See [[Example 16-24|Example 16-24]] and [[Example 16-25|Example 16-25]].

> [!caution] The `$*` and `$@` parameters sometimes display inconsistent and puzzling behavior, depending on the setting of [[another-look-at-variables#^IFSREF|$IFS]].

![[Example 9-7|Example 9-7]]

> [!note] The **$@** and **$*** parameters differ only when between double quotes.

![[Example 9-8|Example 9-8]]

**Other Special Parameters**

$-

Flags passed to script (using [[internal-commands-and-builtins#^SETREF|set]]). See [[Example 15-16|Example 15-16]].

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

![[Example 9-9|Example 9-9]]

$?

[[exit-and-exit-status#^EXITSTATUSREF|Exit status]] of a command, [[functions|function]], or the script itself (see [[Example 24-7|Example 24-7]])

`$$`

Process ID (_PID_) of the script itself. [[debugging#^ONLINE|^5] The $ variable often finds use in scripts to construct "unique" temp file names (see [Example 32-6]], [[Example 16-31|Example 16-31]], and [[Example 15-27|Example 15-27]]). This is usually simpler than invoking [[file-and-archiving-commands#^MKTEMPREF|mktemp]].

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

Certain arithmetic operations are permitted for declared integer variables without the need for [[external-filters-programs-and-commands#^EXPRREF|expr]] or [[internal-commands-and-builtins#^LETREF|let]].

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

![[Example 9-10|Example 9-10]]

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

![[Example 9-11|Example 9-11]]

![[Example 9-12|Example 9-12]]

![[Example 9-13|Example 9-13]]

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

![[Example 9-14|Example 9-14]]

Just how random is $RANDOM? The best way to test this is to write a script that tracks the distribution of "random" numbers generated by $RANDOM. Let's roll a $RANDOM die a few times . . .

![[Example 9-15|Example 9-15]]

As we have seen in the last example, it is best to _reseed_ the _RANDOM_ generator each time it is invoked. Using the same seed for _RANDOM_ repeats the same series of numbers. [^8] (This mirrors the behavior of the _random()_ function in _C_.)

![[Example 9-16|Example 9-16]]

> [!note]
> The /dev/urandom pseudo-device file provides a method of generating much more "random" pseudorandom numbers than the $RANDOM variable. **dd if=/dev/urandom of=targetfile bs=1 count=XX** creates a file of well-scattered pseudorandom numbers. However, assigning these numbers to a variable in a script requires a workaround, such as filtering through [[miscellaneous-commands#^ODREF|od]] (as in above example, [[Example 16-14|Example 16-14]], and [[Example A-36|Example A-36]]), or even piping to [[file-and-archiving-commands#^MD5SUMREF|md5sum]] (see [[Example 36-16|Example 36-16]]).
>
> There are also other ways to generate pseudorandom numbers in a script. **Awk** provides a convenient means of doing this.
>
> ![[Example 9-17|Example 9-17]]
>
> The [[external-filters-programs-and-commands#^DATEREF|date]] command also lends itself to [[external-filters-programs-and-commands#^DATERANDREF|generating pseudorandom integer sequences]].

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
