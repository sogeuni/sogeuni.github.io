---
title: 17.1. Analyzing a System Script
---


Using our knowledge of administrative commands, let us examine a system script. One of the shortest and simplest to understand scripts is "killall," [^1] used to suspend running processes at system shutdown.

###### Example 17-12. *killall*, from /etc/rc.d/init.d

```bash
#!/bin/sh

# --> Comments added by the author of this document marked by "# -->".

# --> This is part of the 'rc' script package
# --> by Miquel van Smoorenburg, <miquels@drinkel.nl.mugnet.org>.

# --> This particular script seems to be Red Hat / FC specific
# --> (may not be present in other distributions).

#  Bring down all unneeded services that are still running
#+ (there shouldn't be any, so this is just a sanity check)

for i in /var/lock/subsys/*; do
        # --> Standard for/in loop, but since "do" is on same line,
        # --> it is necessary to add ";".
        # Check if the script is there.
        [ ! -f $i ] && continue
        # --> This is a clever use of an "and list", equivalent to:
        # --> if [ ! -f "$i" ]; then continue

        # Get the subsystem name.
        subsys=${i#/var/lock/subsys/}
        # --> Match variable name, which, in this case, is the file name.
        # --> This is the exact equivalent of subsys=`basename $i`.
	
        # -->  It gets it from the lock file name
        # -->+ (if there is a lock file,
        # -->+ that's proof the process has been running).
        # -->  See the "lockfile" entry, above.


        # Bring the subsystem down.
        if [ -f /etc/rc.d/init.d/$subsys.init ]; then
           /etc/rc.d/init.d/$subsys.init stop
        else
           /etc/rc.d/init.d/$subsys stop
        # -->  Suspend running jobs and daemons.
        # -->  Note that "stop" is a positional parameter,
        # -->+ not a shell builtin.
        fi
done
```

That wasn't so bad. Aside from a little fancy footwork with variable matching, there is no new material there.

**Exercise 1.** In /etc/rc.d/init.d, analyze the **halt** script. It is a bit longer than **killall**, but similar in concept. Make a copy of this script somewhere in your home directory and experiment with it (do _not_ run it as _root_). Do a simulated run with the -vn flags (**sh -vn scriptname**). Add extensive comments. Change the commands to [[internal-commands-and-builtins#^ECHOREF|echos]].

**Exercise 2.** Look at some of the more complex scripts in /etc/rc.d/init.d. Try to understand at least portions of them. Follow the above procedure to analyze them. For some additional insight, you might also examine the file sysvinitfiles in /usr/share/doc/initscripts-?.??, which is part of the "initscripts" documentation.

[^1]: The _killall_ system script should not be confused with the [[internal-commands-and-builtins#^KILLALLREF|killall]] command in /usr/bin.
