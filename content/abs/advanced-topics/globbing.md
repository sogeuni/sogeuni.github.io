---
title: 18.2. Globbing
---


Bash itself cannot recognize Regular Expressions. Inside scripts, it is commands and utilities -- such as [[Appendix%20C.%20A%20Sed%20and%20Awk%20Micro-Primer#^SEDREF|sed]] and [[awk#^AWKREF|awk]] -- that interpret RE's.

Bash _does_ carry out _filename expansion_ [^1] -- a process known as _globbing_ -- but this does _not_ use the standard RE set. Instead, globbing recognizes and expands _wild cards_. Globbing interprets the standard wild card characters [^2] -- [[special-characters#^ASTERISKREF|*]] and [[special-characters#^WILDCARDQU|?]], character lists in square brackets, and certain other special characters (such as ^ for negating the sense of a match). There are important limitations on wild card characters in globbing, however. Strings containing _*_ will not match filenames that start with a dot, as, for example, [[sample-bashrc-and-bash-profile-files.html|.bashrc]]. [^3] Likewise, the _?_ has a different meaning in globbing than as part of an RE.

```bash
bash$ ls -l
total 2
 -rw-rw-r--    1 bozo  bozo         0 Aug  6 18:42 a.1
 -rw-rw-r--    1 bozo  bozo         0 Aug  6 18:42 b.1
 -rw-rw-r--    1 bozo  bozo         0 Aug  6 18:42 c.1
 -rw-rw-r--    1 bozo  bozo       466 Aug  6 17:48 t2.sh
 -rw-rw-r--    1 bozo  bozo       758 Jul 30 09:02 test1.txt

bash$ ls -l t?.sh
-rw-rw-r--    1 bozo  bozo       466 Aug  6 17:48 t2.sh

bash$ ls -l [ab]*
-rw-rw-r--    1 bozo  bozo         0 Aug  6 18:42 a.1
 -rw-rw-r--    1 bozo  bozo         0 Aug  6 18:42 b.1

bash$ ls -l [a-c]*
-rw-rw-r--    1 bozo  bozo         0 Aug  6 18:42 a.1
 -rw-rw-r--    1 bozo  bozo         0 Aug  6 18:42 b.1
 -rw-rw-r--    1 bozo  bozo         0 Aug  6 18:42 c.1

bash$ ls -l [^ab]*
-rw-rw-r--    1 bozo  bozo         0 Aug  6 18:42 c.1
 -rw-rw-r--    1 bozo  bozo       466 Aug  6 17:48 t2.sh
 -rw-rw-r--    1 bozo  bozo       758 Jul 30 09:02 test1.txt

bash$ ls -l {b*,c*,*est*}
-rw-rw-r--    1 bozo  bozo         0 Aug  6 18:42 b.1
 -rw-rw-r--    1 bozo  bozo         0 Aug  6 18:42 c.1
 -rw-rw-r--    1 bozo  bozo       758 Jul 30 09:02 test1.txt
	      
```

Bash performs filename expansion on unquoted command-line arguments. The [[internal-commands-and-builtins#^ECHOREF|echo]] command demonstrates this.

```bash
bash$ echo *
a.1 b.1 c.1 t2.sh test1.txt

bash$ echo t*
t2.sh test1.txt

bash$ echo t?.sh
t2.sh
	      
```

> [!note]
> It is possible to modify the way Bash interprets special characters in globbing. A **set -f** command disables globbing, and the nocaseglob and nullglob options to [[internal-commands-and-builtins#^SHOPTREF|shopt]] change globbing behavior.

See also [[Example 11-5|Example 11-5]].

> [!caution]
> Filenames with embedded [[special-characters#Whitespace|whitespace]] can cause _globbing_ to choke. [David Wheeler](http://www.dwheeler.com/essays/filenames-in-shell.html) shows how to avoid many such pitfalls.

```bash
IFS="$(printf '\n\t')"   # Remove space.

#  Correct glob use:
#  Always use for-loop, prefix glob, check if exists file.
for file in ./* ; do         # Use ./* ... NEVER bare *
  if [ -e "$file" ] ; then   # Check whether file exists.
     COMMAND ... "$file" ...
  fi
done

# This example taken from David Wheeler's site, with permission.
```

[^1]: _Filename expansion_ means expanding filename patterns or templates containing special characters. For example, example.??? might expand to example.001 and/or example.txt.

[^2]: A _wild card_ character, analogous to a wild card in poker, can represent (almost) any other character.

[^3]: Filename expansion _can_ match dotfiles, but only if the pattern explicitly includes the dot as a literal character.

    ```bash
    ~/[.]bashrc    #  Will not expand to ~/.bashrc
    ~/?bashrc      #  Neither will this.
                #  Wild cards and metacharacters will NOT
                #+ expand to a dot in globbing.

    ~/.[b]ashrc    #  Will expand to ~/.bashrc
    ~/.ba?hrc      #  Likewise.
    ~/.bashr*      #  Likewise.

    # Setting the "dotglob" option turns this off.

    # Thanks, S.C.
    ```
