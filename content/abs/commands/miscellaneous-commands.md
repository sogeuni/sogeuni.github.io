---
title: 16.9. Miscellaneous Commands
---


**Command that fit in no special category**

**jot**, **seq**

These utilities emit a sequence of integers, with a user-selectable increment.

The default separator character between each integer is a newline, but this can be changed with the -s option.

```bash
bash$ seq 5
1
 2
 3
 4
 5

bash$ seq -s : 5
1:2:3:4:5
```

Both **jot** and **seq** come in handy in a [[loops-and-branches#^FORLOOPREF1|for loop]].

![[Example 16-54|Example 16-54]]

A simpler example:

```bash
#  Create a set of 10 files,
#+ named file.1, file.2 . . . file.10.
COUNT=10
PREFIX=file

for filename in `seq $COUNT`
do
  touch $PREFIX.$filename
  #  Or, can do other operations,
  #+ such as rm, grep, etc.
done
```

![[Example 16-55|Example 16-55]]

> [!note]
> Somewhat more capable than _seq_, **jot** is a classic UNIX utility that is not normally included in a standard Linux distro. However, the source _rpm_ is available for download from the [MIT repository](http://www.mit.edu/afs/athena/system/rhlinux/athena-9.0/free/SRPMS/athena-jot-9.0-3.src.rpm).
>
> Unlike _seq_, **jot** can generate a sequence of random numbers, using the -r option.
>
> ```bash
> bash$ jot -r 3 999
> 1069
>  1272
>  1428
> ```

**getopt**

The **getopt** command parses command-line options preceded by a [[special-characters#^DASHREF|dash]]. This external command corresponds to the [[internal-commands-and-builtins#^GETOPTSX|getopts]] Bash builtin. Using **getopt** permits handling long options by means of the -l flag, and this also allows parameter reshuffling.

![[Example 16-56|Example 16-56]]

> [!note]
> As _Peggy Russell_ points out:
>
> It is often necessary to include an [[internal-commands-and-builtins#^EVALREF|eval]] to correctly process [[special-characters#Whitespace|whitespace]] and _quotes_.
>
> ```bash
> args=$(getopt -o a:bc:d -- "$@")
> eval set -- "$args"
> ```

See [[Example 10-5|Example 10-5]] for a simplified emulation of **getopt**.

**run-parts**

The **run-parts** command [^1] executes all the scripts in a target directory, sequentially in ASCII-sorted filename order. Of course, the scripts need to have execute permission.

The [[system-and-administrative-commands#^CRONREF|cron]] [[communications-commands#^DAEMONREF|daemon]] invokes **run-parts** to run the scripts in the /etc/cron.* directories.

**yes**

In its default behavior the **yes** command feeds a continuous string of the character y followed by a line feed to stdout. A **control**-**C** terminates the run. A different output string may be specified, as in **yes different string**, which would continually output different string to stdout.

One might well ask the purpose of this. From the command-line or in a script, the output of **yes** can be redirected or piped into a program expecting user input. In effect, this becomes a sort of poor man's version of _expect_.

**yes | fsck /dev/hda1** runs **fsck** non-interactively (careful!).

**yes | rm -r dirname** has same effect as **rm -rf dirname** (careful!).

> [!warning]
> Caution advised when piping _yes_ to a potentially dangerous system command, such as [[system-and-administrative-commands#^FSCKREF|fsck]] or [[system-and-administrative-commands#^FDISKREF|fdisk]]. It might have unintended consequences.

> [!note]
> The _yes_ command parses variables, or more accurately, it echoes parsed variables. For example:
>
> ```bash
> bash$ yes $BASH_VERSION
> 3.1.17(1)-release
>  3.1.17(1)-release
>  3.1.17(1)-release
>  3.1.17(1)-release
>  3.1.17(1)-release
>  . . .
> ```
>
> This particular "feature" may be used to create a _very large_ ASCII file on the fly:
>
> ```bash
> bash$ yes $PATH > huge_file.txt
> **Ctl-C**
> ```
>
> Hit **Ctl-C** _very quickly_, or you just might get more than you bargained for. . . .

The _yes_ command may be emulated in a very simple script [[functions|function]].

```bash
yes ()
{ # Trivial emulation of "yes" ...
  local DEFAULT_TEXT="y"
  while [ true ]   # Endless loop.
  do
    if [ -z "$1" ]
    then
      echo "$DEFAULT_TEXT"
    else           # If argument ...
      echo "$1"    # ... expand and echo it.
    fi
  done             #  The only things missing are the
}                  #+ --help and --version options.
```

**banner**

Prints arguments as a large vertical banner to stdout, using an [[special-characters#^ASCIIDEF|ASCII]] character (default '#'). This may be redirected to a printer for hardcopy.

Note that _banner_ has been dropped from many Linux distros, presumably because it is no longer considered useful.

**printenv**

Show all the [[othertypesv#^ENVREF|environmental variables]] set for a particular user.

```bash
bash$ **printenv | grep HOME**
HOME=/home/bozo
```

**lp**

The **lp** and **lpr** commands send file(s) to the print queue, to be printed as hard copy. [^2] These commands trace the origin of their names to the line printers of another era. [^3]

bash$ **lp file1.txt** or bash **lp <file1.txt**

It is often useful to pipe the formatted output from **pr** to **lp**.

bash$ **pr -options file1.txt | lp**

Formatting packages, such as [[external-filters-programs-and-commands#^GROFFREF|groff]] and _Ghostscript_ may send their output directly to **lp**.

bash$ **groff -Tascii file.tr | lp**

bash$ **gs -options | lp file.ps**

Related commands are **lpq**, for viewing the print queue, and **lprm**, for removing jobs from the print queue.

**tee**

[UNIX borrows an idea from the plumbing trade.]

This is a redirection operator, but with a difference. Like the plumber's _tee,_ it permits "siphoning off" _to a file_ the output of a command or commands within a pipe, but without affecting the result. This is useful for printing an ongoing process to a file or paper, perhaps to keep track of it for debugging purposes.

```bash
(redirection)
                            |----> to file
                            |
  ==========================|====================
  command ---> command ---> |tee ---> command ---> ---> output of pipe
  ===============================================
```

```bash
cat listfile* | sort | tee check.file | uniq > result.file
#                      ^^^^^^^^^^^^^^   ^^^^    

#  The file "check.file" contains the concatenated sorted "listfiles,"
#+ before the duplicate lines are removed by 'uniq.'
```

**mkfifo**

This obscure command creates a _named pipe_, a temporary _first-in-first-out buffer_ for transferring data between processes. [^4] Typically, one process writes to the FIFO, and the other reads from it. See [[Example A-14|Example A-14]].

```bash
#!/bin/bash
# This short script by Omair Eshkenazi.
# Used in ABS Guide with permission (thanks!).

mkfifo pipe1   # Yes, pipes can be given names.
mkfifo pipe2   # Hence the designation "named pipe."

(cut -d' ' -f1 | tr "a-z" "A-Z") >pipe2 <pipe1 &
ls -l | tr -s ' ' | cut -d' ' -f3,9- | tee pipe1 |
cut -d' ' -f2 | paste - pipe2

rm -f pipe1
rm -f pipe2

# No need to kill background processes when script terminates (why not?).

exit $?

Now, invoke the script and explain the output:
sh mkfifo-example.sh

4830.tar.gz          BOZO
pipe1   BOZO
pipe2   BOZO
mkfifo-example.sh    BOZO
Mixed.msg BOZO
```

**pathchk**

This command checks the validity of a filename. If the filename exceeds the maximum allowable length (255 characters) or one or more of the directories in its path is not searchable, then an error message results.

Unfortunately, **pathchk** does not return a recognizable error code, and it is therefore pretty much useless in a script. Consider instead the [[tests#^RTIF|file test operators]].

**dd**

Though this somewhat obscure and much feared **d**ata **d**uplicator command originated as a utility for exchanging data on magnetic tapes between UNIX minicomputers and IBM mainframes, it still has its uses. The **dd** command simply copies a file (or stdin/stdout), but with conversions. Possible conversions include ASCII/EBCDIC, [^5] upper/lower case, swapping of byte pairs between input and output, and skipping and/or truncating the head or tail of the input file.

```bash
# Converting a file to all uppercase:

dd if=$filename conv=ucase > $filename.uppercase
#                    lcase   # For lower case conversion
```

Some basic options to **dd** are:

- if=INFILE
    
    INFILE is the _source_ file.
    
- of=OUTFILE
    
    OUTFILE is the _target_ file, the file that will have the data written to it.
    
- bs=BLOCKSIZE
    
    This is the size of each block of data being read and written, usually a power of 2.
    
- skip=BLOCKS
    
    How many blocks of data to skip in INFILE before starting to copy. This is useful when the INFILE has "garbage" or garbled data in its header or when it is desirable to copy only a portion of the INFILE.
    
- seek=BLOCKS
    
    How many blocks of data to skip in OUTFILE before starting to copy, leaving blank data at beginning of OUTFILE.
    
- count=BLOCKS
    
    Copy only this many blocks of data, rather than the entire INFILE.
    
- conv=CONVERSION
    
    Type of conversion to be applied to INFILE data before copying operation.
    

A **dd --help** lists all the options this powerful utility takes.

![[Example 16-57|Example 16-57]]

![[Example 16-58|Example 16-58]]

To demonstrate just how versatile **dd** is, let's use it to capture keystrokes.

![[Example 16-59|Example 16-59]]

The **dd** command can do random access on a data stream.

```bash
echo -n . | dd bs=1 seek=4 of=file conv=notrunc
#  The "conv=notrunc" option means that the output file
#+ will not be truncated.

# Thanks, S.C.
```

The **dd** command can copy raw data and disk images to and from devices, such as floppies and tape drives ([[Example A-5|Example A-5]]). A common use is creating boot floppies.

**dd if=kernel-image of=/dev/fd0H1440**

Similarly, **dd** can copy the entire contents of a floppy, even one formatted with a "foreign" OS, to the hard drive as an image file.

**dd if=/dev/fd0 of=/home/bozo/projects/floppy.img**

Likewise, **dd** can create bootable flash drives and SD cards.

**dd if=image.iso of=/dev/sdb**

![[Example 16-60|Example 16-60]]

Other applications of **dd** include initializing temporary swap files ([[Example 31-2|Example 31-2]]) and ramdisks ([[Example 31-3|Example 31-3]]). It can even do a low-level copy of an entire hard drive partition, although this is not necessarily recommended.

People (with presumably nothing better to do with their time) are constantly thinking of interesting applications of **dd**.

![[Example 16-61|Example 16-61]]

See also the [[Bibliography#^DDLINK|dd thread]] entry in the [[bibliography.md#^BIBLIOREF|bibliography]].

**od**

The **od**, or _octal dump_ filter converts input (or files) to octal (base-8) or other bases. This is useful for viewing or processing binary data files or otherwise unreadable system [[dev#^DEVFILEREF|device files]], such as /dev/urandom, and as a filter for binary data.

```bash
head -c4 /dev/urandom | od -N4 -tu4 | sed -ne '1s/.* //p'
# Sample output: 1324725719, 3918166450, 2989231420, etc.

# From rnd.sh example script, by Stéphane Chazelas
```

See also [[Example 9-16|Example 9-16]] and [[Example A-36|Example A-36]].

**hexdump**

Performs a hexadecimal, octal, decimal, or ASCII dump of a binary file. This command is the rough equivalent of **od**, above, but not nearly as useful. May be used to view the contents of a binary file, in combination with [[miscellaneous-commands#^DDREF|dd]] and [[file-and-archiving-commands#^LESSREF|less]].

```bash
dd if=/bin/ls | hexdump -C | less
# The -C option nicely formats the output in tabular form.
```

**objdump**

Displays information about an object file or binary executable in either hexadecimal form or as a disassembled listing (with the -d option).

```bash
bash$ **objdump -d /bin/ls**
/bin/ls:     file format elf32-i386

 Disassembly of section .init:

 080490bc <.init>:
  80490bc:       55                      push   %ebp
  80490bd:       89 e5                   mov    %esp,%ebp
  . . .
```

**mcookie**

This command generates a "magic cookie," a 128-bit (32-character) pseudorandom hexadecimal number, normally used as an authorization "signature" by the X server. This also available for use in a script as a "quick 'n dirty" random number.

```bash
random000=$(mcookie)
```

Of course, a script could use [[file-and-archiving-commands#^MD5SUMREF|md5sum]] for the same purpose.

```bash
# Generate md5 checksum on the script itself.
random001=`md5sum $0 | awk '{print $1}'`
# Uses 'awk' to strip off the filename.
```

The **mcookie** command gives yet another way to generate a "unique" filename.

![[Example 16-62|Example 16-62]]

**units**

This utility converts between different _units of measure_. While normally invoked in interactive mode, **units** may find use in a script.

![[Example 16-63|Example 16-63]]

**m4**

A hidden treasure, **m4** is a powerful macro [^6] processing filter, virtually a complete language. Although originally written as a pre-processor for _RatFor_, **m4** turned out to be useful as a stand-alone utility. In fact, **m4** combines some of the functionality of [[internal-commands-and-builtins#^EVALREF|eval]], [[external-filters-programs-and-commands#^TRREF|tr]], and [[awk#^AWKREF|awk]], in addition to its extensive macro expansion facilities.

The April, 2002 issue of [_Linux Journal_](http://www.linuxjournal.com) has a very nice article on **m4** and its uses.

![[Example 16-64|Example 16-64]]

**xmessage**

This X-based variant of [[internal-commands-and-builtins#^ECHOREF|echo]] pops up a message/query window on the desktop.

```bash
xmessage Left click to continue -button okay
```

**zenity**

The [zenity](http://freshmeat.net/projects/zenity) utility is adept at displaying _GTK+_ dialog [[assorted-tips#^WIDGETREF|widgets]] and [[assorted-tips#^ZENITYREF2|very suitable for scripting purposes]].

**doexec**

The **doexec** command enables passing an arbitrary list of arguments to a _binary executable_. In particular, passing _argv[0]_ (which corresponds to [[othertypesv#^POSPARAMREF1|$0]] in a script) lets the executable be invoked by various names, and it can then carry out different sets of actions, according to the name by which it was called. What this amounts to is roundabout way of passing options to an executable.

For example, the /usr/local/bin directory might contain a binary called "aaa". Invoking **doexec /usr/local/bin/aaa list** would _list_ all those files in the current working directory beginning with an "a", while invoking (the same executable with) **doexec /usr/local/bin/aaa delete** would _delete_ those files.

> [!note]
> The various behaviors of the executable must be defined within the code of the executable itself, analogous to something like the following in a shell script:
>
> ```bash
> case `basename $0` in
> "name1" ) do_something;;
> "name2" ) do_something_else;;
> "name3" ) do_yet_another_thing;;
> *       ) bail_out;;
> esac
> ```

**dialog**

The [[assorted-tips#^DIALOGREF|dialog]] family of tools provide a method of calling interactive "dialog" boxes from a script. The more elaborate variations of **dialog** -- **gdialog**, **Xdialog**, and **kdialog** -- actually invoke X-Windows [[assorted-tips#^WIDGETREF|widgets]].

**sox**

The **sox**, or "**so**und e**x**change" command plays and performs transformations on sound files. In fact, the /usr/bin/play executable (now deprecated) is nothing but a shell wrapper for _sox_.

For example, **sox soundfile.wav soundfile.au** changes a WAV sound file into a (Sun audio format) AU sound file.

Shell scripts are ideally suited for batch-processing **sox** operations on sound files. For examples, see the [Linux Radio Timeshift HOWTO](http://osl.iu.edu/~tveldhui/radio/) and the [MP3do Project](http://savannah.nongnu.org/projects/audiodo).

[^1]: This is actually a script adapted from the Debian Linux distribution.

[^2]: The _print queue_ is the group of jobs "waiting in line" to be printed.

[^3]: Large mechanical _line printers_ printed a single line of type at a time onto joined sheets of _greenbar_ paper, to the accompaniment of [a great deal of noise](http://www.columbia.edu/cu/computinghistory/1403.html). The hardcopy thusly printed was referred to as a _printout_.

[^4]: For an excellent overview of this topic, see Andy Vaught's article, [Introduction to Named Pipes](http://www2.linuxjournal.com/lj-issues/issue41/2156.html), in the September, 1997 issue of [_Linux Journal_](http://www.linuxjournal.com).

[^5]: EBCDIC (pronounced "ebb-sid-ick") is an acronym for Extended Binary Coded Decimal Interchange Code, an obsolete IBM data format. A bizarre application of the conv=ebcdic option of **dd** is as a quick 'n easy, but not very secure text file encoder.

    ```bash
    cat $file | dd conv=swab,ebcdic > $file_encrypted
    # Encode (looks like gibberish).		    
    # Might as well switch bytes (swab), too, for a little extra obscurity.

    cat $file_encrypted | dd conv=swab,ascii > $file_plaintext
    # Decode.
    ```

[^6]: A _macro_ is a symbolic constant that expands into a command string or a set of operations on parameters. Simply put, it's a shortcut or abbreviation.
