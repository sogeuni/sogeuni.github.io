---
title: 11.2. Nested Loops
---


A _nested loop_ is a loop within a loop, an inner loop within the body of an outer one. How this works is that the first pass of the outer loop triggers the inner loop, which executes to completion. Then the second pass of the outer loop triggers the inner loop again. This repeats until the outer loop finishes. Of course, a _break_ within either the inner or outer loop would interrupt this process.

###### Example 11-20. Nested Loop

```bash
#!/bin/bash
# nested-loop.sh: Nested "for" loops.

outer=1             # Set outer loop counter.

# Beginning of outer loop.
for a in 1 2 3 4 5
do
  echo "Pass $outer in outer loop."
  echo "---------------------"
  inner=1           # Reset inner loop counter.

  # ===============================================
  # Beginning of inner loop.
  for b in 1 2 3 4 5
  do
    echo "Pass $inner in inner loop."
    let "inner+=1"  # Increment inner loop counter.
  done
  # End of inner loop.
  # ===============================================

  let "outer+=1"    # Increment outer loop counter. 
  echo              # Space between output blocks in pass of outer loop.
done               
# End of outer loop.

exit 0
```

See [[../advanced-topics/arrays#^BUBBLE|Example 27-11]] for an illustration of nested [[./loops#^WHILELOOPREF|while loops]], and [[../advanced-topics/arrays#^EX68|Example 27-13]] to see a while loop nested inside an [[./loops#^UNTILLOOPREF|until loop]].
