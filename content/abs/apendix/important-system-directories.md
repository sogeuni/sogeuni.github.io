---
title: Appendix I. Important System Directories
---


Sysadmins and anyone else writing administrative scripts should be intimately familiar with the following system directories.

- /bin
    
    Binaries (executables). Basic system programs and utilities (such as **bash**).
    
- /usr/bin [^1]
    
    More system binaries.
    
- /usr/local/bin
    
    Miscellaneous binaries local to the particular machine.
    
- /sbin
    
    System binaries. Basic system administrative programs and utilities (such as **fsck**).
    
- /usr/sbin
    
    More system administrative programs and utilities.
    
- /etc
    
    _Et cetera_. Systemwide configuration scripts.
    
    Of particular interest are the [[system-and-administrative-commands#^FSTABREF|/etc/fstab]] (filesystem table), /etc/mtab (mounted filesystem table), and the [[system-and-administrative-commands#^INITTABREF|/etc/inittab]] files.
    
- /etc/rc.d
    
    Boot scripts, on Red Hat and derivative distributions of Linux.
    
- /usr/share/doc
    
    Documentation for installed packages.
    
- /usr/man
    
    The systemwide [[basic-commands#^MANREF|manpages]].
    
- /dev
    
    Device directory. Entries (but _not_ mount points) for physical and virtual devices. See [[dev-and-proc|Chapter 29]].
    
- /proc
    
    Process directory. Contains information and statistics about running processes and kernel parameters. See [[dev-and-proc|Chapter 29]].
    
- /sys
    
    Systemwide device directory. Contains information and statistics about device and device names. This is newly added to Linux with the 2.6.X kernels.
    
- /mnt
    
    _Mount_. Directory for mounting hard drive partitions, such as /mnt/dos, and physical devices. In newer Linux distros, the /media directory has taken over as the preferred mount point for I/O devices.
    
- /media
    
    In newer Linux distros, the preferred mount point for I/O devices, such as CD/DVD drives or USB flash drives.
    
- /var
    
    _Variable_ (changeable) system files. This is a catchall "scratchpad" directory for data generated while a Linux/UNIX machine is running.
    
- /var/log
    
    Systemwide log files.
    
- /var/spool/mail
    
    User mail spool.
    
- /lib
    
    Systemwide library files.
    
- /usr/lib
    
    More systemwide library files.
    
- /tmp
    
    System temporary files.
    
- /boot
    
    System _boot_ directory. The kernel, module links, system map, and boot manager reside here.
    
> [!warning]
> Altering files in this directory may result in an unbootable system.

[^1]: Some early UNIX systems had a fast, small-capacity fixed disk (containing /, the root partition), and a second drive which was larger, but slower (containing /usr and other partitions). The most frequently used programs and utilities therefore resided on the small-but-fast drive, in /bin, and the others on the slower drive, in /usr/bin.

    This likewise accounts for the split between /sbin and /usr/sbin, /lib and /usr/lib, etc.
