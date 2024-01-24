---
title: Appendix L. History Commands
---


The Bash shell provides command-line tools for editing and manipulating a user's _command history_. This is primarily a convenience, a means of saving keystrokes.

Bash history commands:

1. **history**
    
2. **fc**
    

```bash
bash$ history
   1  mount /mnt/cdrom
    2  cd /mnt/cdrom
    3  ls
     ...
	      
```

Internal variables associated with Bash history commands:

1. $HISTCMD
    
2. $HISTCONTROL
    
3. $HISTIGNORE
    
4. $HISTFILE
    
5. $HISTFILESIZE
    
6. $HISTSIZE
    
7. $HISTTIMEFORMAT (Bash, ver. 3.0 or later)
    
8. !!
    
9. !$
    
10. !#
    
11. !N
    
12. !-N
    
13. !STRING
    
14. !?STRING?
    
15. ^STRING^string^
    

Unfortunately, the Bash history tools find no use in scripting.

```bash
#!/bin/bash
# history.sh
# A (vain) attempt to use the 'history' command in a script.

history                      # No output.

var=$(history); echo "$var"  # $var is empty.

#  History commands are, by default, disabled within a script.
#  However, as dhw points out,
#+ set -o history
#+ enables the history mechanism.

set -o history
var=$(history); echo "$var"   # 1  var=$(history)
```

```bash
bash$ ./history.sh
(no output)	      
	      
```

The [Advancing in the Bash Shell](http://samrowe.com/wordpress/advancing-in-the-bash-shell/) site gives a good introduction to the use of history commands in Bash.
