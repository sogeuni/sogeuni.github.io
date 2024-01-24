---
title: G.1. Standard Command-Line Options
---


Over time, there has evolved a loose standard for the meanings of command-line option flags. The GNU utilities conform more closely to this "standard" than older UNIX utilities.

Traditionally, UNIX command-line options consist of a dash, followed by one or more lowercase letters. The GNU utilities added a double-dash, followed by a complete word or compound word.

The two most widely-accepted options are:

- -h
    
    --help
    
    _Help_: Give usage message and exit.
    
- -v
    
    --version
    
    _Version_: Show program version and exit.
    

Other common options are:

- -a
    
    --all
    
    _All_: show _all_ information or operate on _all_ arguments.
    
- -l
    
    --list
    
    _List_: list files or arguments without taking other action.
    
- -o
    
    _Output_ filename
    
- -q
    
    --quiet
    
    _Quiet_: suppress stdout.
    
- -r
    
    -R
    
    --recursive
    
    _Recursive_: Operate recursively (down directory tree).
    
- -v
    
    --verbose
    
    _Verbose_: output additional information to stdout or stderr.
    
- -z
    
    --compress
    
    _Compress_: apply compression (usually [[../commands/file-and-archiving-commands#^GZIPREF|gzip]]).
    

However:

- In **tar** and **gawk**:
    
    -f
    
    --file
    
    _File_: filename follows.
    
- In **cp**, **mv**, **rm**:
    
    -f
    
    --force
    
    _Force_: force overwrite of target file(s).
    

> [!caution]
> Many UNIX and Linux utilities deviate from this "standard," so it is dangerous to _assume_ that a given option will behave in a standard way. Always check the man page for the command in question when in doubt.

A complete table of recommended options for the GNU utilities is available at [the GNU standards page](http://www.gnu.org/prep/standards/).
