---
title: 31. Of Zeros and Nulls
---


> Faultily faultless, icily regular, splendidly null
>
> Dead perfection; no more.
>
> --<cite>Alfred Lord Tennyson</cite>

**/dev/zero ... /dev/null**

Uses of /dev/null

Think of /dev/null as a _black hole_. It is essentially the equivalent of a write-only file. Everything written to it disappears. Attempts to read or output from it result in nothing. All the same, /dev/null can be quite useful from both the command-line and in scripts.

Suppressing stdout.

```bash
cat $filename >/dev/null
# Contents of the file will not list to stdout.
```

Suppressing stderr (from [[Example 16-3|Example 16-3]]).

```bash
rm $badname 2>/dev/null
#           So error messages [stderr] deep-sixed.
```

Suppressing output from _both_ stdout and stderr.

```bash
cat $filename 2>/dev/null >/dev/null
# If "$filename" does not exist, there will be no error message output.
# If "$filename" does exist, the contents of the file will not list to stdout.
# Therefore, no output at all will result from the above line of code.
#
#  This can be useful in situations where the return code from a command
#+ needs to be tested, but no output is desired.
#
# cat $filename &>/dev/null
#     also works, as Baris Cicek points out.
```

Deleting contents of a file, but preserving the file itself, with all attendant permissions (from [[Example 2-1|Example 2-1]] and [[Example 2-3|Example 2-3]]):

```bash
cat /dev/null > /var/log/messages
#  : > /var/log/messages   has same effect, but does not spawn a new process.

cat /dev/null > /var/log/wtmp
```

Automatically emptying the contents of a logfile (especially good for dealing with those nasty "cookies" sent by commercial Web sites):

![[Example 31-1|Example 31-1]]

Uses of /dev/zero

Like /dev/null, /dev/zero is a pseudo-device file, but it actually produces a stream of nulls (_binary_ zeros, not the [[special-characters#^ASCIIDEF|ASCII]] kind). Output written to /dev/zero disappears, and it is fairly difficult to actually read the nulls emitted there, though it can be done with [[miscellaneous-commands#^ODREF|od]] or a hex editor. The chief use of /dev/zero is creating an initialized dummy file of predetermined length intended as a temporary swap file.

![[Example 31-2|Example 31-2]]

Another application of /dev/zero is to "zero out" a file of a designated size for a special purpose, such as mounting a filesystem on a [[dev#^LOOPBACKREF|loopback device]] (see [[Example 17-8|Example 17-8]]) or "securely" deleting a file (see [[Example 16-61|Example 16-61]]).

![[Example 31-3|Example 31-3]]

In addition to all the above, `/dev/zero` is needed by ELF (_Executable and Linking Format_) UNIX/Linux binaries.
