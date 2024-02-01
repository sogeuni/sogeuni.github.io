**Example 32-2.** Missing [[internal-commands-and-builtins#^keywordref|keyword]]

```bash
#!/bin/bash
# missing-keyword.sh
# What error message will this script generate? And why?

for a in 1 2 3
do
  echo "$a"
# done     # Required keyword 'done' commented out in line 8.

exit 0     # Will not exit here!

# === #

# From command line, after script terminates:
  echo $?    # 2
```

Output from script:

```bash
missing-keyword.sh: line 10: syntax error: unexpected end of file
	
```
