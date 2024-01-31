---
title: 16.6. Communications Commands
---


Certain of the following commands find use in network data transfer and analysis, as well as in [[writing-scripts#^CSPAMMERS|chasing spammers]].

**Information and Statistics**

**host**

Searches for information about an Internet host by name or IP address, using DNS.

```bash
bash$ host surfacemail.com
surfacemail.com. has address 202.92.42.236
```

**ipcalc**

Displays IP information for a host. With the -h option, **ipcalc** does a reverse DNS lookup, finding the name of the host (server) from the IP address.

```bash
bash$ ipcalc -h 202.92.42.236
HOSTNAME=surfacemail.com
```

**nslookup**

Do an Internet "name server lookup" on a host by IP address. This is essentially equivalent to **ipcalc -h** or **dig -x** . The command may be run either interactively or noninteractively, i.e., from within a script.

The **nslookup** command has allegedly been "deprecated," but it is still useful.

```bash
bash$ nslookup -sil 66.97.104.180
nslookup kuhleersparnis.ch
 Server:         135.116.137.2
 Address:        135.116.137.2#53

 Non-authoritative answer:
 Name:   kuhleersparnis.ch
```

**dig**

**D**omain **I**nformation **G**roper. Similar to **nslookup**, _dig_ does an Internet _name server lookup_ on a host. May be run from the command-line or from within a script.

Some interesting options to _dig_ are +time=N for setting a query timeout to _N_ seconds, +nofail for continuing to query servers until a reply is received, and -x for doing a reverse address lookup.

Compare the output of **dig -x** with **ipcalc -h** and **nslookup**.

```bash
bash$ dig -x 81.9.6.2
;; Got answer:
 ;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 11649
 ;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0

 ;; QUESTION SECTION:
 ;2.6.9.81.in-addr.arpa.         IN      PTR

 ;; AUTHORITY SECTION:
 6.9.81.in-addr.arpa.    3600    IN      SOA     ns.eltel.net. noc.eltel.net.
 2002031705 900 600 86400 3600

 ;; Query time: 537 msec
 ;; SERVER: 135.116.137.2#53(135.116.137.2)
 ;; WHEN: Wed Jun 26 08:35:24 2002
 ;; MSG SIZE  rcvd: 91
```

![[Example 16-40|Example 16-40]]

![[Example 16-41|Example 16-41]]

For a much more elaborate version of the above script, see [[Example A-28|Example A-28]].

**traceroute**

Trace the route taken by packets sent to a remote host. This command works within a LAN, WAN, or over the Internet. The remote host may be specified by an IP address. The output of this command may be filtered by [[external-filters-programs-and-commands#^GREPREF|grep]] or [[Appendix%20C.%20A%20Sed%20and%20Awk%20Micro-Primer.md#^SEDREF|sed]] in a pipe.

```bash
bash$ traceroute 81.9.6.2
traceroute to 81.9.6.2 (81.9.6.2), 30 hops max, 38 byte packets
 1  tc43.xjbnnbrb.com (136.30.178.8)  191.303 ms  179.400 ms  179.767 ms
 2  or0.xjbnnbrb.com (136.30.178.1)  179.536 ms  179.534 ms  169.685 ms
 3  192.168.11.101 (192.168.11.101)  189.471 ms  189.556 ms *
 ...
```

**ping**

Broadcast an _ICMP ECHO_REQUEST_ packet to another machine, either on a local or remote network. This is a diagnostic tool for testing network connections, and it should be used with caution.

```bash
bash$ ping localhost
PING localhost.localdomain (127.0.0.1) from 127.0.0.1 : 56(84) bytes of data.
 64 bytes from localhost.localdomain (127.0.0.1): icmp_seq=0 ttl=255 time=709 usec
 64 bytes from localhost.localdomain (127.0.0.1): icmp_seq=1 ttl=255 time=286 usec

 --- localhost.localdomain ping statistics ---
 2 packets transmitted, 2 packets received, 0% packet loss
 round-trip min/avg/max/mdev = 0.286/0.497/0.709/0.212 ms
```

A successful _ping_ returns an [[exit-and-exit-status#^EXITSTATUSREF|exit status]] of 0. This can be tested for in a script.

```bash
HNAME=news-15.net  # Notorious spammer.
# HNAME=$HOST     # Debug: test for localhost.
  count=2  # Send only two pings.

if [[ `ping -c $count "$HNAME"` ]]
then
  echo ""$HNAME" still up and broadcasting spam your way."
else
  echo ""$HNAME" seems to be down. Pity."
fi
```

**whois**

Perform a DNS (Domain Name System) lookup. The -h option permits specifying which particular _whois_ server to query. See [[Example 4-6|Example 4-6]] and [[Example 16-40|Example 16-40]].

**finger**

Retrieve information about users on a network. Optionally, this command can display a user's ~/.plan, ~/.project, and ~/.forward files, if present.

```bash
bash$ finger
Login  Name           Tty      Idle  Login Time   Office     Office Phone
 bozo   Bozo Bozeman   tty1        8  Jun 25 16:59                (:0)
 bozo   Bozo Bozeman   ttyp0          Jun 25 16:59                (:0.0)
 bozo   Bozo Bozeman   ttyp1          Jun 25 17:07                (:0.0)

bash$ **finger bozo**
Login: bozo                             Name: Bozo Bozeman
 Directory: /home/bozo                   Shell: /bin/bash
 Office: 2355 Clown St., 543-1234
 On since Fri Aug 31 20:13 (MST) on tty1    1 hour 38 minutes idle
 On since Fri Aug 31 20:13 (MST) on pts/0   12 seconds idle
 On since Fri Aug 31 20:13 (MST) on pts/1
 On since Fri Aug 31 20:31 (MST) on pts/2   1 hour 16 minutes idle
 Mail last read Tue Jul  3 10:08 2007 (MST) 
 No Plan.
```

Out of security considerations, many networks disable **finger** and its associated daemon. [^1]

**chfn**

Change information disclosed by the **finger** command.

**vrfy**

Verify an Internet e-mail address.

This command seems to be missing from newer Linux distros.

**Remote Host Access**

**sx**, **rx**

The **sx** and **rx** command set serves to transfer files to and from a remote host using the _xmodem_ protocol. These are generally part of a communications package, such as **minicom**.

**sz**, **rz**

The **sz** and **rz** command set serves to transfer files to and from a remote host using the _zmodem_ protocol. _Zmodem_ has certain advantages over _xmodem_, such as faster transmission rate and resumption of interrupted file transfers. Like **sx** and **rx**, these are generally part of a communications package.

**ftp**

Utility and protocol for uploading / downloading files to or from a remote host. An ftp session can be automated in a script (see [[Example 19-6|Example 19-6]] and [[Example A-4|Example A-4]]).

**uucp**, **uux**, **cu**

**uucp**: _UNIX to UNIX copy_. This is a communications package for transferring files between UNIX servers. A shell script is an effective way to handle a **uucp** command sequence.

Since the advent of the Internet and e-mail, **uucp** seems to have faded into obscurity, but it still exists and remains perfectly workable in situations where an Internet connection is not available or appropriate. The advantage of **uucp** is that it is fault-tolerant, so even if there is a service interruption the copy operation will resume where it left off when the connection is restored.

> **uux**: _UNIX to UNIX execute_. Execute a command on a remote system. This command is part of the **uucp** package.

**cu**: **C**all **U**p a remote system and connect as a simple terminal. It is a sort of dumbed-down version of [[communications-commands#^TELNETREF|telnet]]. This command is part of the **uucp** package.

**telnet**

Utility and protocol for connecting to a remote host.

> [!caution]
> The _telnet_ protocol contains security holes and should therefore probably be avoided. Its use within a shell script is _not_ recommended.

**wget**

The **wget** utility _noninteractively_ retrieves or downloads files from a Web or ftp site. It works well in a script.

```bash
wget -p http://www.xyz23.com/file01.html
#  The -p or --page-requisite option causes wget to fetch all files
#+ required to display the specified page.

wget -r ftp://ftp.xyz24.net/~bozo/project_files/ -O $SAVEFILE
#  The -r option recursively follows and retrieves all links
#+ on the specified site.

wget -c ftp://ftp.xyz25.net/bozofiles/filename.tar.bz2
#  The -c option lets wget resume an interrupted download.
#  This works with ftp servers and many HTTP sites.
```

![[Example 16-42|Example 16-42]]

See also [[Example A-30|Example A-30]] and [[Example A-31|Example A-31]].

**lynx**

The **lynx** Web and file browser can be used inside a script (with the -dump option) to retrieve a file from a Web or ftp site noninteractively.

```bash
lynx -dump http://www.xyz23.com/file01.html >$SAVEFILE
```

With the -traversal option, **lynx** starts at the HTTP URL specified as an argument, then "crawls" through all links located on that particular server. Used together with the -crawl option, outputs page text to a log file.

**rlogin**

_Remote login_, initates a session on a remote host. This command has security issues, so use [[communications-commands#^SSHREF|ssh]] instead.

**rsh**

_Remote shell_, executes command(s) on a remote host. This has security issues, so use **ssh** instead.

**rcp**

_Remote copy_, copies files between two different networked machines.

**rsync**

_Remote synchronize_, updates (synchronizes) files between two different networked machines.

```bash
bash$ rsync -a ~/sourcedir/*txt /node1/subdirectory/
```

![[Example 16-43|Example 16-43]]

See also [[Example A-32|Example A-32]].

> [!note]
> Using [[communications-commands#^RCPREF|rcp]], [[communications-commands#^RSYNCREF|rsync]], and similar utilities with security implications in a shell script may not be advisable. Consider, instead, using **ssh**, [[communications-commands#^SCPREF|scp]], or an **expect** script.

**ssh**

_Secure shell_, logs onto a remote host and executes commands there. This secure replacement for **telnet**, **rlogin**, **rcp**, and **rsh** uses identity authentication and encryption. See its [[external-filters-programs-and-commands#^MANREF|manpage]] for details.

![[Example 16-44|Example 16-44]]

> [!caution]
> Within a loop, **ssh** may cause unexpected behavior. According to a [Usenet post](http://groups-beta.google.com/group/comp.unix.shell/msg/dcb446b5fff7d230) in the comp.unix shell archives, **ssh** inherits the loop's stdin. To remedy this, pass **ssh** either the -n or -f option.
>
> Thanks, Jason Bechtel, for pointing this out.

**scp**

_Secure copy_, similar in function to **rcp**, copies files between two different networked machines, but does so using authentication, and with a security level similar to **ssh**.

**Local Network**

**write**

This is a utility for terminal-to-terminal communication. It allows sending lines from your terminal (console or _xterm_) to that of another user. The [[system-and-administrative-commands#^MESGREF|mesg]] command may, of course, be used to disable write access to a terminal

Since **write** is interactive, it would not normally find use in a script.

**netconfig**

A command-line utility for configuring a network adapter (using _DHCP_). This command is native to Red Hat centric Linux distros.

**Mail**

**mail**

Send or read e-mail messages.

This stripped-down command-line mail client works fine as a command embedded in a script.

![[Example 16-45|Example 16-45]]

**mailto**

Similar to the **mail** command, **mailto** sends e-mail messages from the command-line or in a script. However, **mailto** also permits sending MIME (multimedia) messages.

**mailstats**

Show _mail statistics_. This command may be invoked only by _root_.

```bash
root# mailstats
Statistics from Tue Jan  1 20:32:08 2008
  M   msgsfr  bytes_from   msgsto    bytes_to  msgsrej msgsdis msgsqur  Mailer
  4     1682      24118K        0          0K        0       0       0  esmtp
  9      212        640K     1894      25131K        0       0       0  local
 =====================================================================
  T     1894      24758K     1894      25131K        0       0       0
  C      414                    0
```

**vacation**

This utility automatically replies to e-mails that the intended recipient is on vacation and temporarily unavailable. It runs on a network, in conjunction with **sendmail**, and is not applicable to a dial-up POPmail account.

[^1]: A _daemon_ is a background process not attached to a terminal session. Daemons perform designated services either at specified times or explicitly triggered by certain events.
    
    The word "daemon" means ghost in Greek, and there is certainly something mysterious, almost supernatural, about the way UNIX daemons wander about behind the scenes, silently carrying out their appointed tasks.

