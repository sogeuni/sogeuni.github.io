---
title: 16.6. Communications Commands
---


Certain of the following commands find use in network data transfer and analysis, as well as in [[../apendix/writing-scripts#^CSPAMMERS|chasing spammers]].

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

###### Example 16-40. Finding out where to report a spammer

```bash
#!/bin/bash
# spam-lookup.sh: Look up abuse contact to report a spammer.
# Thanks, Michael Zick.

# Check for command-line arg.
ARGCOUNT=1
E_WRONGARGS=85
if [ $# -ne "$ARGCOUNT" ]
then
  echo "Usage: `basename $0` domain-name"
  exit $E_WRONGARGS
fi


dig +short $1.contacts.abuse.net -c in -t txt
# Also try:
#     dig +nssearch $1
#     Tries to find "authoritative name servers" and display SOA records.

# The following also works:
#     whois -h whois.abuse.net $1
#           ^^ ^^^^^^^^^^^^^^^  Specify host.  
#     Can even lookup multiple spammers with this, i.e."
#     whois -h whois.abuse.net $spamdomain1 $spamdomain2 . . .


#  Exercise:
#  --------
#  Expand the functionality of this script
#+ so that it automatically e-mails a notification
#+ to the responsible ISP's contact address(es).
#  Hint: use the "mail" command.

exit $?

# spam-lookup.sh chinatietong.com
#                A known spam domain.

# "crnet_mgr@chinatietong.com"
# "crnet_tec@chinatietong.com"
# "postmaster@chinatietong.com"


#  For a more elaborate version of this script,
#+ see the SpamViz home page, http://www.spamviz.net/index.html.
```

###### Example 16-41. Analyzing a spam domain

```bash
#! /bin/bash
# is-spammer.sh: Identifying spam domains

# $Id: is-spammer, v 1.4 2004/09/01 19:37:52 mszick Exp $
# Above line is RCS ID info.
#
#  This is a simplified version of the "is_spammer.bash
#+ script in the Contributed Scripts appendix.

# is-spammer <domain.name>

# Uses an external program: 'dig'
# Tested with version: 9.2.4rc5

# Uses functions.
# Uses IFS to parse strings by assignment into arrays.
# And even does something useful: checks e-mail blacklists.

# Use the domain.name(s) from the text body:
# http://www.good_stuff.spammer.biz/just_ignore_everything_else
#                       ^^^^^^^^^^^
# Or the domain.name(s) from any e-mail address:
# Really_Good_Offer@spammer.biz
#
# as the only argument to this script.
#(PS: have your Inet connection running)
#
# So, to invoke this script in the above two instances:
#       is-spammer.sh spammer.biz


# Whitespace == :Space:Tab:Line Feed:Carriage Return:
WSP_IFS=$'\x20'$'\x09'$'\x0A'$'\x0D'

# No Whitespace == Line Feed:Carriage Return
No_WSP=$'\x0A'$'\x0D'

# Field separator for dotted decimal ip addresses
ADR_IFS=${No_WSP}'.'

# Get the dns text resource record.
# get_txt <error_code> <list_query>
get_txt() {

    # Parse $1 by assignment at the dots.
    local -a dns
    IFS=$ADR_IFS
    dns=( $1 )
    IFS=$WSP_IFS
    if [ "${dns[0]}" == '127' ]
    then
        # See if there is a reason.
        echo $(dig +short $2 -t txt)
    fi
}

# Get the dns address resource record.
# chk_adr <rev_dns> <list_server>
chk_adr() {
    local reply
    local server
    local reason

    server=${1}${2}
    reply=$( dig +short ${server} )

    # If reply might be an error code . . .
    if [ ${#reply} -gt 6 ]
    then
        reason=$(get_txt ${reply} ${server} )
        reason=${reason:-${reply}}
    fi
    echo ${reason:-' not blacklisted.'}
}

# Need to get the IP address from the name.
echo 'Get address of: '$1
ip_adr=$(dig +short $1)
dns_reply=${ip_adr:-' no answer '}
echo ' Found address: '${dns_reply}

# A valid reply is at least 4 digits plus 3 dots.
if [ ${#ip_adr} -gt 6 ]
then
    echo
    declare query

    # Parse by assignment at the dots.
    declare -a dns
    IFS=$ADR_IFS
    dns=( ${ip_adr} )
    IFS=$WSP_IFS

    # Reorder octets into dns query order.
    rev_dns="${dns[3]}"'.'"${dns[2]}"'.'"${dns[1]}"'.'"${dns[0]}"'.'

# See: http://www.spamhaus.org (Conservative, well maintained)
    echo -n 'spamhaus.org says: '
    echo $(chk_adr ${rev_dns} 'sbl-xbl.spamhaus.org')

# See: http://ordb.org (Open mail relays)
    echo -n '   ordb.org  says: '
    echo $(chk_adr ${rev_dns} 'relays.ordb.org')

# See: http://www.spamcop.net/ (You can report spammers here)
    echo -n ' spamcop.net says: '
    echo $(chk_adr ${rev_dns} 'bl.spamcop.net')

# # # other blacklist operations # # #

# See: http://cbl.abuseat.org.
    echo -n ' abuseat.org says: '
    echo $(chk_adr ${rev_dns} 'cbl.abuseat.org')

# See: http://dsbl.org/usage (Various mail relays)
    echo
    echo 'Distributed Server Listings'
    echo -n '       list.dsbl.org says: '
    echo $(chk_adr ${rev_dns} 'list.dsbl.org')

    echo -n '   multihop.dsbl.org says: '
    echo $(chk_adr ${rev_dns} 'multihop.dsbl.org')

    echo -n 'unconfirmed.dsbl.org says: '
    echo $(chk_adr ${rev_dns} 'unconfirmed.dsbl.org')

else
    echo
    echo 'Could not use that address.'
fi

exit 0

# Exercises:
# --------

# 1) Check arguments to script,
#    and exit with appropriate error message if necessary.

# 2) Check if on-line at invocation of script,
#    and exit with appropriate error message if necessary.

# 3) Substitute generic variables for "hard-coded" BHL domains.

# 4) Set a time-out for the script using the "+time=" option
     to the 'dig' command.
```

For a much more elaborate version of the above script, see [[../apendix/contributed-scripts#^ISSPAMMER2|Example A-28]].

**traceroute**

Trace the route taken by packets sent to a remote host. This command works within a LAN, WAN, or over the Internet. The remote host may be specified by an IP address. The output of this command may be filtered by [[./text-processing-commands#^GREPREF|grep]] or [[Appendix%20C.%20A%20Sed%20and%20Awk%20Micro-Primer.md#^SEDREF|sed]] in a pipe.

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

Perform a DNS (Domain Name System) lookup. The -h option permits specifying which particular _whois_ server to query. See [[othertypesv#^EX18|Example 4-6]] and [[communications-commands#^SPAMLOOKUP|Example 16-40]].

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

Utility and protocol for uploading / downloading files to or from a remote host. An ftp session can be automated in a script (see [[../advanced-topics/here-documents#^EX72|Example 19-6]] and [[../apendix/contributed-scripts#^ENCRYPTEDPW|Example A-4]]).

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

###### Example 16-42. Getting a stock quote

```bash
#!/bin/bash
# quote-fetch.sh: Download a stock quote.


E_NOPARAMS=86

if [ -z "$1" ]  # Must specify a stock (symbol) to fetch.
  then echo "Usage: `basename $0` stock-symbol"
  exit $E_NOPARAMS
fi

stock_symbol=$1

file_suffix=.html
# Fetches an HTML file, so name it appropriately.
URL='http://finance.yahoo.com/q?s='
# Yahoo finance board, with stock query suffix.

# -----------------------------------------------------------
wget -O ${stock_symbol}${file_suffix} "${URL}${stock_symbol}"
# -----------------------------------------------------------


# To look up stuff on http://search.yahoo.com:
# -----------------------------------------------------------
# URL="http://search.yahoo.com/search?fr=ush-news&p=${query}"
# wget -O "$savefilename" "${URL}"
# -----------------------------------------------------------
# Saves a list of relevant URLs.

exit $?

# Exercises:
# ---------
#
# 1) Add a test to ensure the user running the script is on-line.
#    (Hint: parse the output of 'ps -ax' for "ppp" or "connect."
#
# 2) Modify this script to fetch the local weather report,
#+   taking the user's zip code as an argument.
```

See also [[../apendix/contributed-scripts#^WGETTER2|Example A-30]] and [[../apendix/contributed-scripts#^BASHPODDER|Example A-31]].

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

###### Example 16-43. Updating FC4

```bash
#!/bin/bash
# fc4upd.sh

# Script author: Frank Wang.
# Slight stylistic modifications by ABS Guide author.
# Used in ABS Guide with permission.


#  Download Fedora Core 4 update from mirror site using rsync. 
#  Should also work for newer Fedora Cores -- 5, 6, . . .
#  Only download latest package if multiple versions exist,
#+ to save space.

URL=rsync://distro.ibiblio.org/fedora-linux-core/updates/
# URL=rsync://ftp.kddilabs.jp/fedora/core/updates/
# URL=rsync://rsync.planetmirror.com/fedora-linux-core/updates/

DEST=${1:-/var/www/html/fedora/updates/}
LOG=/tmp/repo-update-$(/bin/date +%Y-%m-%d).txt
PID_FILE=/var/run/${0##*/}.pid

E_RETURN=85        # Something unexpected happened.


# General rsync options
# -r: recursive download
# -t: reserve time
# -v: verbose

OPTS="-rtv --delete-excluded --delete-after --partial"

# rsync include pattern
# Leading slash causes absolute path name match.
INCLUDE=(
    "/4/i386/kde-i18n-Chinese*" 
#   ^                         ^
# Quoting is necessary to prevent globbing.
) 


# rsync exclude pattern
# Temporarily comment out unwanted pkgs using "#" . . .
EXCLUDE=(
    /1
    /2
    /3
    /testing
    /4/SRPMS
    /4/ppc
    /4/x86_64
    /4/i386/debug
   "/4/i386/kde-i18n-*"
   "/4/i386/openoffice.org-langpack-*"
   "/4/i386/*i586.rpm"
   "/4/i386/GFS-*"
   "/4/i386/cman-*"
   "/4/i386/dlm-*"
   "/4/i386/gnbd-*"
   "/4/i386/kernel-smp*"
#  "/4/i386/kernel-xen*" 
#  "/4/i386/xen-*" 
)


init () {
    # Let pipe command return possible rsync error, e.g., stalled network.
    set -o pipefail                  # Newly introduced in Bash, version 3.

    TMP=${TMPDIR:-/tmp}/${0##*/}.$$  # Store refined download list.
    trap "{
        rm -f $TMP 2>/dev/null
    }" EXIT                          # Clear temporary file on exit.
}


check_pid () {
# Check if process exists.
    if [ -s "$PID_FILE" ]; then
        echo "PID file exists. Checking ..."
        PID=$(/bin/egrep -o "^[[:digit:]]+" $PID_FILE)
        if /bin/ps --pid $PID &>/dev/null; then
            echo "Process $PID found. ${0##*/} seems to be running!"
           /usr/bin/logger -t ${0##*/} \
                 "Process $PID found. ${0##*/} seems to be running!"
            exit $E_RETURN
        fi
        echo "Process $PID not found. Start new process . . ."
    fi
}


#  Set overall file update range starting from root or $URL,
#+ according to above patterns.
set_range () {
    include=
    exclude=
    for p in "${INCLUDE[@]}"; do
        include="$include --include \"$p\""
    done

    for p in "${EXCLUDE[@]}"; do
        exclude="$exclude --exclude \"$p\""
    done
}


# Retrieve and refine rsync update list.
get_list () {
    echo $$ > $PID_FILE | {
        echo "Can't write to pid file $PID_FILE"
        exit $E_RETURN
    }

    echo -n "Retrieving and refining update list . . ."

    # Retrieve list -- 'eval' is needed to run rsync as a single command.
    # $3 and $4 is the date and time of file creation.
    # $5 is the full package name.
    previous=
    pre_file=
    pre_date=0
    eval /bin/nice /usr/bin/rsync \
        -r $include $exclude $URL | \
        egrep '^dr.x|^-r' | \
        awk '{print $3, $4, $5}' | \
        sort -k3 | \
        { while read line; do
            # Get seconds since epoch, to filter out obsolete pkgs.
            cur_date=$(date -d "$(echo $line | awk '{print $1, $2}')" +%s)
            #  echo $cur_date

            # Get file name.
            cur_file=$(echo $line | awk '{print $3}')
            #  echo $cur_file

            # Get rpm pkg name from file name, if possible.
            if [[ $cur_file == *rpm ]]; then
                pkg_name=$(echo $cur_file | sed -r -e \
                    's/(^([^_-]+[_-])+)[[:digit:]]+\..*[_-].*$/\1/')
            else
                pkg_name=
            fi
            # echo $pkg_name

            if [ -z "$pkg_name" ]; then   #  If not a rpm file,
                echo $cur_file >> $TMP    #+ then append to download list.
            elif [ "$pkg_name" != "$previous" ]; then   # A new pkg found.
                echo $pre_file >> $TMP                  # Output latest file.
                previous=$pkg_name                      # Save current.
                pre_date=$cur_date
                pre_file=$cur_file
            elif [ "$cur_date" -gt "$pre_date" ]; then
                                                #  If same pkg, but newer,
                pre_date=$cur_date              #+ then update latest pointer.
                pre_file=$cur_file
            fi
            done
            echo $pre_file >> $TMP              #  TMP contains ALL
                                                #+ of refined list now.
            # echo "subshell=$BASH_SUBSHELL"

    }       # Bracket required here to let final "echo $pre_file >> $TMP" 
            # Remained in the same subshell ( 1 ) with the entire loop.

    RET=$?  # Get return code of the pipe command.

    [ "$RET" -ne 0 ] && {
        echo "List retrieving failed with code $RET"
        exit $E_RETURN
    }

    echo "done"; echo
}

# Real rsync download part.
get_file () {

    echo "Downloading..."
    /bin/nice /usr/bin/rsync \
        $OPTS \
        --filter "merge,+/ $TMP" \
        --exclude '*'  \
        $URL $DEST     \
        | /usr/bin/tee $LOG

    RET=$?

   #  --filter merge,+/ is crucial for the intention. 
   #  + modifier means include and / means absolute path.
   #  Then sorted list in $TMP will contain ascending dir name and 
   #+ prevent the following --exclude '*' from "shortcutting the circuit." 

    echo "Done"

    rm -f $PID_FILE 2>/dev/null

    return $RET
}

# -------
# Main
init
check_pid
set_range
get_list
get_file
RET=$?
# -------

if [ "$RET" -eq 0 ]; then
    /usr/bin/logger -t ${0##*/} "Fedora update mirrored successfully."
else
    /usr/bin/logger -t ${0##*/} \
    "Fedora update mirrored with failure code: $RET"
fi

exit $RET
```

See also [[../apendix/contributed-scripts#^NIGHTLYBACKUP|Example A-32]].

> [!note]
> Using [[communications-commands#^RCPREF|rcp]], [[communications-commands#^RSYNCREF|rsync]], and similar utilities with security implications in a shell script may not be advisable. Consider, instead, using **ssh**, [[communications-commands#^SCPREF|scp]], or an **expect** script.

**ssh**

_Secure shell_, logs onto a remote host and executes commands there. This secure replacement for **telnet**, **rlogin**, **rcp**, and **rsh** uses identity authentication and encryption. See its [[./basic-commands#^MANREF|manpage]] for details.

###### Example 16-44. Using *ssh*

```bash
#!/bin/bash
# remote.bash: Using ssh.

# This example by Michael Zick.
# Used with permission.


#   Presumptions:
#   ------------
#   fd-2 isn't being captured ( '2>/dev/null' ).
#   ssh/sshd presumes stderr ('2') will display to user.
#
#   sshd is running on your machine.
#   For any 'standard' distribution, it probably is,
#+  and without any funky ssh-keygen having been done.

# Try ssh to your machine from the command-line:
#
# $ ssh $HOSTNAME
# Without extra set-up you'll be asked for your password.
#   enter password
#   when done,  $ exit
#
# Did that work? If so, you're ready for more fun.

# Try ssh to your machine as 'root':
#
#   $  ssh -l root $HOSTNAME
#   When asked for password, enter root's, not yours.
#          Last login: Tue Aug 10 20:25:49 2004 from localhost.localdomain
#   Enter 'exit' when done.

#  The above gives you an interactive shell.
#  It is possible for sshd to be set up in a 'single command' mode,
#+ but that is beyond the scope of this example.
#  The only thing to note is that the following will work in
#+ 'single command' mode.


# A basic, write stdout (local) command.

ls -l

# Now the same basic command on a remote machine.
# Pass a different 'USERNAME' 'HOSTNAME' if desired:
USER=${USERNAME:-$(whoami)}
HOST=${HOSTNAME:-$(hostname)}

#  Now excute the above command-line on the remote host,
#+ with all transmissions encrypted.

ssh -l ${USER} ${HOST} " ls -l "

#  The expected result is a listing of your username's home
#+ directory on the remote machine.
#  To see any difference, run this script from somewhere
#+ other than your home directory.

#  In other words, the Bash command is passed as a quoted line
#+ to the remote shell, which executes it on the remote machine.
#  In this case, sshd does  ' bash -c "ls -l" '   on your behalf.

#  For information on topics such as not having to enter a
#+ password/passphrase for every command-line, see
#+    man ssh
#+    man ssh-keygen
#+    man sshd_config.

exit 0
```

> [!caution]
> Within a loop, **ssh** may cause unexpected behavior. According to a [Usenet post](http://groups-beta.google.com/group/comp.unix.shell/msg/dcb446b5fff7d230) in the comp.unix shell archives, **ssh** inherits the loop's stdin. To remedy this, pass **ssh** either the -n or -f option.
>
> Thanks, Jason Bechtel, for pointing this out.

**scp**

_Secure copy_, similar in function to **rcp**, copies files between two different networked machines, but does so using authentication, and with a security level similar to **ssh**.

**Local Network**

**write**

This is a utility for terminal-to-terminal communication. It allows sending lines from your terminal (console or _xterm_) to that of another user. The [[./system-and-administrative-commands#^MESGREF|mesg]] command may, of course, be used to disable write access to a terminal

Since **write** is interactive, it would not normally find use in a script.

**netconfig**

A command-line utility for configuring a network adapter (using _DHCP_). This command is native to Red Hat centric Linux distros.

**Mail**

**mail**

Send or read e-mail messages.

This stripped-down command-line mail client works fine as a command embedded in a script.

###### Example 16-45. A script that mails itself

```bash
#!/bin/sh
# self-mailer.sh: Self-mailing script

adr=${1:-`whoami`}     # Default to current user, if not specified.
#  Typing 'self-mailer.sh wiseguy@superdupergenius.com'
#+ sends this script to that addressee.
#  Just 'self-mailer.sh' (no argument) sends the script
#+ to the person invoking it, for example, bozo@localhost.localdomain.
#
#  For more on the ${parameter:-default} construct,
#+ see the "Parameter Substitution" section
#+ of the "Variables Revisited" chapter.

# ============================================================================
  cat $0 | mail -s "Script \"`basename $0`\" has mailed itself to you." "$adr"
# ============================================================================

# --------------------------------------------
#  Greetings from the self-mailing script.
#  A mischievous person has run this script,
#+ which has caused it to mail itself to you.
#  Apparently, some people have nothing better
#+ to do with their time.
# --------------------------------------------

echo "At `date`, script \"`basename $0`\" mailed to "$adr"."

exit 0

#  Note that the "mailx" command (in "send" mode) may be substituted
#+ for "mail" ... but with somewhat different options.
```

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

