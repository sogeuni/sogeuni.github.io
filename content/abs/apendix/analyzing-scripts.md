---
title: O.1. Analyzing Scripts
---


Examine the following script. Run it, then explain what it does. Annotate the script and rewrite it in a more compact and elegant manner.

```bash
#!/bin/bash

MAX=10000


  for((nr=1; nr<$MAX; nr++))
  do

    let "t1 = nr % 5"
    if [ "$t1" -ne 3 ]
    then
      continue
    fi

    let "t2 = nr % 7"
    if [ "$t2" -ne 4 ]
    then
      continue
    fi

    let "t3 = nr % 9"
    if [ "$t3" -ne 5 ]
    then
      continue
    fi

  break   # What happens when you comment out this line? Why?

  done

  echo "Number = $nr"


exit 0
```

---

Explain what the following script does. It is really just a parameterized command-line pipe.

```bash
#!/bin/bash

DIRNAME=/usr/bin
FILETYPE="shell script"
LOGFILE=logfile

file "$DIRNAME"/* | fgrep "$FILETYPE" | tee $LOGFILE | wc -l

exit 0
```

---

Examine and explain the following script. For hints, you might refer to the listings for [[../commands/complex-commands#^FINDREF|find]] and [[../commands/system-and-administrative-commands#^STATREF|stat]].

```bash
#!/bin/bash

# Author:  Nathan Coulter
# This code is released to the public domain.
# The author gave permission to use this code snippet in the ABS Guide.

find -maxdepth 1 -type f -printf '%f\000'  | {
   while read -d $'\000'; do
      mv "$REPLY" "$(date -d "$(stat -c '%y' "$REPLY") " '+%Y%m%d%H%M%S'
      )-$REPLY"
   done
}

# Warning: Test-drive this script in a "scratch" directory.
# It will somehow affect all the files there.
```

---

A reader sent in the following code snippet.

```bash
while read LINE
do
  echo $LINE
done < `tail -f /var/log/messages`
```

He wished to write a script tracking changes to the system log file, /var/log/messages. Unfortunately, the above code block hangs and does nothing useful. Why? Fix this so it does work. (Hint: rather than [[../advanced-topics/redirecting-code-blocks#^REDIRREF|redirecting the stdin of the loop]], try a [[../basic/special-characters#^PIPEREF|pipe]].)

---

Analyze the following "one-liner" (here split into two lines for clarity) contributed by Rory Winston:

```bash
export SUM=0; for f in $(find src -name "*.java");
do export SUM=$(($SUM + $(wc -l $f | awk '{ print $1 }'))); done; echo $SUM
```

Hint: First, break the script up into bite-sized sections. Then, carefully examine its use of [[operations-and-related-topics.html|double-parentheses]] arithmetic, the [[../commands/internal-commands-and-builtins#^EXPORTREF|export]] command, the [[../commands/complex-commands#^FINDREF|find]] command, the [[../commands/text-processing-commands#^WCREF|wc]] command, and [[./awk#^AWKREF|awk]].

---

Analyze [[./contributed-scripts#^LIFESLOW|Example A-10]], and reorganize it in a simplified and more logical style. See how many of the variables can be eliminated, and try to optimize the script to speed up its execution time.

Alter the script so that it accepts any ordinary ASCII text file as input for its initial "generation". The script will read the first _$ROW*$COL_ characters, and set the occurrences of vowels as "living" cells. Hint: be sure to translate the spaces in the input file to underscore characters.
