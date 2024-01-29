---
title: 6. Exit and Exit Status
---


> ... there are dark corners in the Bourne shell, and people use all of them.
>
> --<cite>Chet Ramey</cite>

The **exit** command terminates a script, just as in a **C** program. It can also return a value, which is available to the script's parent process.

Every command returns an _exit status_ (sometimes referred to as a _return status_ or _exit code_). A successful command returns a 0, while an unsuccessful one returns a non-zero value that usually can be interpreted as an _error code_. Well-behaved UNIX commands, programs, and utilities return a 0 exit code upon successful completion, though there are some exceptions.

Likewise, [[functions|functions]] within a script and the script itself return an exit status. The last command executed in the function or script determines the exit status. Within a script, an **exit _nnn_** command may be used to deliver an _nnn_ exit status to the shell (_nnn_ must be an integer in the 0 - 255 range).

> [!note]
> When a script ends with an **exit** that has no parameter, the exit status of the script is the exit status of the last command executed in the script (previous to the **exit**).
>
> ```bash
> #!/bin/bash
> 
> COMMAND_1
> 
> . . .
> 
> COMMAND_LAST
> 
> # Will exit with status of last command.
> 
> exit
> ```
>
> The equivalent of a bare **exit** is **exit $?** or even just omitting the **exit**.
>
> ```bash
> #!/bin/bash
> 
> COMMAND_1
> 
> . . .
> 
> COMMAND_LAST
> 
> # Will exit with status of last command.
> 
> exit $?
> ```
>
> ```bash
> #!/bin/bash
> 
> COMMAND1
> 
> . . . 
> 
> COMMAND_LAST
> 
> # Will exit with status of last command.
> ```

`$?` reads the exit status of the last command executed. After a function returns, `$?` gives the exit status of the last command executed in the function. This is Bash's way of giving functions a "return value." [^1]

Following the execution of a [[special-characters#^PIPEREF|pipe]], a $? gives the exit status of the last command executed.

After a script terminates, a $? from the command-line gives the exit status of the script, that is, the last command executed in the script, which is, by convention, **0** on success or an integer in the range 1 - 255 on error.

![[Example 6-1|Example 6-1]]

[[another-look-at-variables#^XSTATVARREF|$?]] is especially useful for testing the result of a command in a script (see [[Example 16-35|Example 16-35]] and [[Example 16-20|Example 16-20]]).

> [!note] The [[special-characters#^NOTREF|!]], the _logical not_ qualifier, reverses the outcome of a test or command, and this affects its [[exit-and-exit-status#^EXITSTATUSREF|exit status]].
>
> ![[Example 6-2|Example 6-2]]

> [!caution] Certain exit status codes have [[exit-codes-with-special-meanings#^EXITCODESREF|reserved meanings]] and should not be user-specified in a script.

[^1]: In those instances when there is no [[complex-functions-and-function-complexities#^RETURNREF|return]] terminating the function.
