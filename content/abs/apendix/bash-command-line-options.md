---
title: G.2. Bash Command-Line Options
---


_Bash_ itself has a number of command-line options. Here are some of the more useful ones.

- -c
    _Read commands from the following string and assign any arguments to the [[another-look-at-variables#^POSPARAMREF|positional parameters]]._
    
    ```bash
    bash$ bash -c 'set a b c d; IFS="+-;"; echo "$*"'
    a+b+c+d
	      
    ```
- -r
    --restricted

    _Runs the shell, or a script, in [[restricted-shells#^RESTRICTEDSHREF|restricted mode]]._

- --posix

    _Forces Bash to conform to [[sha-bang#^POSIX2REF|POSIX]] mode._

- --version

    _Display Bash version information and exit._

- \--

    _End of options. Anything further on the command line is an argument, not an option._
