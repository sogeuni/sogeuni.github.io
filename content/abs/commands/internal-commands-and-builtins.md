---
title: 15. Internal Commands and Builtins
---

A _builtin_ is a **command** contained within the Bash tool set, literally _built in_. This is either for performance reasons -- builtins execute faster than external commands, which usually require _forking off_ [^1] a separate process -- or because a particular builtin needs direct access to the shell internals.

> When a command or the shell itself initiates (or _spawns_) a new subprocess to carry out a task, this is called _forking_. This new process is the _child_, and the process that _forked_ it off is the _parent_. While the _child process_ is doing its work, the _parent process_ is still executing.
>
> Note that while a _parent process_ gets the _process ID_ of the _child process_, and can thus pass arguments to it, _the reverse is not true_. [[gotchas#^PARCHILDPROBREF|This can create problems that are subtle and hard to track down.]]
>
> ![[Example 15-1|Example 15-1]]
>
> Generally, a Bash _builtin_ does not fork a subprocess when it executes within a script. An external system command or filter in a script usually _will_ fork a subprocess.

A builtin may be a synonym to a system command of the same name, but Bash reimplements it internally. For example, the Bash **echo** command is not the same as /bin/echo, although their behavior is almost identical.

```bash
#!/bin/bash

echo "This line uses the \"echo\" builtin."
/bin/echo "This line uses the /bin/echo system command."
```

A _keyword_ is a _reserved_ word, token or operator. Keywords have a special meaning to the shell, and indeed are the building blocks of the shell's syntax. As examples, _for_, _while_, _do_, and _!_ are keywords. Similar to a [[internal-commands-and-builtins|builtin]], a keyword is hard-coded into Bash, but unlike a _builtin_, a keyword is not in itself a command, but _a subunit of a command construct_. [^2] ^keywordref

**I/O**

**echo**

prints (to stdout) an expression or variable (see [[Example 4-1|Example 4-1]]).

```bash
echo Hello
echo $a
```

An **echo** requires the -e option to print escaped characters. See [[Example 5-2|Example 5-2]].

Normally, each **echo** command prints a terminal newline, but the -n option suppresses this.

> [!note]
> An **echo** can be used to feed a sequence of commands down a pipe.
>
> ```bash
> if echo "$VAR" | grep -q txt   # if [[ $VAR = *txt* ]]
> then
>   echo "$VAR contains the substring sequence \"txt\""
> fi
> ```

> [!note]
> An **echo**, in combination with [[command-substitution#^COMMANDSUBREF|command substitution]] can set a variable.
>
> **a=`echo "HELLO" \| tr A-Z a-z`**
>
> See also [[Example 16-22|Example 16-22]], [[Example 16-3|Example 16-3]], [[Example 16-47|Example 16-47]], and [[Example 16-48|Example 16-48]].|

Be aware that **echo `command`** deletes any linefeeds that the output of _command_ generates.

The [[another-look-at-variables#^IFSREF|$IFS]] (internal field separator) variable normally contains \n (linefeed) as one of its set of [[special-characters#Whitespace|whitespace]] characters. Bash therefore splits the output of _command_ at linefeeds into arguments to **echo**. Then **echo** outputs these arguments, separated by spaces.

```bash

bash$ ls -l /usr/share/apps/kjezz/sounds
-rw-r--r--    1 root     root         1407 Nov  7  2000 reflect.au
 -rw-r--r--    1 root     root          362 Nov  7  2000 seconds.au




bash$ echo `ls -l /usr/share/apps/kjezz/sounds`
total 40 -rw-r--r-- 1 root root 716 Nov 7 2000 reflect.au -rw-r--r-- 1 root root ...
	      
```

So, how can we embed a linefeed within an [[internal-commands-and-builtins#^ECHOREF|echoed]] character string?

```bash
# Embedding a linefeed?
echo "Why doesn't this string \n split on two lines?"
# Doesn't split.

# Let's try something else.

echo
	     
echo $"A line of text containing
a linefeed."
# Prints as two distinct lines (embedded linefeed).
# But, is the "$" variable prefix really necessary?

echo

echo "This string splits
on two lines."
# No, the "$" is not needed.

echo
echo "---------------"
echo

echo -n $"Another line of text containing
a linefeed."
# Prints as two distinct lines (embedded linefeed).
# Even the -n option fails to suppress the linefeed here.

echo
echo
echo "---------------"
echo
echo

# However, the following doesn't work as expected.
# Why not? Hint: Assignment to a variable.
string1=$"Yet another line of text containing
a linefeed (maybe)."

echo $string1
# Yet another line of text containing a linefeed (maybe).
#                                    ^
# Linefeed becomes a space.

# Thanks, Steve Parker, for pointing this out.
```

> [!note]
> This command is a shell builtin, and not the same as /bin/echo, although its behavior is similar.

```bash
bash$ type -a echo
echo is a shell builtin
 echo is /bin/echo
	      
```

**printf**

The **printf**, formatted print, command is an enhanced **echo**. It is a limited variant of the _C_ language printf() library function, and its syntax is somewhat different.

**printf** _format-string_... _parameter_...

This is the Bash _builtin_ version of the /bin/printf or /usr/bin/printf command. See the **printf** [[external-filters-programs-and-commands#^MANREF|manpage]] (of the system command) for in-depth coverage.

> [!caution]
> Older versions of Bash may not support **printf**.

![[Example 15-2|Example 15-2]]

Formatting error messages is a useful application of **printf**

```bash
E_BADDIR=85

var=nonexistent_directory

error()
{
  printf "$@" >&2
  # Formats positional params passed, and sends them to stderr.
  echo
  exit $E_BADDIR
}

cd $var || error $"Can't cd to %s." "$var"

# Thanks, S.C.
```

See also [[Example 36-17|Example 36-17]].

**read**

"Reads" the value of a variable from stdin, that is, interactively fetches input from the keyboard. The -a option lets **read** get array variables (see [[Example 27-6|Example 27-6]]).

![[Example 15-3|Example 15-3]]

A **read** without an associated variable assigns its input to the dedicated variable [[another-look-at-variables#^REPLYREF|$REPLY]].

![[Example 15-4|Example 15-4]]

Normally, inputting a **\** suppresses a newline during input to a **read**. The -r option causes an inputted **\** to be interpreted literally.

![[Example 15-5|Example 15-5]]

The **read** command has some interesting options that permit echoing a prompt and even reading keystrokes without hitting **ENTER**.

```bash
# Read a keypress without hitting ENTER.

read -s -n1 -p "Hit a key " keypress
echo; echo "Keypress was "\"$keypress\""."

# -s option means do not echo input.
# -n N option means accept only N characters of input.
# -p option means echo the following prompt before reading input.

# Using these options is tricky, since they need to be in the correct order.
```

The -n option to **read** also allows detection of the **arrow keys** and certain of the other unusual keys.

![[Example 15-6|Example 15-6]]

> [!note]
> The -n option to **read** will not detect the **ENTER** (newline) key.

The -t option to **read** permits timed input (see [[Example 9-4|Example 9-4]] and [[Example A-41|Example A-41]]).

The -u option takes the [[io-redirection#^FDREF|file descriptor]] of the target file.

The **read** command may also "read" its variable value from a file [[io-redirection|redirected]] to stdin. If the file contains more than one line, only the first line is assigned to the variable. If **read** has more than one parameter, then each of these variables gets assigned a successive [[special-characters#Whitespace|whitespace-delineated]] string. Caution!

![[Example 15-7|Example 15-7]]

> [!note]
> [[special-characters#^PIPEREF|Piping]] output to a _read_, using [[internal-commands-and-builtins#^ECHOREF|echo]] to set variables [[gotchas#^BADREAD0|will fail]].
>
> Yet, piping the output of [[external-filters-programs-and-commands#^CATREF|cat]] _seems_ to work.
>
> ```bash
> cat file1 file2 |
> while read line
> do
> echo $line
> done
> ```
>
> However, as Bjön Eriksson shows:
>
> ![[Example 15-8|Example 15-8]]
>
> The _gendiff_ script, usually found in /usr/bin on many Linux distros, pipes the output of [[external-filters-programs-and-commands#^FINDREF|find]] to a _while read_ construct.
>
> ```bash
> find $1 \( -name "*$2" -o -name ".*$2" \) -print |
> while read f; do
> . . .
> ```

> [!tip]
> It is possible to _paste_ text into the input field of a _read_ (but _not_ multiple lines!). See [[Example A-38|Example A-38]].

**Filesystem**

**cd**

The familiar **cd** change directory command finds use in scripts where execution of a command requires being in a specified directory.

```bash
(cd /source/directory && tar cf - . ) | (cd /dest/directory && tar xpvf -)
```

[from the [[special-characters#^COXEX|previously cited]] example by Alan Cox]

The -P (physical) option to **cd** causes it to ignore symbolic links.

**cd -** changes to [[another-look-at-variables#^OLDPWD|$OLDPWD]], the previous working directory.

> [!caution]
> The **cd** command does not function as expected when presented with two forward slashes.
>
> ```bash
> bash$ cd //
> bash$ pwd
> //
> 	
> ```
>
> The output should, of course, be /. This is a problem both from the command-line and in a script.|

**pwd**

Print Working Directory. This gives the user's (or script's) current directory (see [[Example 15-9|Example 15-9]]). The effect is identical to reading the value of the builtin variable [[another-look-at-variables#^PWDREF|$PWD]].

**pushd**, **popd**, **dirs**

This command set is a mechanism for bookmarking working directories, a means of moving back and forth through directories in an orderly manner. A pushdown [[another-look-at-variables#^STACKDEFREF|stack]] is used to keep track of directory names. Options allow various manipulations of the directory stack.

**pushd dir-name** pushes the path _dir-name_ onto the directory stack (to the _top_ of the stack) and simultaneously changes the current working directory to _dir-name_

**popd** removes (pops) the top directory path name off the directory stack and simultaneously changes the current working directory to the directory now at the _top_ of the stack.

**dirs** lists the contents of the directory stack (compare this with the [[another-look-at-variables#^DIRSTACKREF|$DIRSTACK]] variable). A successful **pushd** or **popd** will automatically invoke **dirs**.

Scripts that require various changes to the current working directory without hard-coding the directory name changes can make good use of these commands. Note that the implicit $DIRSTACK array variable, accessible from within a script, holds the contents of the directory stack.

![[Example 15-9|Example 15-9]]

**Variables**

**let**

The **let** command carries out _arithmetic_ operations on variables. [^3] In many cases, it functions as a less complex version of [[external-filters-programs-and-commands#^EXPRREF|expr]].

![[Example 15-10|Example 15-10]]

> [!caution]
> The _let_ command can, in certain contexts, return a surprising [[exit-and-exit-status#^EXITSTATUSREF|exit status]].
>
> ```bash
> # Evgeniy Ivanov points out:
> 
> var=0
> echo $?     # 0
>             # As expected.
> 
> let var++
> echo $?     # 1
>             # The command was successful, so why isn't $?=0 ???
>             # Anomaly!
> 
> let var++
> echo $?     # 0
>             # As expected.
> 
> 
> # Likewise . . .
> 
> let var=0
> echo $?     # 1
>             # The command was successful, so why isn't $?=0 ???
> 
> #  However, as Jeff Gorak points out,
> #+ this is part of the design spec for 'let' . . .
> # "If the last ARG evaluates to 0, let returns 1;
> #  let returns 0 otherwise." ['help let']
> ```


**eval**

**eval arg1 [arg2] ... [argN]**

Combines the arguments in an expression or list of expressions and _evaluates_ them. Any variables within the expression are expanded. The net result is to **convert a string into a command**.

> [!tip]
> The **eval** command can be used for code generation from the command-line or within a script.

```bash
bash$ command_string="ps ax"
bash$ process="ps ax"
bash$ eval "$command_string" | grep "$process"
26973 pts/3    R+     0:00 grep --color ps ax
 26974 pts/3    R+     0:00 ps ax
	      
```

Each invocation of _eval_ forces a re-_evaluation_ of its arguments.

```bash
a='$b'
b='$c'
c=d

echo $a             # $b
                    # First level.
eval echo $a        # $c
                    # Second level.
eval eval echo $a   # d
                    # Third level.

# Thank you, E. Choroba.
```

![[Example 15-11|Example 15-11]]

![[Example 15-12|Example 15-12]]

![[Example 15-13|Example 15-13]]

![[Example 15-14|Example 15-14]]

![[Example 15-15|Example 15-15]]

Here is another example of using _eval_ to _evaluate_ a complex expression, this one from an earlier version of YongYe's [Tetris game script](https://github.com/yongye/shell/blob/master/Tetris_Game.sh).

```bash
eval ${1}+=\"${x} ${y} \"
```

[[Example A-53|Example A-53]] uses _eval_ to convert [[arrays#^ARRAYREF|array]] elements into a command list.

The _eval_ command occurs in the older version of [[indirect-references#^IVRREF|indirect referencing]].

```bash
eval var=\$$var
```

> [!tip]
> The _eval_ command can be used to [[bashver3#^BRACEEXPREF3|parameterize _brace expansion_]].

> [!caution]
> The **eval** command can be risky, and normally should be avoided when there exists a reasonable alternative. An **eval $COMMANDS** executes the contents of _COMMANDS_, which may contain such unpleasant surprises as **rm -rf ***. Running an **eval** on unfamiliar code written by persons unknown is living dangerously.

**set**

The **set** command changes the value of internal script variables/options. One use for this is to toggle [[options#^OPTIONSREF|option flags]] which help determine the behavior of the script. Another application for it is to reset the [[another-look-at-variables#^POSPARAMREF|positional parameters]] that a script sees as the result of a command (**set `command`**). The script can then parse the [[special-characters#^FIELDREF|fields]] of the command output.

![[Example 15-16|Example 15-16]]

More fun with positional parameters.

![[Example 15-17|Example 15-17]]

Invoking **set** without any options or arguments simply lists all the [[othertypesv#^ENVREF|environmental]] and other variables that have been initialized.

```bash
bash$ set
AUTHORCOPY=/home/bozo/posts
 BASH=/bin/bash
 BASH_VERSION=$'2.05.8(1)-release'
 ...
 XAUTHORITY=/home/bozo/.Xauthority
 _=/etc/bashrc
 variable22=abc
 variable23=xzy
	      
```

Using **set** with the -- option explicitly assigns the contents of a variable to the positional parameters. If no variable follows the -- it _unsets_ the positional parameters.

![[Example 15-18|Example 15-18]]

See also [[Example 11-2|Example 11-2]] and [[Example 16-56|Example 16-56]].

**unset**

The **unset** command deletes a shell variable, effectively setting it to _null_. Note that this command does not affect positional parameters.

```bash
bash$ unset PATH

bash$ echo $PATH

bash$ 
```

![[Example 15-19|Example 15-19]]

> [!note]
> In most contexts, an _undeclared_ variable and one that has been _unset_ are equivalent. However, the [[parameter-substitution#^UNDDR|${parameter:-default}]] parameter substitution construct can distinguish between the two.

**export**

The **export** [^4] command makes available variables to all child processes of the running script or shell. One important use of the **export** command is in [[important-files#^FILESREF1|startup files]], to initialize and make accessible [[othertypesv#^ENVREF|environmental variables]] to subsequent user processes.

> [!caution]
> Unfortunately, [[gotchas#^PARCHILDPROBREF|there is no way to export variables back to the parent process]], to the process that called or invoked the script or shell.

![[Example 15-20|Example 15-20]]

> [!tip]
> It is possible to initialize and export variables in the same operation, as in **export var1=xxx**.
>
> However, as Greg Keraunen points out, in certain situations this may have a different effect than setting a variable, then exporting it.
>
> ```bash
> bash$ export var=(a b); echo ${var[0]}
> (a b)
> 
> 
> 
> bash$ var=(a b); export var; echo ${var[0]}
> a
> 	      
> ```

> [!note]
> A variable to be exported may require special treatment. See [[Example M-2|Example M-2]].

**declare**, **typeset**

The [[typing-variables.html|declare]] and [[typing-variables.html|typeset]] commands specify and/or restrict properties of variables.

**readonly**

Same as [[typing-variables.html|declare -r]], sets a variable as read-only, or, in effect, as a constant. Attempts to change the variable fail with an error message. This is the shell analog of the _C_ language **const** type qualifier.

**getopts**

This powerful tool parses command-line arguments passed to the script. This is the Bash analog of the [[miscellaneous-commands#^GETOPTY|getopt]] external command and the _getopt_ library function familiar to _C_ programmers. It permits passing and concatenating multiple options [^5] and associated arguments to a script (for example **scriptname -abc -e /usr/local**).

The **getopts** construct uses two implicit variables. $OPTIND is the argument pointer (_OPTion INDex_) and $OPTARG (_OPTion ARGument_) the (optional) argument attached to an option. A colon following the option name in the declaration tags that option as having an associated argument.

A **getopts** construct usually comes packaged in a [[loops-and-branches#^WHILELOOPREF|while loop]], which processes the options and arguments one at a time, then increments the implicit $OPTIND variable to point to the next.

> [!note]
>
> 1. The arguments passed from the command-line to the script must be preceded by a dash (-). It is the prefixed - that lets **getopts** recognize command-line arguments as _options_. In fact, **getopts** will not process arguments without the prefixed -, and will terminate option processing at the first argument encountered lacking them.
> 2. The **getopts** template differs slightly from the standard [[loops-and-branches#^WHILELOOPREF|while loop]], in that it lacks condition brackets.
> 3. The **getopts** construct is a highly functional replacement for the traditional [[miscellaneous-commands#^GETOPTY|getopt]] external command.

```bash
while getopts ":abcde:fg" Option
# Initial declaration.
# a, b, c, d, e, f, and g are the options (flags) expected.
# The : after option 'e' shows it will have an argument passed with it.
do
  case $Option in
    a ) # Do something with variable 'a'.
    b ) # Do something with variable 'b'.
    ...
    e)  # Do something with 'e', and also with $OPTARG,
        # which is the associated argument passed with option 'e'.
    ...
    g ) # Do something with variable 'g'.
  esac
done
shift $(($OPTIND - 1))
# Move argument pointer to next.

# All this is not nearly as complicated as it looks <grin>.
```

![[Example 15-21|Example 15-21]]

**Script Behavior**

**source**, . ([[special-characters#^DOTREF|dot]] command)

This command, when invoked from the command-line, executes a script. Within a script, a **source file-name** loads the file file-name. _Sourcing_ a file (dot-command) _imports_ code into the script, appending to the script (same effect as the `#include` directive in a _C_ program). The net result is the same as if the "sourced" lines of code were physically present in the body of the script. This is useful in situations when multiple scripts use a common data file or function library.

![[Example 15-22|Example 15-22]]

File data-file for [[Example 15-22|Example 15-22]], above. Must be present in same directory.

```bash
# This is a data file loaded by a script.
# Files of this type may contain variables, functions, etc.
# It loads with a 'source' or '.' command from a shell script.

# Let's initialize some variables.

variable1=23
variable2=474
variable3=5
variable4=97

message1="Greetings from *** line $LINENO *** of the data file!"
message2="Enough for now. Goodbye."

print_message ()
{   # Echoes any message passed to it.

  if [ -z "$1" ]
  then
    return 1 # Error, if argument missing.
  fi

  echo

  until [ -z "$1" ]
  do             # Step through arguments passed to function.
    echo -n "$1" # Echo args one at a time, suppressing line feeds.
    echo -n " "  # Insert spaces between words.
    shift        # Next one.
  done  

  echo

  return 0
}
```

If the _sourced_ file is itself an executable script, then it will run, then return control to the script that called it. A _sourced_ executable script may use a [[complex-functions-and-function-complexities#^RETURNREF|return]] for this purpose.

Arguments may be (optionally) passed to the _sourced_ file as [[othertypesv#^POSPARAMREF1|positional parameters]].

```bash
source $filename $arg1 arg2
```

It is even possible for a script to _source_ itself, though this does not seem to have any practical applications.

![[Example 15-23|Example 15-23]]

**exit**

Unconditionally terminates a script. [^6] The **exit** command may optionally take an integer argument, which is returned to the shell as the [[exit-and-exit-status#^EXITSTATUSREF|exit status]] of the script. It is good practice to end all but the simplest scripts with an **exit 0**, indicating a successful run.

> [!note]
> If a script terminates with an **exit** lacking an argument, the exit status of the script is the exit status of the last command executed in the script, not counting the **exit**. This is equivalent to an **exit $?**.

> [!note]
> An **exit** command may also be used to terminate a [[subshells#^SUBSHELLSREF|subshell]].

**exec**

This shell builtin replaces the current process with a specified command. Normally, when the shell encounters a command, it [[internal-commands-and-builtins#^FORKREF|forks off]] a child process to actually execute the command. Using the **exec** builtin, the shell does not fork, and the command _exec_'ed replaces the shell. When used in a script, therefore, it forces an exit from the script when the **exec**'ed command terminates. [^7]

![[Example 15-24|Example 15-24]]

![[Example 15-25|Example 15-25]]

An **exec** also serves to [[using-exec#^USINGEXECREF|reassign file descriptors]]. For example, **exec <zzz-file** replaces stdin with the file zzz-file.

> [!note]
> The -exec option to [[external-filters-programs-and-commands#^FINDREF|find]] is _not_ the same as the **exec** shell builtin.

**shopt**

This command permits changing _shell options_ on the fly (see [[Example 25-1|Example 25-1]] and [[Example 25-2|Example 25-2]]). It often appears in the Bash [[important-files#^FILESREF1|startup files]], but also has its uses in scripts. Needs [[bash-version-2-3-and-4#^BASH2REF|version 2]] or later of Bash.

```bash
shopt -s cdspell
# Allows minor misspelling of directory names with 'cd'
# Option -s sets, -u unsets.

cd /hpme  # Oops! Mistyped '/home'.
pwd       # /home
          # The shell corrected the misspelling.
```

**caller**

Putting a **caller** command inside a [[functions|function]] echoes to stdout information about the _caller_ of that function.

```bash
#!/bin/bash

function1 ()
{
  # Inside function1 ().
  caller 0   # Tell me about it.
}

function1    # Line 9 of script.

# 9 main test.sh
# ^                 Line number that the function was called from.
#   ^^^^            Invoked from "main" part of script.
#        ^^^^^^^    Name of calling script.

caller 0     # Has no effect because it's not inside a function.
```

A **caller** command can also return _caller_ information from a script [[internal-commands-and-builtins#^SOURCEREF|sourced]] within another script. Analogous to a function, this is a "subroutine call."

You may find this command useful in debugging.

**Commands**

**true**

A command that returns a successful (zero) [[exit-and-exit-status#^EXITSTATUSREF|exit status]], but does nothing else.

```bash
bash$ true
bash$ echo $?
0
	      
```

```bash
# Endless loop
while true   # alias for ":"
do
   operation-1
   operation-2
   ...
   operation-n
   # Need a way to break out of loop or script will hang.
done
```

**false**

A command that returns an unsuccessful [[exit-and-exit-status#^EXITSTATUSREF|exit status]], but does nothing else.

```bash
bash$ false
bash$ echo $?
1
	      
```

```bash
# Testing "false" 
if false
then
  echo "false evaluates \"true\""
else
  echo "false evaluates \"false\""
fi
# false evaluates "false"


# Looping while "false" (null loop)
while false
do
   # The following code will not execute.
   operation-1
   operation-2
   ...
   operation-n
   # Nothing happens!
done   
```

**type [cmd]**

Similar to the [[file-and-archiving-commands#^WHICHREF|which]] external command, **type cmd** identifies "cmd." Unlike **which**, **type** is a Bash builtin. The useful -a option to **type** identifies _keywords_ and _builtins_, and also locates system commands with identical names.

```bash
bash$ type '['
[ is a shell builtin
bash$ type -a '['
[ is a shell builtin
 [ is /usr/bin/[


bash$ type type
type is a shell builtin
	      
```

The **type** command can be useful for [[special-characters#^DEVNULLREDIRECT|testing whether a certain command exists]].

**hash [cmds]**

Records the _path_ name of specified commands -- in the shell _hash table_ [^8] -- so the shell or script will not need to search the [[another-look-at-variables#^PATHREF|$PATH]] on subsequent calls to those commands. When **hash** is called with no arguments, it simply lists the commands that have been hashed. The -r option resets the hash table.

**bind**

The **bind** builtin displays or modifies _readline_ [^9] key bindings.

**help**

Gets a short usage summary of a shell builtin. This is the counterpart to [[file-and-archiving-commands#^WHATISREF|whatis]], but for builtins. The display of _help_ information got a much-needed update in the [[bash-version-2-3-and-4#^BASH4REF|version 4 release]] of Bash.

```bash
bash$ help exit
exit: exit [n]
    Exit the shell with a status of N.  If N is omitted, the exit status
    is that of the last command executed.
	      
```

## Job Control Commands

Certain of the following job control commands take a _job identifier_ as an argument. See the [[internal-commands-and-builtins#^JOBIDTABLE|table]] at end of the chapter.

**jobs**

Lists the jobs running in the background, giving the _job number_. Not as useful as [[system-and-administrative-commands#^PPSSREF|ps]].

> [!note]
> It is all too easy to confuse _jobs_ and _processes_. Certain [[internal-commands-and-builtins|builtins]], such as **kill**, **disown**, and **wait** accept either a job number or a process number as an argument. The [[internal-commands-and-builtins#^FGREF|fg]], [[internal-commands-and-builtins#^BGREF|bg]] and **jobs** commands accept only a job number.
>
> ```bash
> bash$ sleep 100 &
> [1] 1384
> 
> bash $ jobs
> [1]+  Running                 sleep 100 &
> ```
>
> "1" is the job number (jobs are maintained by the current shell). "1384" is the [[another-look-at-variables#^PPIDREF|PID]] or _process ID number_ (processes are maintained by the system). To kill this job/process, either a **kill %1** or a **kill 1384** works.
>
> _Thanks, S.C._

**disown**

Remove job(s) from the shell's table of active jobs.

**fg**, **bg**

The **fg** command switches a job running in the background into the foreground. The **bg** command restarts a suspended job, and runs it in the background. If no job number is specified, then the **fg** or **bg** command acts upon the currently running job.

**wait**

Suspend script execution until all jobs running in background have terminated, or until the job number or process ID specified as an option terminates. Returns the [[exit-and-exit-status#^EXITSTATUSREF|exit status]] of waited-for command.

You may use the **wait** command to prevent a script from exiting before a background job finishes executing (this would create a dreaded [[internal-commands-and-builtins#^ZOMBIEREF|orphan process]]).

![[Example 15-26|Example 15-26]]

Optionally, **wait** can take a _job identifier_ as an argument, for example, _wait%1_ or _wait $PPID_. [^10] See the [[internal-commands-and-builtins#^JOBIDTABLE|job id table]].

> [!tip]
> Within a script, running a command in the background with an ampersand (&) may cause the script to hang until **ENTER** is hit. This seems to occur with commands that write to stdout. It can be a major annoyance.
>
> ```bash
> #!/bin/bash
> # test.sh		  
> 
> ls -l &
> echo "Done."
> bash$ ./test.sh
> Done.
>  [bozo@localhost test-scripts]$ total 1
>  -rwxr-xr-x    1 bozo     bozo           34 Oct 11 15:09 test.sh
>  _
>                
> ```
>
>     As Walter Brameld IV explains it:  
>   
>     As far as I can tell, such scripts don't actually hang. It just  
>     seems that they do because the background command writes text to  
>     the console after the prompt. The user gets the impression that  
>     the prompt was never displayed. Here's the sequence of events:  
>   
>     1. Script launches background command.  
>     2. Script exits.  
>     3. Shell displays the prompt.  
>     4. Background command continues running and writing text to the  
>        console.  
>     5. Background command finishes.  
>     6. User doesn't see a prompt at the bottom of the output, thinks script  
>        is hanging.  
>
> Placing a **wait** after the background command seems to remedy this.
>
> ```bash
> #!/bin/bash
> # test.sh		  
> 
> ls -l &
> echo "Done."
> wait
> bash$ ./test.sh
> Done.
>  [bozo@localhost test-scripts]$ total 1
>  -rwxr-xr-x    1 bozo     bozo           34 Oct 11 15:09 test.sh
>                
> ```
>
> [[io-redirection|Redirecting]] the output of the command to a file or even to /dev/null also takes care of this problem.|

**suspend**

This has a similar effect to **Control**-**Z**, but it suspends the shell (the shell's parent process should resume it at an appropriate time).

**logout**

Exit a login shell, optionally specifying an [[exit-and-exit-status#^EXITSTATUSREF|exit status]].

**times**

Gives statistics on the system time elapsed when executing commands, in the following form:

```bash
0m0.020s 0m0.020s
```

This capability is of relatively limited value, since it is not common to profile and benchmark shell scripts.

**kill**

Forcibly terminate a process by sending it an appropriate _terminate_ signal (see [[Example 17-6|Example 17-6]]).

![[Example 15-27|Example 15-27]]

> [!note]
> **kill -l** lists all the [[debugging#^SIGNALD|signals]] (as does the file /usr/include/asm/signal.h). A **kill -9** is a _sure kill_, which will usually terminate a process that stubbornly refuses to die with a plain **kill**. Sometimes, a **kill -15** works. A _zombie_ process, that is, a child process that has terminated, but that the [[internal-commands-and-builtins#^FORKREF|parent process]] has not (yet) killed, cannot be killed by a logged-on user -- you can't kill something that is already dead -- but **init** will generally clean it up sooner or later.

**killall**

The **killall** command kills a running process by _name_, rather than by [[special-characters#^PROCESSIDREF|process ID]]. If there are multiple instances of a particular command running, then doing a _killall_ on that command will terminate them _all_.

> [!note]
> This refers to the **killall** command in /usr/bin, _not_ the [[#KILLALL2REF|killall script]] in /etc/rc.d/init.d.

**command**

The **command** directive disables aliases and functions for the command immediately following it.

```bash
bash$ command ls
              
```

> [!note]
> This is one of three shell directives that effect script command processing. The others are [[internal-commands-and-builtins#^BLTREF|builtin]] and [[internal-commands-and-builtins#^ENABLEREF|enable]].

**builtin**

Invoking **builtin BUILTIN_COMMAND** runs the command _BUILTIN_COMMAND_ as a shell [[internal-commands-and-builtins|builtin]], temporarily disabling both functions and external system commands with the same name.

**enable**

This either enables or disables a shell builtin command. As an example, _enable -n kill_ disables the shell builtin [[internal-commands-and-builtins#^KILLREF|kill]], so that when Bash subsequently encounters _kill_, it invokes the external command /bin/kill.

The -a option to _enable_ lists all the shell builtins, indicating whether or not they are enabled. The -f filename option lets _enable_ load a [[internal-commands-and-builtins|builtin]] as a shared library (DLL) module from a properly compiled object file. [^11].

**autoload**

This is a port to Bash of the _ksh_ autoloader. With **autoload** in place, a function with an _autoload_ declaration will load from an external file at its first invocation. [^12] This saves system resources.

Note that _autoload_ is not a part of the core Bash installation. It needs to be loaded in with _enable -f_ (see above).

###### Table 15-1. Job identifiers

|Notation|Meaning|
|:--|:--|
|%N|Job number [N]|
|%S|Invocation (command-line) of job begins with string _S_|
|%?S|Invocation (command-line) of job contains within it string _S_|
|%%|"current" job (last job stopped in foreground or started in background)|
|%+|"current" job (last job stopped in foreground or started in background)|
|%-|Last job|
|$!|Last background process|

[^1]: As Nathan Coulter points out, "while forking a process is a low-cost operation, executing a new program in the newly-forked child process adds more overhead."

[^2]: An exception to this is the [[external-filters-programs-and-commands#^TIMREF|time]] command, listed in the official Bash documentation as a keyword ("reserved word").

[^3]: Note that _let_ [[gotchas#^LETBAD|cannot be used for setting _string_ variables.]]

[^4]: To _Export_ information is to make it available in a more general context. See also [[subshells#^SCOPEREF|scope]].

[^5]: An _option_ is an argument that acts as a flag, switching script behaviors on or off. The argument associated with a particular option indicates the behavior that the option (flag) switches on or off.

[^6]: Technically, an **exit** only terminates the process (or shell) in which it is running _not the parent process_.

[^7]: Unless the **exec** is used to [[using-exec#^USINGEXECREF|reassign file descriptors]].

[^8]: _Hashing_ is a method of creating lookup keys for data stored in a table. The _data items themselves_ are "scrambled" to create keys, using one of a number of simple mathematical _algorithms_ (methods, or recipes).

    An advantage of _hashing_ is that it is fast. A disadvantage is that _collisions_ -- where a single key maps to more than one data item -- are possible.

    For examples of hashing see [[Example A-20|Example A-20]] and [[Example A-21|Example A-21]].

[^9]: The _readline_ library is what Bash uses for reading input in an interactive shell.

[^10]: This only applies to _child processes_, of course.

[^11]: The C source for a number of loadable builtins is typically found in the /usr/share/doc/bash-?.??/functions directory.

    Note that the -f option to **enable** is not [[portability-issues|portable]] to all systems.

[^12]: The same effect as **autoload** can be achieved with [[typing-variables.html|typeset -fu]].