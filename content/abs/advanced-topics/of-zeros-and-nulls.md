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

**Example 31-1.** Hiding the cookie jar

```bash
# Obsolete Netscape browser.
# Same principle applies to newer browsers.

if [ -f ~/.netscape/cookies ]  # Remove, if exists.
then
  rm -f ~/.netscape/cookies
fi

ln -s /dev/null ~/.netscape/cookies
# All cookies now get sent to a black hole, rather than saved to disk.
```

Uses of /dev/zero

Like /dev/null, /dev/zero is a pseudo-device file, but it actually produces a stream of nulls (_binary_ zeros, not the [[special-characters#^ASCIIDEF|ASCII]] kind). Output written to /dev/zero disappears, and it is fairly difficult to actually read the nulls emitted there, though it can be done with [[miscellaneous-commands#^ODREF|od]] or a hex editor. The chief use of /dev/zero is creating an initialized dummy file of predetermined length intended as a temporary swap file.

**Example 31-2.** Setting up a swapfile using /dev/zero

```bash
#!/bin/bash
# Creating a swap file.

#  A swap file provides a temporary storage cache
#+ which helps speed up certain filesystem operations.

ROOT_UID=0         # Root has $UID 0.
E_WRONG_USER=85    # Not root?

FILE=/swap
BLOCKSIZE=1024
MINBLOCKS=40
SUCCESS=0


# This script must be run as root.
if [ "$UID" -ne "$ROOT_UID" ]
then
  echo; echo "You must be root to run this script."; echo
  exit $E_WRONG_USER
fi  
  

blocks=${1:-$MINBLOCKS}          #  Set to default of 40 blocks,
                                 #+ if nothing specified on command-line.
# This is the equivalent of the command block below.
# --------------------------------------------------
# if [ -n "$1" ]
# then
#   blocks=$1
# else
#   blocks=$MINBLOCKS
# fi
# --------------------------------------------------


if [ "$blocks" -lt $MINBLOCKS ]
then
  blocks=$MINBLOCKS              # Must be at least 40 blocks long.
fi  


######################################################################
echo "Creating swap file of size $blocks blocks (KB)."
dd if=/dev/zero of=$FILE bs=$BLOCKSIZE count=$blocks  # Zero out file.
mkswap $FILE $blocks             # Designate it a swap file.
swapon $FILE                     # Activate swap file.
retcode=$?                       # Everything worked?
#  Note that if one or more of these commands fails,
#+ then it could cause nasty problems.
######################################################################

#  Exercise:
#  Rewrite the above block of code so that if it does not execute
#+ successfully, then:
#    1) an error message is echoed to stderr,
#    2) all temporary files are cleaned up, and
#    3) the script exits in an orderly fashion with an
#+      appropriate error code.

echo "Swap file created and activated."

exit $retcode
```

Another application of /dev/zero is to "zero out" a file of a designated size for a special purpose, such as mounting a filesystem on a [[dev#^LOOPBACKREF|loopback device]] (see [[Example 17-8|Example 17-8]]) or "securely" deleting a file (see [[Example 16-61|Example 16-61]]).

**Example 31-3.** Creating a ramdisk

```bash
#!/bin/bash
# ramdisk.sh

#  A "ramdisk" is a segment of system RAM memory
#+ which acts as if it were a filesystem.
#  Its advantage is very fast access (read/write time).
#  Disadvantages: volatility, loss of data on reboot or powerdown,
#+                less RAM available to system.
#
#  Of what use is a ramdisk?
#  Keeping a large dataset, such as a table or dictionary on ramdisk,
#+ speeds up data lookup, since memory access is much faster than disk access.


E_NON_ROOT_USER=70             # Must run as root.
ROOTUSER_NAME=root

MOUNTPT=/mnt/ramdisk           # Create with mkdir /mnt/ramdisk.
SIZE=2000                      # 2K blocks (change as appropriate)
BLOCKSIZE=1024                 # 1K (1024 byte) block size
DEVICE=/dev/ram0               # First ram device

username=`id -nu`
if [ "$username" != "$ROOTUSER_NAME" ]
then
  echo "Must be root to run \"`basename $0`\"."
  exit $E_NON_ROOT_USER
fi

if [ ! -d "$MOUNTPT" ]         #  Test whether mount point already there,
then                           #+ so no error if this script is run
  mkdir $MOUNTPT               #+ multiple times.
fi

##############################################################################
dd if=/dev/zero of=$DEVICE count=$SIZE bs=$BLOCKSIZE  # Zero out RAM device.
                                                      # Why is this necessary?
mke2fs $DEVICE                 # Create an ext2 filesystem on it.
mount $DEVICE $MOUNTPT         # Mount it.
chmod 777 $MOUNTPT             # Enables ordinary user to access ramdisk.
                               # However, must be root to unmount it.
##############################################################################
# Need to test whether above commands succeed. Could cause problems otherwise.
# Exercise: modify this script to make it safer.

echo "\"$MOUNTPT\" now available for use."
# The ramdisk is now accessible for storing files, even by an ordinary user.

#  Caution, the ramdisk is volatile, and its contents will disappear
#+ on reboot or power loss.
#  Copy anything you want saved to a regular directory.

# After reboot, run this script to again set up ramdisk.
# Remounting /mnt/ramdisk without the other steps will not work.

#  Suitably modified, this script can by invoked in /etc/rc.d/rc.local,
#+ to set up ramdisk automatically at bootup.
#  That may be appropriate on, for example, a database server.

exit 0
```

In addition to all the above, `/dev/zero` is needed by ELF (_Executable and Linking Format_) UNIX/Linux binaries.
