**Example 32-1.** A buggy script

```bash
#!/bin/bash
# ex74.sh

# This is a buggy script.
# Where, oh where is the error?

a=37

if [$a -gt 27 ]
then
  echo $a
fi  

exit $?   # 0! Why?
```

Output from script:

```bash
./ex74.sh: [37: command not found
```
