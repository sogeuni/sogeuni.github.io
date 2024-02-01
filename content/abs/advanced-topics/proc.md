---
title: 29.2. /proc
---


The /proc directory is actually a pseudo-filesystem. The files in /proc mirror currently running system and kernel [[special-characters#^PROCESSREF|processes]] and contain information and statistics about them.

```bash
bash$ cat /proc/devices
Character devices:
   1 mem
   2 pty
   3 ttyp
   4 ttyS
   5 cua
   7 vcs
  10 misc
  14 sound
  29 fb
  36 netlink
 128 ptm
 136 pts
 162 raw
 254 pcmcia

 Block devices:
   1 ramdisk
   2 fd
   3 ide0
   9 md



bash$ cat /proc/interrupts
           CPU0       
   0:      84505          XT-PIC  timer
   1:       3375          XT-PIC  keyboard
   2:          0          XT-PIC  cascade
   5:          1          XT-PIC  soundblaster
   8:          1          XT-PIC  rtc
  12:       4231          XT-PIC  PS/2 Mouse
  14:     109373          XT-PIC  ide0
 NMI:          0 
 ERR:          0


bash$ cat /proc/partitions
major minor  #blocks  name     rio rmerge rsect ruse wio wmerge wsect wuse running use aveq

    3     0    3007872 hda 4472 22260 114520 94240 3551 18703 50384 549710 0 111550 644030
    3     1      52416 hda1 27 395 844 960 4 2 14 180 0 800 1140
    3     2          1 hda2 0 0 0 0 0 0 0 0 0 0 0
    3     4     165280 hda4 10 0 20 210 0 0 0 0 0 210 210
    ...



bash$ cat /proc/loadavg
0.13 0.42 0.27 2/44 1119



bash$ cat /proc/apm
1.16 1.2 0x03 0x01 0xff 0x80 -1% -1 ?



bash$ cat /proc/acpi/battery/BAT0/info
present:                 yes
 design capacity:         43200 mWh
 last full capacity:      36640 mWh
 battery technology:      rechargeable
 design voltage:          10800 mV
 design capacity warning: 1832 mWh
 design capacity low:     200 mWh
 capacity granularity 1:  1 mWh
 capacity granularity 2:  1 mWh
 model number:            IBM-02K6897
 serial number:            1133
 battery type:            LION
 OEM info:                Panasonic
 
 
 
bash$ fgrep Mem /proc/meminfo
MemTotal:       515216 kB
 MemFree:        266248 kB
         
```

Shell scripts may extract data from certain of the files in /proc. [^1]

```bash
FS=iso                       # ISO filesystem support in kernel?

grep $FS /proc/filesystems   # iso9660
```

```bash
kernel_version=$( awk '{ print $3 }' /proc/version )
```

```bash
CPU=$( awk '/model name/ {print $5}' < /proc/cpuinfo )

if [ "$CPU" = "Pentium(R)" ]
then
  run_some_commands
  ...
else
  run_other_commands
  ...
fi



cpu_speed=$( fgrep "cpu MHz" /proc/cpuinfo | awk '{print $4}' )
#  Current operating speed (in MHz) of the cpu on your machine.
#  On a laptop this may vary, depending on use of battery
#+ or AC power.
```

```bash
#!/bin/bash
# get-commandline.sh
# Get the command-line parameters of a process.

OPTION=cmdline

# Identify PID.
pid=$( echo $(pidof "$1") | awk '{ print $1 }' )
# Get only first            ^^^^^^^^^^^^^^^^^^ of multiple instances.

echo
echo "Process ID of (first instance of) "$1" = $pid"
echo -n "Command-line arguments: "
cat /proc/"$pid"/"$OPTION" | xargs -0 echo
#   Formats output:        ^^^^^^^^^^^^^^^
#   (Thanks, Han Holl, for the fixup!)

echo; echo


# For example:
# sh get-commandline.sh xterm
```

+

```bash
devfile="/proc/bus/usb/devices"
text="Spd"
USB1="Spd=12"
USB2="Spd=480"


bus_speed=$(fgrep -m 1 "$text" $devfile | awk '{print $9}')
#                 ^^^^ Stop after first match.

if [ "$bus_speed" = "$USB1" ]
then
  echo "USB 1.1 port found."
  # Do something appropriate for USB 1.1.
fi
```

> [!note]
> It is even possible to control certain peripherals with commands sent to the /proc directory.
>
> ```bash
	  root# echo on > /proc/acpi/ibm/light
          
> ```
>
> This turns on the _Thinklight_ in certain models of IBM/Lenovo Thinkpads. (May not work on all Linux distros.)
>
> Of course, caution is advised when writing to /proc.

The /proc directory contains subdirectories with unusual numerical names. Every one of these names maps to the [[another-look-at-variables#^PPIDREF|process ID]] of a currently running process. Within each of these subdirectories, there are a number of files that hold useful information about the corresponding process. The stat and status files keep running statistics on the process, the cmdline file holds the command-line arguments the process was invoked with, and the exe file is a symbolic link to the complete path name of the invoking process. There are a few more such files, but these seem to be the most interesting from a scripting standpoint.

![[Example 29-3|Example 29-3]]

![[Example 29-4|Example 29-4]]

> [!warning]
> In general, it is dangerous to _write_ to the files in /proc, as this can corrupt the filesystem or crash the machine.

[^1]: Certain system commands, such as [[system-and-administrative-commands#^PROCINFOREF|procinfo]], [[system-and-administrative-commands#^FREEREF|free]], [[system-and-administrative-commands#^VMSTATREF|vmstat]], [[system-and-administrative-commands#^LSDEVREF|lsdev]], and [[system-and-administrative-commands#^UPTIMEREF|uptime]] do this as well.
