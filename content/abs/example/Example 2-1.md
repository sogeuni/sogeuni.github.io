---
title: "Exampla 2-1. cleanup: A script to clean up log files in /var/log"
---

```bash
# Cleanup
# Run as root, of course.

cd /var/log
cat /dev/null > messages
cat /dev/null > wtmp
echo "Log files cleaned up."
```
