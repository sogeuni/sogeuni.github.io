---
title: Example 11-4. Operating on a parameterized file list
---


```bash
#!/bin/bash

filename="*txt"

for file in $filename
do
 echo "Contents of $file"
 echo "---"
 cat "$file"
 echo
done
```
