---
title: 28. Indirect References
---


We have seen that [[varsubn.html|referencing a variable]], $var, fetches its _value_. But, what about the _value of a value_? What about $$var?

The actual notation is _\$$var_, usually preceded by an [[internal.html#EVALREF|eval]] (and sometimes an [[internal.html#ECHOREF|echo]). This is called an _indirect reference_.

![[Example 28-1|Example 28-1]]

> Indirect referencing in Bash is a multi-step process. First, take the name of a variable: varname. Then, reference it: $varname. Then, reference the reference: $$varname. Then, _escape_ the first $: \$$varname. Finally, force a reevaluation of the expression and assign it: **eval newvar=\$$varname**.

Of what practical use is indirect referencing of variables? It gives Bash a little of the functionality of [[varsubn.html#POINTERREF|pointers]] in _C_, for instance, in [[bash-version-2-3-and-4#^RESISTOR|table lookup]]. And, it also has some other very interesting applications. . . .

Nils Radtke shows how to build "dynamic" variable names and evaluate their contents. This can be useful when [[internal.html#SOURCEREF|sourcing]] configuration files.

```bash
#!/bin/bash


# ---------------------------------------------
# This could be "sourced" from a separate file.
isdnMyProviderRemoteNet=172.16.0.100
isdnYourProviderRemoteNet=10.0.0.10
isdnOnlineService="MyProvider"
# ---------------------------------------------
      

remoteNet=$(eval "echo \$$(echo isdn${isdnOnlineService}RemoteNet)")
remoteNet=$(eval "echo \$$(echo isdnMyProviderRemoteNet)")
remoteNet=$(eval "echo \$isdnMyProviderRemoteNet")
remoteNet=$(eval "echo $isdnMyProviderRemoteNet")

echo "$remoteNet"    # 172.16.0.100

# ================================================================

#  And, it gets even better.

#  Consider the following snippet given a variable named getSparc,
#+ but no such variable getIa64:

chkMirrorArchs () { 
  arch="$1";
  if [ "$(eval "echo \${$(echo get$(echo -ne $arch |
       sed 's/^\(.\).*/\1/g' | tr 'a-z' 'A-Z'; echo $arch |
       sed 's/^.\(.*\)/\1/g')):-false}")" = true ]
  then
     return 0;
  else
     return 1;
  fi;
}

getSparc="true"
unset getIa64
chkMirrorArchs sparc
echo $?        # 0
               # True

chkMirrorArchs Ia64
echo $?        # 1
               # False

# Notes:
# -----
# Even the to-be-substituted variable name part is built explicitly.
# The parameters to the chkMirrorArchs calls are all lower case.
# The variable name is composed of two parts: "get" and "Sparc" . . .
```

![[Example 28-2|Example 28-2]]

> [!caution] This method of indirect referencing is a bit tricky. If the second order variable changes its value, then the first order variable must be properly dereferenced (as in the above example). Fortunately, the _${!variable}_ notation introduced with [[bashver2#^BASH2REF|version 2]] of Bash (see [[Example 37-2|Example 37-2]] and [[contributed-scripts.html#HASHEX2|Example A-22]]) makes indirect referencing more intuitive.

> Bash does not support pointer arithmetic, and this severely limits the usefulness of indirect referencing. In fact, indirect referencing in a scripting language is, at best, something of an afterthought.
