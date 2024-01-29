---
title: Example 11-12. Symbolic links in a directory, saved to a file
---


```bash
#!/bin/bash
# symlinks.sh: Lists symbolic links in a directory.

OUTFILE=symlinks.list                         # save-file

directory=${1-`pwd`}
#  Defaults to current working directory,
#+ if not otherwise specified.


echo "symbolic links in directory \"$directory\"" > "$OUTFILE"
echo "---------------------------" >> "$OUTFILE"

for file in "$( find $directory -type l )"    # -type l = symbolic links
do
  echo "$file"
done | sort >> "$OUTFILE"                     # stdout of loop
#           ^^^^^^^^^^^^^                       redirected to save file.

# echo "Output file = $OUTFILE"

exit $?
```
