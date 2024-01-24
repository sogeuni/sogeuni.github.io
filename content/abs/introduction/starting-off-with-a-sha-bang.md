---
title: 2. Starting Off With a Sha-Bang
---

> Shell programming is a 1950s juke box . . .
>
> --<cite>Larry Wall</cite>

In the simplest case, a script is nothing more than a list of system commands stored in a file. At the very least, this saves the effort of retyping that particular sequence of commands each time it is invoked.

![[./Example 2-1|Example 2-1]]

###### Example 2-1. *cleanup*: A script to clean up log files in `/var/log`

```bash
# Cleanup
# Run as root, of course.

cd /var/log
cat /dev/null > messages
cat /dev/null > wtmp
echo "Log files cleaned up."
```

There is nothing unusual here, only a set of commands that could just as easily have been invoked one by one from the command-line on the console or in a terminal window. The advantages of placing the commands in a script go far beyond not having to retype them time and again. The script becomes a *program* -- a *tool* -- and it can easily be modified or customized for a particular application.

###### Example 2-2. *cleanup*: An improved clean-up script

```bash
#!/bin/bash
# Proper header for a Bash script.

# Cleanup, version 2

# Run as root, of course.
# Insert code here to print error message and exit if not root.

LOG_DIR=/var/log
# Variables are better than hard-coded values.
cd $LOG_DIR

cat /dev/null > messages
cat /dev/null > wtmp


echo "Logs cleaned up."

exit #  The right and proper method of "exiting" from a script.
     #  A bare "exit" (no parameter) returns the exit status
     #+ of the preceding command. 
```
^111

Now *that's* beginning to look like a real script. But we can go even farther...

###### Example 2-3. *cleanup*: An enhanced and generalized version of above scripts.

```bash
#!/bin/bash
# Cleanup, version 3

#  Warning:
#  -------
#  This script uses quite a number of features that will be explained
#+ later on.
#  By the time you've finished the first half of the book,
#+ there should be nothing mysterious about it.



LOG_DIR=/var/log
ROOT_UID=0     # Only users with $UID 0 have root privileges.
LINES=50       # Default number of lines saved.
E_XCD=86       # Can't change directory?
E_NOTROOT=87   # Non-root exit error.


# Run as root, of course.
if [ "$UID" -ne "$ROOT_UID" ]
then
  echo "Must be root to run this script."
  exit $E_NOTROOT
fi  

if [ -n "$1" ]
# Test whether command-line argument is present (non-empty).
then
  lines=$1
else  
  lines=$LINES # Default, if not specified on command-line.
fi  


#  Stephane Chazelas suggests the following,
#+ as a better way of checking command-line arguments,
#+ but this is still a bit advanced for this stage of the tutorial.
#
#    E_WRONGARGS=85  # Non-numerical argument (bad argument format).
#
#    case "$1" in
#    ""      ) lines=50;;
#    *[!0-9]*) echo "Usage: `basename $0` lines-to-cleanup";
#     exit $E_WRONGARGS;;
#    *       ) lines=$1;;
#    esac
#
#* Skip ahead to "Loops" chapter to decipher all this.


cd $LOG_DIR

if [ `pwd` != "$LOG_DIR" ]  # or   if [ "$PWD" != "$LOG_DIR" ]
                            # Not in /var/log?
then
  echo "Can't change to $LOG_DIR."
  exit $E_XCD
fi  # Doublecheck if in right directory before messing with log file.

# Far more efficient is:
#
# cd /var/log || {
#   echo "Cannot change to necessary directory." >&2
#   exit $E_XCD;
# }




tail -n $lines messages > mesg.temp # Save last section of message log file.
mv mesg.temp messages               # Rename it as system log file.


#  cat /dev/null > messages
#* No longer needed, as the above method is safer.

cat /dev/null > wtmp  #  ': > wtmp' and '> wtmp'  have the same effect.
echo "Log files cleaned up."
#  Note that there are other log files in /var/log not affected
#+ by this script.

exit 0
#  A zero return value from the script upon exit indicates success
#+ to the shell.
```

Since you may not wish to wipe out the entire system log, this version of the script keeps the last section of the message log intact. You will constantly discover ways of fine-tuning previously written scripts for increased effectiveness.

* * *

The *sha-bang* (`#!`) [^1] at the head of a script tells your system that this file is a set of commands to be fed to the command interpreter indicated. The #! is actually a two-byte [^2] *magic number*, a special marker that designates a file type, or in this case an executable shell script (type **man magic** for more details on this fascinating topic). Immediately following the *sha-bang* is a *path name*. This is the path to the program that interprets the commands in the script, whether it be a shell, a programming language, or a utility. This command interpreter then executes the commands in the script, starting at the top (the line following the *sha-bang* line), and ignoring comments. [^3] ^magnumref

```bash
#!/bin/sh
#!/bin/bash
#!/usr/bin/perl
#!/usr/bin/tcl
#!/bin/sed -f
#!/bin/awk -f
```

Each of the above script header lines calls a different command interpreter, be it `/bin/sh`, the default shell (**bash** in a Linux system) or otherwise. [^4] Using `#!/bin/sh`, the default Bourne shell in most commercial variants of UNIX, makes the script [[../advanced-topics/portability-issues|portable]] to non-Linux machines, though you [[../advanced-topics/gotchas#^BINSH|sacrifice Bash-specific features]]. The script will, however, conform to the POSIX [^5] **sh** standard.

Note that the path given at the "sha-bang" must be correct, otherwise an error message -- usually "Command not found." -- will be the only result of running the script. [^6]

#! can be omitted if the script consists only of a set of generic system commands, using no internal shell directives. The second example, above, requires the initial #!, since the variable assignment line, **lines=50**, uses a shell-specific construct. [^7] Note again that **#!/bin/sh** invokes the default shell interpreter, which defaults to /bin/bash on a Linux machine.

> [!note]
> This tutorial encourages a modular approach to constructing a script. Make note of and collect "boilerplate" code snippets that might be useful in future scripts. Eventually you will build quite an extensive library of nifty routines. As an example, the following script prolog tests whether the script has been invoked with the correct number of parameters.
>
> ```bash
> E_WRONG_ARGS=85
> script_parameters="-a -h -m -z"
> #                  -a = all, -h = help, etc.
> 
> if [ $# -ne $Number_of_expected_args ]
> then
>   echo "Usage: `basename $0` $script_parameters"
>   # `basename $0` is the script's filename.
>   exit $E_WRONG_ARGS
> fi 
> ```
>
> Many times, you will write a script that carries out one particular task. The first script in this chapter is an example. Later, it might occur to you to generalize the script to do other, similar tasks. Replacing the literal ("hard-wired") constants by variables is a step in that direction, as is replacing repetitive code blocks by [[../advanced-topics/functions|functions]].

## Invoking the script

Having written the script, you can invoke it by **`sh scriptname`**, [^8] or alternatively **`bash scriptname`**. (Not recommended is using **`sh <scriptname`**, since this effectively disables reading from [[../apendix/a-detailed-introduction-to-io-and-io-redirection|stdin]] within the script.) Much more convenient is to make the script itself directly executable with a [[../commands/basic-commands#chmod|chmod]].

Either:

**`chmod 555 scriptname`** (gives everyone read/execute permission) [^9]

or

**`chmod +rx scriptname`** (gives everyone read/execute permission)

**`chmod u+rx scriptname`** (gives only the script owner read/execute permission)

Having made the script executable, you may now test it by **`./scriptname`**. [^10] If it begins with a "sha-bang" line, invoking the script calls the correct command interpreter to run it.

As a final step, after testing and debugging, you would likely want to move it to `/usr/local/bin` (as *root*, of course), to make the script available to yourself and all other users as a systemwide executable. The script could then be invoked by simply typing **`scriptname [ENTER]`** from the command-line.

## Preliminary Exercises

1. System administrators often write scripts to automate common tasks. Give several instances where such scripts would be useful.
2. Write a script that upon invocation shows the [[../commands/time-date-commands#date|time and date]], [[../commands/system-and-administrative-commands#who|lists all logged-in users]], and gives the system [[../commands/system-and-administrative-commands#uptime|uptime]]. The script then [[../advanced-topics/io-redirection|saves this information]] to a logfile.

[^1]: More commonly seen in the literature as *she-bang* or *sh-bang*. This derives from the concatenation of the tokens *sharp* (#) and *bang* (!).

[^2]: Some flavors of UNIX (those based on 4.2 BSD) allegedly take a four-byte magic number, requiring a blank after the ! -- **#! /bin/sh**. [According to Sven Mascheck](http://www.in-ulm.de/~mascheck/various/shebang/#details) this is probably a myth.

[^3]: The #! line in a shell script will be the first thing the command interpreter (**sh** or **bash**) sees. Since this line begins with a #, it will be correctly interpreted as a comment when the command interpreter finally executes the script. The line has already served its purpose - calling the command interpreter.

    If, in fact, the script includes an *extra* #! line, then **bash** will interpret it as a comment.

    ```bash
    #!/bin/bash

    echo "Part 1 of script."
    a=1

    #!/bin/bash
    # This does *not* launch a new script.

    echo "Part 2 of script."
    echo $a  # Value of $a stays at 1.
    ```

[^4]: This allows some cute tricks.
    
    ```bash
    #!/bin/rm
    # Self-deleting script.

    # Nothing much seems to happen when you run this... except that the file disappears.

    WHATEVER=85

    echo "This line will never print (betcha!)."

    exit $WHATEVER  # Doesn't matter. The script will not exit here.
                    # Try an echo $? after script termination.
                    # You'll get a 0, not a 85.
    ```

    Also, try starting a README file with a **#!/bin/more**, and making it executable. The result is a self-listing documentation file. (A [[../advanced-topics/here-documents#^HEREDOCREF|here document]] using [[../commands/basic-commands#^CATREF|cat]] is possibly a better alternative -- see [[../advanced-topics/here-documents#^EX71|Example 19-3]]).

[^5]: **P**ortable **O**perating **S**ystem *I*nterface, an attempt to standardize UNI**X**-like OSes. The POSIX specifications are listed on the [Open Group site](http://www.opengroup.org/onlinepubs/007904975/toc.htm).

[^6]: To avoid this possibility, a script may begin with a [[../commands/system-and-administrative-commands#^ENVV2REF|#!/bin/env bash]] *sha-bang* line. This may be useful on UNIX machines where *bash* is not located in /bin

[^7]: If *Bash* is your default shell, then the #! isn't necessary at the beginning of a script. However, if launching a script from a different shell, such as *tcsh*, then you *will* need the #!.

[^8]: Caution: invoking a *Bash* script by **`sh scriptname`** turns off Bash-specific extensions, and the script may therefore fail to execute.

[^9]: A script needs *read*, as well as execute permission for it to run, since the shell needs to be able to read it.

[^10]: Why not simply invoke the script with **scriptname**? If the directory you are in ([[../beyond-the-basic/another-look-at-variables#PWDREF|$PWD]]) is where scriptname is located, why doesn't this work? This fails because, for security reasons, the current directory (./) is not by default included in a user's [[../beyond-the-basic/another-look-at-variables#PATHREF|$PATH]]. It is therefore necessary to explicitly invoke the script in the current directory with a **./scriptname**.
