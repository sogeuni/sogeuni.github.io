---
title: 20.2. Redirecting Code Blocks
---


Blocks of code, such as [[loops-and-branches#^WHILELOOPREF|while]], [[loops-and-branches#^UNTILLOOPREF|until]], and [[loops-and-branches#^FORLOOPREF1|for]] loops, even [[tests#^IFTHEN|if/then]] test blocks can also incorporate redirection of stdin. Even a function may use this form of redirection (see [[Example 24-11|Example 24-11]]). The < operator at the end of the code block accomplishes this.

![[Example 20-5|Example 20-5]]

![[Example 20-6|Example 20-6]]

![[Example 20-7|Example 20-7]]

![[Example 20-8|Example 20-8]]

We can modify the previous example to also redirect the output of the loop.

![[Example 20-9|Example 20-9]]

![[Example 20-10|Example 20-10]]

![[Example 20-11|Example 20-11]]

Redirecting the stdout of a code block has the effect of saving its output to a file. See [[Example 3-2|Example 3-2]].

[[here-documents#^HEREDOCREF|Here documents]] are a special case of redirected code blocks. That being the case, it should be possible to feed the output of a _here document_ into the stdin for a _while loop_.

```bash
# This example by Albert Siersema
# Used with permission (thanks!).

function doesOutput()
 # Could be an external command too, of course.
 # Here we show you can use a function as well.
{
  ls -al *.jpg | awk '{print $5,$9}'
}


nr=0          #  We want the while loop to be able to manipulate these and
totalSize=0   #+ to be able to see the changes after the 'while' finished.

while read fileSize fileName ; do
  echo "$fileName is $fileSize bytes"
  let nr++
  totalSize=$((totalSize+fileSize))   # Or: "let totalSize+=fileSize"
done<<EOF
$(doesOutput)
EOF

echo "$nr files totaling $totalSize bytes"
```
