---
title: 29.1. /dev
---


The /dev directory contains entries for the _physical devices_ that may or may not be present in the hardware. [^1] Appropriately enough, these are called _device files_. As an example, the hard drive partitions containing the mounted filesystem(s) have entries in /dev, as [[system-and-administrative-commands#^DFREF|df]] shows.

```bash
bash$ df
Filesystem           1k-blocks      Used Available Use%
 Mounted on
 /dev/hda6               495876    222748    247527  48% /
 /dev/hda1                50755      3887     44248   9% /boot
 /dev/hda8               367013     13262    334803   4% /home
 /dev/hda5              1714416   1123624    503704  70% /usr
	      
```

Among other things, the /dev directory contains _loopback_ devices, such as /dev/loop0. A loopback device is a gimmick that allows an ordinary file to be accessed as if it were a block device. [^2] This permits mounting an entire filesystem within a single large file. See [[Example 17-8|Example 17-8]] and [[Example 17-7|Example 17-7]].

A few of the pseudo-devices in /dev have other specialized uses, such as [[of-zeros-and-nulls#^ZEROSREF|/dev/null]], [[of-zeros-and-nulls#^ZEROSREF1|/dev/zero]], [[another-look-at-variables#^URANDOMREF|/dev/urandom]], /dev/sda1 (hard drive partition), /dev/udp (_User Datagram Packet_ port), and [[dev#^DEVTCP|/dev/tcp]].

For instance:

To manually [[system-and-administrative-commands#^MOUNTREF|mount]] a USB flash drive, append the following line to [[system-and-administrative-commands#^FSTABREF|/etc/fstab]]. [^3]

```bash
/dev/sda1    /mnt/flashdrive    auto    noauto,user,noatime    0 0
```

(See also [[Example A-23|Example A-23]].)

Checking whether a disk is in the CD-burner (soft-linked to /dev/hdc):

```bash
head -1 /dev/hdc


#  head: cannot open '/dev/hdc' for reading: No medium found
#  (No disc in the drive.)

#  head: error reading '/dev/hdc': Input/output error
#  (There is a disk in the drive, but it can't be read;
#+  possibly it's an unrecorded CDR blank.)   

#  Stream of characters and assorted gibberish
#  (There is a pre-recorded disk in the drive,
#+ and this is raw output -- a stream of ASCII and binary data.)
#  Here we see the wisdom of using 'head' to limit the output
#+ to manageable proportions, rather than 'cat' or something similar.


#  Now, it's just a matter of checking/parsing the output and taking
#+ appropriate action.
```

When executing a command on a /dev/tcp/$host/$port pseudo-device file, Bash opens a TCP connection to the associated _socket_.

> A _socket_ is a communications node associated with a specific I/O port. (This is analogous to a _hardware socket_, or _receptacle_, for a connecting cable.) It permits data transfer between hardware devices on the same machine, between machines on the same network, between machines across different networks, and, of course, between machines at different locations on the Internet. ^SOCKETREF

The following examples assume an active Internet connection.

Getting the time from nist.gov:

```bash
bash$ cat </dev/tcp/time.nist.gov/13
53082 04-03-18 04:26:54 68 0 0 502.3 UTC(NIST) *
	      
```

\[Mark contributed this example.]

Generalizing the above into a script:

```bash
#!/bin/bash
# This script must run with root permissions.

URL="time.nist.gov/13"

Time=$(cat </dev/tcp/"$URL")
UTC=$(echo "$Time" | awk '{print$3}')   # Third field is UTC (GMT) time.
# Exercise: modify this for different time zones.

echo "UTC Time = "$UTC""
```

Downloading a URL:

```bash
bash$ exec 5<>/dev/tcp/www.net.cn/80
bash$ echo -e "GET / HTTP/1.0\n" >&5
bash$ cat <&5
```

\[Thanks, Mark and Mihai Maties.]

![[Example 29-1|Example 29-1]]

![[Example 29-2|Example 29-2]]

[^1]: The entries in /dev provide mount points for physical and virtual devices. These entries use very little drive space.
    
    Some devices, such as /dev/null, /dev/zero, and /dev/urandom are virtual. They are not actual physical devices and exist only in software.

[^2]: A _block device_ reads and/or writes data in chunks, or _blocks_, in contrast to a _character device_, which acesses data in _character_ units. Examples of block devices are hard drives, CDROM drives, and flash drives. Examples of character devices are keyboards, modems, sound cards.

[^3]: Of course, the mount point /mnt/flashdrive must exist. If not, then, as _root_, **mkdir /mnt/flashdrive**.
    
    To actually mount the drive, use the following command: **mount /mnt/flashdrive**
    
    Newer Linux distros automount flash drives in the /media directory without user intervention.
