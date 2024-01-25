---
title: 22. Restricted Shells
---


**Disabled commands in restricted shells**

**.** Running a script or portion of a script in _restricted mode_ disables certain commands that would otherwise be available. This is a security measure intended to limit the privileges of the script user and to minimize possible damage from running the script.

The following commands and actions are disabled:

- Using _cd_ to change the working directory.
- Changing the values of the _$PATH_, _$SHELL_, _$BASH_ENV_, or _$ENV_ [[othertypesv#^ENVREF|environmental variables]].
- Reading or changing the _$SHELLOPTS_, shell environmental options.
- Output redirection.
- Invoking commands containing one or more /'s.
- Invoking [[internal-commands-and-builtins#^EXECREF|exec]] to substitute a different process for the shell.
- Various other commands that would enable monkeying with or attempting to subvert the script for an unintended purpose.
- Getting out of restricted mode within the script.

###### Example 22-1. Running a script in restricted mode

```bash
#!/bin/bash

#  Starting the script with "#!/bin/bash -r"
#+ runs entire script in restricted mode.

echo

echo "Changing directory."
cd /usr/local
echo "Now in `pwd`"
echo "Coming back home."
cd
echo "Now in `pwd`"
echo

# Everything up to here in normal, unrestricted mode.

set -r
# set --restricted    has same effect.
echo "==> Now in restricted mode. <=="

echo
echo

echo "Attempting directory change in restricted mode."
cd ..
echo "Still in `pwd`"

echo
echo

echo "\$SHELL = $SHELL"
echo "Attempting to change shell in restricted mode."
SHELL="/bin/ash"
echo
echo "\$SHELL= $SHELL"

echo
echo

echo "Attempting to redirect output in restricted mode."
ls -l /usr/bin > bin.files
ls -l bin.files    # Try to list attempted file creation effort.

echo

exit 0
```
