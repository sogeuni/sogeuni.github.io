---
title: 16.5. File and Archiving Commands
---


**Archiving**

**tar**

The standard UNIX archiving utility. [^1] Originally a _Tape ARchiving_ program, it has developed into a general purpose package that can handle all manner of archiving with all types of destination devices, ranging from tape drives to regular files to even stdout (see [[Example 3-4|Example 3-4]]). GNU _tar_ has been patched to accept various compression filters, for example: **tar czvf archive_name.tar.gz ***, which recursively archives and [[file-and-archiving-commands#^GZIPREF|gzips]] all files in a directory tree except [[external-filters-programs-and-commands#^DOTFILESREF|dotfiles]] in the current working directory ([[another-look-at-variables#^PWDREF|$PWD]]). [^2]

Some useful **tar** options:

1. -c create (a new archive)
2. -x extract (files from existing archive)
3. --delete delete (files from existing archive)
    > [!caution] This option will not work on magnetic tape devices.
4. -r append (files to existing archive)
5. -A append (_tar_ files to existing archive)
6. -t list (contents of existing archive)
7. -u update archive
8. -d compare archive with specified filesystem
9. --after-date only process files with a date stamp _after_ specified date
10. -z [[file-and-archiving-commands#^GZIPREF|gzip]] the archive
    (compress or uncompress, depending on whether combined with the -c or -x) option
11. -j [[file-and-archiving-commands#^BZIPREF|bzip2]] the archive
    > [!caution] It may be difficult to recover data from a corrupted _gzipped_ tar archive. When archiving important files, make multiple backups.

**shar**

_Shell archiving_ utility. The text and/or binary files in a shell archive are concatenated without compression, and the resultant archive is essentially a shell script, complete with #!/bin/sh header, containing all the necessary unarchiving commands, as well as the files themselves. Unprintable binary characters in the target file(s) are converted to printable ASCII characters in the output _shar_ file. _Shar archives_ still show up in Usenet newsgroups, but otherwise **shar** has been replaced by **tar**/**gzip**. The **unshar** command unpacks _shar_ archives.

The **mailshar** command is a Bash script that uses **shar** to concatenate multiple files into a single one for e-mailing. This script supports compression and [[file-and-archiving-commands#^UUENCODEREF|uuencoding]].

**ar**

Creation and manipulation utility for archives, mainly used for binary object file libraries.

**rpm**

The _Red Hat Package Manager_, or **rpm** utility provides a wrapper for source or binary archives. It includes commands for installing and checking the integrity of packages, among other things.

A simple **rpm -i package_name.rpm** usually suffices to install a package, though there are many more options available.

> [!tip]
> **rpm -qf** identifies which package a file originates from.
>
> ```bash
> bash$ rpm -qf /bin/ls
> coreutils-5.2.1-31
> ```

> [!tip]
> **rpm -qa** gives a complete list of all installed _rpm_ packages on a given system. An **rpm -qa package_name** lists only the package(s) corresponding to package_name.
>
> ```bash
> bash$ rpm -qa
> redhat-logos-1.1.3-1
>  glibc-2.2.4-13
>  cracklib-2.7-12
>  dosfstools-2.7-1
>  gdbm-1.8.0-10
>  ksymoops-2.4.1-1
>  mktemp-1.5-11
>  perl-5.6.0-17
>  reiserfs-utils-3.x.0j-2
>  ...
> 
> bash$ **rpm -qa docbook-utils**
> docbook-utils-0.6.9-2
> 
> bash$ rpm -qa docbook | grep docbook
> docbook-dtd31-sgml-1.0-10
>  docbook-style-dsssl-1.64-3
>  docbook-dtd30-sgml-1.0-10
>  docbook-dtd40-sgml-1.0-11
>  docbook-utils-pdf-0.6.9-2
>  docbook-dtd41-sgml-1.0-10
>  docbook-utils-0.6.9-2
> ```

**cpio**

This specialized archiving copy command (**c**o**p**y **i**nput and **o**utput) is rarely seen any more, having been supplanted by **tar**/**gzip**. It still has its uses, such as moving a directory tree. With an appropriate block size (for copying) specified, it can be appreciably faster than **tar**.

![[Example 16-30|Example 16-30]]

**rpm2cpio**

This command extracts a **cpio** archive from an [[file-and-archiving-commands#^RPMREF|rpm]] one.

![[Example 16-31|Example 16-31]]

**pax**

The _pax_ **p**ortable **a**rchive e**x**change toolkit facilitates periodic file backups and is designed to be cross-compatible between various flavors of UNIX. It was designed to replace [[file-and-archiving-commands#^TARREF|tar]] and [[file-and-archiving-commands#^CPIOREF|cpio]].

```bash
pax -wf daily_backup.pax ~/linux-server/files 
#  Creates a tar archive of all files in the target directory.
#  Note that the options to pax must be in the correct order --
#+ pax -fw     has an entirely different effect.

pax -f daily_backup.pax
#  Lists the files in the archive.

pax -rf daily_backup.pax ~/bsd-server/files
#  Restores the backed-up files from the Linux machine
#+ onto a BSD one.
```

Note that _pax_ handles many of the standard archiving and compression commands.

**Compression**

**gzip**

The standard GNU/UNIX compression utility, replacing the inferior and proprietary **compress**. The corresponding decompression command is **gunzip**, which is the equivalent of **gzip -d**.

> [!note]
> The -c option sends the output of **gzip** to stdout. This is useful when [[special-characters#^PIPEREF|piping]] to other commands.

The **zcat** filter decompresses a _gzipped_ file to stdout, as possible input to a pipe or redirection. This is, in effect, a **cat** command that works on compressed files (including files processed with the older [[file-and-archiving-commands#^COMPRESSREF|compress]] utility). The **zcat** command is equivalent to **gzip -dc**.

> [!caution]
> On some commercial UNIX systems, **zcat** is a synonym for **uncompress -c**, and will not work on _gzipped_ files.

See also [[Example 7-7|Example 7-7]].

**bzip2**

An alternate compression utility, usually more efficient (but slower) than **gzip**, especially on large files. The corresponding decompression command is **bunzip2**.

Similar to the **zcat** command, **bzcat** decompresses a _bzipped2-ed_ file to stdout.

> [!note]
> Newer versions of [[file-and-archiving-commands#^TARREF|tar]] have been patched with **bzip2** support.

**compress**, **uncompress**

This is an older, proprietary compression utility found in commercial UNIX distributions. The more efficient **gzip** has largely replaced it. Linux distributions generally include a **compress** workalike for compatibility, although **gunzip** can unarchive files treated with **compress**.

> [!tip]
> The **znew** command transforms _compressed_ files into _gzipped_ ones.

**sq**

Yet another compression (**sq**ueeze) utility, a filter that works only on sorted [[special-characters#^ASCIIDEF|ASCII]] word lists. It uses the standard invocation syntax for a filter, **sq < input-file > output-file**. Fast, but not nearly as efficient as [[file-and-archiving-commands#^GZIPREF|gzip]]. The corresponding uncompression filter is **unsq**, invoked like **sq**.

> [!tip]
> The output of **sq** may be piped to **gzip** for further compression.

**zip**, **unzip**

Cross-platform file archiving and compression utility compatible with DOS _pkzip.exe_. "Zipped" archives seem to be a more common medium of file exchange on the Internet than "tarballs."

**unarc**, **unarj**, **unrar**

These Linux utilities permit unpacking archives compressed with the DOS _arc.exe_, _arj.exe_, and _rar.exe_ programs.

**lzma**, **unlzma**, **lzcat**

Highly efficient Lempel-Ziv-Markov compression. The syntax of _lzma_ is similar to that of _gzip_. The [7-zip Website](http://www.7-zip.org/sdk.html) has more information.

**xz**, **unxz**, **xzcat**

A new high-efficiency compression tool, backward compatible with _lzma_, and with an invocation syntax similar to _gzip_. For more information, see the [Wikipedia entry](http://en.wikipedia.org/wiki/Xz).

**File Information**

**file**

A utility for identifying file types. The command **file file-name** will return a file specification for file-name, such as ascii text or data. It references the [[sha-bang#^magnumref|magic numbers]] found in /usr/share/magic, /etc/magic, or /usr/lib/magic, depending on the Linux/UNIX distribution.

The -f option causes **file** to run in [[external-filters-programs-and-commands#^BATCHPROCREF|batch]] mode, to read from a designated file a list of filenames to analyze. The -z option, when used on a compressed target file, forces an attempt to analyze the uncompressed file type.

```bash
bash$ file test.tar.gz
test.tar.gz: gzip compressed data, deflated,
 last modified: Sun Sep 16 13:34:51 2001, os: Unix

bash file -z test.tar.gz
test.tar.gz: GNU tar archive (gzip compressed data, deflated,
 last modified: Sun Sep 16 13:34:51 2001, os: Unix)
```

```bash
# Find sh and Bash scripts in a given directory:

DIRECTORY=/usr/local/bin
KEYWORD=Bourne
# Bourne and Bourne-Again shell scripts

file $DIRECTORY/* | fgrep $KEYWORD

# Output:

# /usr/local/bin/burn-cd:          Bourne-Again shell script text executable
# /usr/local/bin/burnit:           Bourne-Again shell script text executable
# /usr/local/bin/cassette.sh:      Bourne shell script text executable
# /usr/local/bin/copy-cd:          Bourne-Again shell script text executable
# . . .
```

![[Example 16-32|Example 16-32]]

**which**

**which command** gives the full path to "command." This is useful for finding out whether a particular command or utility is installed on the system.

**$bash which rm**

```bash
/usr/bin/rm
```

For an interesting use of this command, see [[Example 36-16|Example 36-16]].

**whereis**

Similar to **which**, above, **whereis command** gives the full path to "command," but also to its [[external-filters-programs-and-commands#^MANREF|manpage]].

**$bash whereis rm**

```bash
rm: /bin/rm /usr/share/man/man1/rm.1.bz2
```

**whatis**

**whatis command** looks up "command" in the _whatis_ database. This is useful for identifying system commands and important configuration files. Consider it a simplified **man** command.

**$bash whatis whatis**

```bash
whatis               (1)  - search the whatis database for complete words
```

![[Example 16-33|Example 16-33]]

See also [[Example 11-3|Example 11-3]].

**vdir**

Show a detailed directory listing. The effect is similar to [[external-filters-programs-and-commands#^LSREF|ls -lb]].

This is one of the GNU _fileutils_.

```bash
bash$ vdir
total 10
 -rw-r--r--    1 bozo  bozo      4034 Jul 18 22:04 data1.xrolo
 -rw-r--r--    1 bozo  bozo      4602 May 25 13:58 data1.xrolo.bak
 -rw-r--r--    1 bozo  bozo       877 Dec 17  2000 employment.xrolo

bash ls -l
total 10
 -rw-r--r--    1 bozo  bozo      4034 Jul 18 22:04 data1.xrolo
 -rw-r--r--    1 bozo  bozo      4602 May 25 13:58 data1.xrolo.bak
 -rw-r--r--    1 bozo  bozo       877 Dec 17  2000 employment.xrolo
```

**locate**, **slocate**

The **locate** command searches for files using a database stored for just that purpose. The **slocate** command is the secure version of **locate** (which may be aliased to **slocate**).

**$bash locate hickson**

```bash
/usr/lib/xephem/catalogs/hickson.edb
```

**getfacl**, **setfacl**

These commands _retrieve_ or _set_ the **f**ile **a**ccess **c**ontrol **l**ist -- the _owner_, _group_, and file permissions.

```bash
bash$ getfacl *
# file: test1.txt
 # owner: bozo
 # group: bozgrp
 user::rw-
 group::rw-
 other::r--

 # file: test2.txt
 # owner: bozo
 # group: bozgrp
 user::rw-
 group::rw-
 other::r--
 

 
bash$ **setfacl -m u:bozo:rw yearly_budget.csv**
bash$ **getfacl yearly_budget.csv**
# file: yearly_budget.csv
 # owner: accountant
 # group: budgetgrp
 user::rw-
 user:bozo:rw-
 user:accountant:rw-
 group::rw-
 mask::rw-
 other::r--
```

**readlink**

Disclose the file that a symbolic link points to.

```bash
bash$ readlink /usr/bin/awk
../../bin/gawk
```

**strings**

Use the **strings** command to find printable strings in a binary or data file. It will list sequences of printable characters found in the target file. This might be handy for a quick 'n dirty examination of a core dump or for looking at an unknown graphic image file (**strings image-file | more** might show something like _JFIF_, which would identify the file as a _jpeg_ graphic). In a script, you would probably parse the output of **strings** with [[textproc#^GREPREF|grep]] or [[sedawk#^SEDREF|sed]]. See [[Example 11-8|Example 11-8]] and [[Example 11-10|Example 11-10]].

![[Example 16-34|Example 16-34]]

**Comparison**

**diff**, **patch**

**diff**: flexible file comparison utility. It compares the target files line-by-line sequentially. In some applications, such as comparing word dictionaries, it may be helpful to filter the files through [[external-filters-programs-and-commands#^SORTREF|sort]] and **uniq** before piping them to **diff**. **diff file-1 file-2** outputs the lines in the files that differ, with carets showing which file each particular line belongs to.

The --side-by-side option to **diff** outputs each compared file, line by line, in separate columns, with non-matching lines marked. The -c and -u options likewise make the output of the command easier to interpret.

There are available various fancy frontends for **diff**, such as **sdiff**, **wdiff**, **xdiff**, and **mgdiff**.

> [!tip]
> The **diff** command returns an exit status of 0 if the compared files are identical, and 1 if they differ (or 2 when _binary_ files are being compared). This permits use of **diff** in a test construct within a shell script (see below).

A common use for **diff** is generating difference files to be used with **patch** The -e option outputs files suitable for **ed** or **ex** scripts.

**patch**: flexible versioning utility. Given a difference file generated by **diff**, **patch** can upgrade a previous version of a package to a newer version. It is much more convenient to distribute a relatively small "diff" file than the entire body of a newly revised package. Kernel "patches" have become the preferred method of distributing the frequent releases of the Linux kernel.

```bash
patch -p1 <patch-file
# Takes all the changes listed in 'patch-file'
# and applies them to the files referenced therein.
# This upgrades to a newer version of the package.
```

Patching the kernel:

```bash
cd /usr/src
gzip -cd patchXX.gz | patch -p0
# Upgrading kernel source using 'patch'.
# From the Linux kernel docs "README",
# by anonymous author (Alan Cox?).
```

> [!note]
> The **diff** command can also recursively compare directories (for the filenames present).
>
> ```bash
> bash$ diff -r ~/notes1 ~/notes2
> Only in /home/bozo/notes1: file02
>  Only in /home/bozo/notes1: file03
>  Only in /home/bozo/notes2: file04
> ```

> [!tip]
> Use **zdiff** to compare _gzipped_ files.

> [!tip]
> Use **diffstat** to create a histogram (point-distribution graph) of output from **diff**.

**diff3**, **merge**

An extended version of **diff** that compares three files at a time. This command returns an exit value of 0 upon successful execution, but unfortunately this gives no information about the results of the comparison.

```bash
bash$ diff3 file-1 file-2 file-3
====
 1:1c
   This is line 1 of "file-1".
 2:1c
   This is line 1 of "file-2".
 3:1c
   This is line 1 of "file-3"
```

The **merge** (3-way file merge) command is an interesting adjunct to _diff3_. Its syntax is **merge Mergefile file1 file2**. The result is to output to Mergefile the changes that lead from file1 to file2. Consider this command a stripped-down version of _patch_.

**sdiff**

Compare and/or edit two files in order to merge them into an output file. Because of its interactive nature, this command would find little use in a script.

**cmp**

The **cmp** command is a simpler version of **diff**, above. Whereas **diff** reports the differences between two files, **cmp** merely shows at what point they differ.

> [!note]
> Like **diff**, **cmp** returns an exit status of 0 if the compared files are identical, and 1 if they differ. This permits use in a test construct within a shell script.

![[Example 16-35|Example 16-35]]

> [!tip]
> Use **zcmp** on _gzipped_ files.

**comm**

Versatile file comparison utility. The files must be sorted for this to be useful.

**comm _-options_ _first-file_ _second-file_**

**comm file-1 file-2** outputs three columns:

- column 1 = lines unique to file-1
- column 2 = lines unique to file-2
- column 3 = lines common to both.

The options allow suppressing output of one or more columns.

- -1 suppresses column 1
- -2 suppresses column 2
- -3 suppresses column 3
- -12 suppresses both columns 1 and 2, etc.

This command is useful for comparing "dictionaries" or _word lists_ -- sorted text files with one word per line.

**Utilities**

**basename**

Strips the path information from a file name, printing only the file name. The construction **basename $0** lets the script know its name, that is, the name it was invoked by. This can be used for "usage" messages if, for example a script is called with missing arguments:

```bash
echo "Usage: `basename $0` arg1 arg2 ... argn"
```

**dirname**

Strips the **basename** from a filename, printing only the path information.

> [!note]
> **basename** and **dirname** can operate on any arbitrary string. The argument does not need to refer to an existing file, or even be a filename for that matter (see [[Example A-7|Example A-7]]).

![[Example 16-36|Example 16-36]]

**split**, **csplit**

These are utilities for splitting a file into smaller chunks. Their usual use is for splitting up large files in order to back them up on floppies or preparatory to e-mailing or uploading them.

The **csplit** command splits a file according to _context_, the split occuring where patterns are matched.

![[Example 16-37|Example 16-37]]

**Encoding and Encryption**

**sum**, **cksum**, **md5sum**, **sha1sum**

These are utilities for generating _checksums_. A _checksum_ is a number [^3] mathematically calculated from the contents of a file, for the purpose of checking its integrity. A script might refer to a list of checksums for security purposes, such as ensuring that the contents of key system files have not been altered or corrupted. For security applications, use the **md5sum** (**m**essage **d**igest **5** check**sum**) command, or better yet, the newer **sha1sum** (Secure Hash Algorithm). [^4]

```bash
bash$ cksum /boot/vmlinuz
1670054224 804083 /boot/vmlinuz

bash$ echo -n "Top Secret" | cksum
3391003827 10

bash$ md5sum /boot/vmlinuz
0f43eccea8f09e0a0b2b5cf1dcf333ba  /boot/vmlinuz

bash$ echo -n "Top Secret" | md5sum
8babc97a6f62a4649716f4df8d61728f  -
```

> [!note]
> The **cksum** command shows the size, in bytes, of its target, whether file or stdout.
>
> The **md5sum** and **sha1sum** commands display a [[special-characters#^DASHREF2|dash]] when they receive their input from stdout.

![[Example 16-38|Example 16-38]]

Also see [[Example A-19|Example A-19]], [[Example 36-16|Example 36-16]], and [[Example 10-2|Example 10-2]] for creative uses of the **md5sum** command.

> [!note]
> There have been reports that the 128-bit **md5sum** can be cracked, so the more secure 160-bit **sha1sum** is a welcome new addition to the checksum toolkit.
>
> ```bash
> bash$ md5sum testfile
> e181e2c8720c60522c4c4c981108e367  testfile
> 
> bash$ sha1sum testfile
> 5d7425a9c08a66c3177f1e31286fa40986ffc996  testfile
> ```

Security consultants have demonstrated that even **sha1sum** can be compromised. Fortunately, newer Linux distros include longer bit-length **sha224sum**, **sha256sum**, **sha384sum**, and **sha512sum** commands.

**uuencode**

This utility encodes binary files (images, sound files, compressed files, etc.) into [[special-characters#^ASCIIDEF|ASCII]] characters, making them suitable for transmission in the body of an e-mail message or in a newsgroup posting. This is especially useful where MIME (multimedia) encoding is not available.

**uudecode**

This reverses the encoding, decoding _uuencoded_ files back into the original binaries.

![[Example 16-39|Example 16-39]]

> [!tip]
> The [[external-filters-programs-and-commands#^FOLDREF|fold -s]] command may be useful (possibly in a pipe) to process long uudecoded text messages downloaded from Usenet newsgroups.

**mimencode**, **mmencode**

The **mimencode** and **mmencode** commands process multimedia-encoded e-mail attachments. Although _mail user agents_ (such as _pine_ or _kmail_) normally handle this automatically, these particular utilities permit manipulating such attachments manually from the command-line or in [[external-filters-programs-and-commands#^BATCHPROCREF|batch processing mode]] by means of a shell script.

**crypt**

At one time, this was the standard UNIX file encryption utility. [^5] Politically-motivated government regulations prohibiting the export of encryption software resulted in the disappearance of **crypt** from much of the UNIX world, and it is still missing from most Linux distributions. Fortunately, programmers have come up with a number of decent alternatives to it, among them the author's very own [cruft](ftp://metalab.unc.edu/pub/Linux/utils/file/cruft-0.2.tar.gz) (see [[Example A-4|Example A-4]]).

**openssl**

This is an Open Source implementation of _Secure Sockets Layer_ encryption.

```bash
# To encrypt a file:
openssl aes-128-ecb -salt -in file.txt -out file.encrypted \
-pass pass:my_password
#          ^^^^^^^^^^^   User-selected password.
#       aes-128-ecb      is the encryption method chosen.

# To decrypt an openssl-encrypted file:
openssl aes-128-ecb -d -salt -in file.encrypted -out file.txt \
-pass pass:my_password
#          ^^^^^^^^^^^   User-selected password.
```

[[special-characters#^PIPEREF|Piping]] _openssl_ to/from [[file-and-archiving-commands#^TARREF|tar]] makes it possible to encrypt an entire directory tree.

```bash
# To encrypt a directory:

sourcedir="/home/bozo/testfiles"
encrfile="encr-dir.tar.gz"
password=my_secret_password

tar czvf - "$sourcedir" |
openssl des3 -salt -out "$encrfile" -pass pass:"$password"
#       ^^^^   Uses des3 encryption.
# Writes encrypted file "encr-dir.tar.gz" in current working directory.

# To decrypt the resulting tarball:
openssl des3 -d -salt -in "$encrfile" -pass pass:"$password" |
tar -xzv
# Decrypts and unpacks into current working directory.
```

Of course, _openssl_ has many other uses, such as obtaining signed _certificates_ for Web sites. See the [[external-filters-programs-and-commands#^INFOREF|info]] page.

**shred**

Securely erase a file by overwriting it multiple times with random bit patterns before deleting it. This command has the same effect as [[Example 16-61|Example 16-61]], but does it in a more thorough and elegant manner.

This is one of the GNU _fileutils_.

> [!caution]
> Advanced forensic technology may still be able to recover the contents of a file, even after application of **shred**.

**Miscellaneous**

**mktemp**

Create a _temporary file_ [^6] with a "unique" filename. When invoked from the command-line without additional arguments, it creates a zero-length file in the /tmp directory.

```bash
bash$ mktemp
/tmp/tmp.zzsvql3154
```

```bash
PREFIX=filename
tempfile=`mktemp $PREFIX.XXXXXX`
#                        ^^^^^^ Need at least 6 placeholders
#+                              in the filename template.
#   If no filename template supplied,
#+ "tmp.XXXXXXXXXX" is the default.

echo "tempfile name = $tempfile"
# tempfile name = filename.QA2ZpY
#                 or something similar...

#  Creates a file of that name in the current working directory
#+ with 600 file permissions.
#  A "umask 177" is therefore unnecessary,
#+ but it's good programming practice nevertheless.
```

**make**

Utility for building and compiling binary packages. This can also be used for any set of operations triggered by incremental changes in source files.

The _make_ command checks a Makefile, a list of file dependencies and operations to be carried out.

The _make_ utility is, in effect, a powerful scripting language similar in many ways to _Bash_, but with the capability of recognizing _dependencies_. For in-depth coverage of this useful tool set, see the [GNU software documentation site](http://www.gnu.org/manual/manual.html).

**install**

Special purpose file copying command, similar to [[external-filters-programs-and-commands#^CPREF|cp]], but capable of setting permissions and attributes of the copied files. This command seems tailormade for installing software packages, and as such it shows up frequently in Makefiles (in the _make install :_ section). It could likewise prove useful in installation scripts.

**dos2unix**

This utility, written by Benjamin Lin and collaborators, converts DOS-formatted text files (lines terminated by CR-LF) to UNIX format (lines terminated by LF only), and [[gotchas#^DOSNEWLINES|vice-versa]].

**ptx**

The **ptx [targetfile]** command outputs a permuted index (cross-reference list) of the targetfile. This may be further filtered and formatted in a pipe, if necessary.

**more**, **less**

Pagers that display a text file or stream to stdout, one screenful at a time. These may be used to filter the output of stdout . . . or of a script.

An interesting application of _more_ is to "test drive" a command sequence, to forestall potentially unpleasant consequences.

```bash
ls /home/bozo | awk '{print "rm -rf " $1}' | more
#                                            ^^^^
		 
# Testing the effect of the following (disastrous) command-line:
#      ls /home/bozo | awk '{print "rm -rf " $1}' | sh
#      Hand off to the shell to execute . . .       ^^
```

The _less_ pager has the interesting property of doing a formatted display of _man page_ source. See [[Example A-39|Example A-39]].

[^1]: An _archive_, in the sense discussed here, is simply a set of related files stored in a single location.

[^2]: A _tar czvf ArchiveName.tar.gz *_ _will_ include dotfiles in subdirectories _below_ the current working directory. This is an undocumented GNU **tar** "feature."

[^3]: The checksum may be expressed as a _hexadecimal_ number, or to some other base.

[^4]: For even _better_ security, use the _sha256sum_, _sha512_, and _sha1pass_ commands.

[^5]: This is a symmetric block cipher, used to encrypt files on a single system or local network, as opposed to the _public key_ cipher class, of which _pgp_ is a well-known example.

[^6]: Creates a temporary _directory_ when invoked with the -d option.
