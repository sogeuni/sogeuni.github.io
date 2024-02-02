---
title: Appendix J. An Introduction to Programmable Completion
---


The _programmable completion_ feature in Bash permits typing a partial command, then pressing the **[Tab]** key to auto-complete the command sequence. [^1] If multiple completions are possible, then **[Tab]** lists them all. Let's see how it works.

```bash
bash$ xtra[Tab]
xtraceroute       xtrapin           xtrapproto
 xtraceroute.real  xtrapinfo         xtrapreset
 xtrapchar         xtrapout          xtrapstats


bash$ xtrac[Tab]
xtraceroute       xtraceroute.real


bash$ xtraceroute.r[Tab]
xtraceroute.real
      
```

Tab completion also works for variables and path names.

```bash
bash$ echo $BASH[Tab]
$BASH                 $BASH_COMPLETION      $BASH_SUBSHELL
 $BASH_ARGC            $BASH_COMPLETION_DIR  $BASH_VERSINFO
 $BASH_ARGV            $BASH_LINENO          $BASH_VERSION
 $BASH_COMMAND         $BASH_SOURCE


bash$ echo /usr/local/[Tab]
bin/     etc/     include/ libexec/ sbin/    src/     
 doc/     games/   lib/     man/     share/
      
```

The Bash **complete** and **compgen** [[internal-commands-and-builtins|builtins]] make it possible for _tab completion_ to recognize partial _parameters_ and _options_ to commands. In a very simple case, we can use **complete** from the command-line to specify a short list of acceptable parameters.

```bash
bash$ touch sample_command
bash$ touch file1.txt file2.txt file2.doc file30.txt file4.zzz
bash$ chmod +x sample_command
bash$ complete -f -X '!*.txt' sample_command


bash$ ./sample[Tab][Tab]
sample_command
file1.txt   file2.txt   file30.txt
  
```

The -f option to _complete_ specifies filenames, and -X the filter pattern.

For anything more complex, we could write a script that specifies a list of acceptable command-line parameters. The **compgen** builtin expands a list of _arguments_ to _generate_ completion matches.

Let us take a [[contributed-scripts.html#USEGETOPT2|modified version]] of the _UseGetOpt.sh_ script as an example command. This script accepts a number of command-line parameters, preceded by either a single or double dash. And here is the corresponding _completion script_, by convention given a filename corresponding to its associated command.

![[Example J-1|Example J-1]]

Now, let's try it.

```bash
bash$ source UseGetOpt-2

bash$ ./UseGetOpt-2.sh -[Tab]
--         --aoption  --debug    --file     --help     --log     --test
 -a         -d         -f         -h         -l         -t


bash$ ./UseGetOpt-2.sh --[Tab]
--         --aoption  --debug    --file     --help     --log     --test
  
```

We begin by [[internal.html#SOURCEREF|sourcing]] the "completion script." This sets the command-line parameters. [^2]

In the first instance, hitting **[Tab]** after a single dash, the output is all the possible parameters preceded by _one or more_ dashes. Hitting **[Tab]** after _two_ dashes gives the possible parameters preceded by _two or more_ dashes.

Now, just what is the point of having to jump through flaming hoops to enable command-line tab completion? _It saves keystrokes._ [^3]

--

_Resources:_

Bash [programmable completion](http://freshmeat.net/projects/bashcompletion) project

Mitch Frazier's [_Linux Journal_](http://www.linuxjournal.com) article, [_More on Using the Bash Complete Command_](http://www.linuxjournal.com/content/more-using-bash-complete-command)

Steve's excellent two-part article, "An Introduction to Bash Completion": [Part 1](http://www.debian-administration.org/article/An_introduction_to_bash_completion_part_1) and [Part 2](http://www.debian-administration.org/article/An_introduction_to_bash_completion_part_2)

[^1]: This works only from the _command line_, of course, and not within a script.

[^2]: Normally the default parameter completion files reside in either the /etc/profile.d directory or in /etc/bash_completion. These autoload on system startup. So, after writing a useful completion script, you might wish to move it (as _root_, of course) to one of these directories.

[^3]: It has been extensively documented that programmers are willing to put in long hours of effort in order to save ten minutes of "unnecessary" labor. This is known as _optimization_.
