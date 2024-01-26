---
title: 16.3. Time / Date Commands
---

**Time/date and timing**

## date

Simply invoked, **date** prints the date and time to stdout. Where this command gets interesting is in its formatting and parsing options.

###### Example 16-10. Using *date*

```bash
#!/bin/bash
# Exercising the 'date' command

echo "The number of days since the year's beginning is `date +%j`."
# Needs a leading '+' to invoke formatting.
# %j gives day of year.

echo "The number of seconds elapsed since 01/01/1970 is `date +%s`."
#  %s yields number of seconds since "UNIX epoch" began,
#+ but how is this useful?

prefix=temp
suffix=$(date +%s)  # The "+%s" option to 'date' is GNU-specific.
filename=$prefix.$suffix
echo "Temporary filename = $filename"
#  It's great for creating "unique and random" temp filenames,
#+ even better than using $$.

# Read the 'date' man page for more formatting options.

exit 0
```

The -u option gives the UTC (Universal Coordinated Time).

```bash
bash$ date
Fri Mar 29 21:07:39 MST 2002



bash$ date -u
Sat Mar 30 04:07:42 UTC 2002
	      
```

This option facilitates calculating the time between different dates.

###### Example 16-11. *Date* calculations

```bash
#!/bin/bash
# date-calc.sh
# Author: Nathan Coulter
# Used in ABS Guide with permission (thanks!).

MPHR=60    # Minutes per hour.
HPD=24     # Hours per day.

diff () {
        printf '%s' $(( $(date -u -d"$TARGET" +%s) -
                        $(date -u -d"$CURRENT" +%s)))
#                       %d = day of month.
}


CURRENT=$(date -u -d '2007-09-01 17:30:24' '+%F %T.%N %Z')
TARGET=$(date -u -d'2007-12-25 12:30:00' '+%F %T.%N %Z')
# %F = full date, %T = %H:%M:%S, %N = nanoseconds, %Z = time zone.

printf '\nIn 2007, %s ' \
       "$(date -d"$CURRENT +
        $(( $(diff) /$MPHR /$MPHR /$HPD / 2 )) days" '+%d %B')" 
#       %B = name of month                ^ halfway
printf 'was halfway between %s ' "$(date -d"$CURRENT" '+%d %B')"
printf 'and %s\n' "$(date -d"$TARGET" '+%d %B')"

printf '\nOn %s at %s, there were\n' \
        $(date -u -d"$CURRENT" +%F) $(date -u -d"$CURRENT" +%T)
DAYS=$(( $(diff) / $MPHR / $MPHR / $HPD ))
CURRENT=$(date -d"$CURRENT +$DAYS days" '+%F %T.%N %Z')
HOURS=$(( $(diff) / $MPHR / $MPHR ))
CURRENT=$(date -d"$CURRENT +$HOURS hours" '+%F %T.%N %Z')
MINUTES=$(( $(diff) / $MPHR ))
CURRENT=$(date -d"$CURRENT +$MINUTES minutes" '+%F %T.%N %Z')
printf '%s days, %s hours, ' "$DAYS" "$HOURS"
printf '%s minutes, and %s seconds ' "$MINUTES" "$(diff)"
printf 'until Christmas Dinner!\n\n'

#  Exercise:
#  --------
#  Rewrite the diff () function to accept passed parameters,
#+ rather than using global variables.
```

The _date_ command has quite a number of _output_ options. For example %N gives the nanosecond portion of the current time. One interesting use for this is to generate random integers.

```bash
date +%N | sed -e 's/000$//' -e 's/^0//'
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#  Strip off leading and trailing zeroes, if present.
#  Length of generated integer depends on
#+ how many zeroes stripped off.

# 115281032
# 63408725
# 394504284
```

There are many more options (try **man date**).

```bash
date +%j
# Echoes day of the year (days elapsed since January 1).

date +%k%M
# Echoes hour and minute in 24-hour format, as a single digit string.



# The 'TZ' parameter permits overriding the default time zone.
date                 # Mon Mar 28 21:42:16 MST 2005
TZ=EST date          # Mon Mar 28 23:42:16 EST 2005
# Thanks, Frank Kannemann and Pete Sjoberg, for the tip.


SixDaysAgo=$(date --date='6 days ago')
OneMonthAgo=$(date --date='1 month ago')  # Four weeks back (not a month!)
OneYearAgo=$(date --date='1 year ago')
```

See also [[Example 3-4|Example 3-4]] and [[Example A-43|Example A-43]].

## zdump

Time zone dump: echoes the time in a specified time zone.

```bash
bash$ zdump EST
EST  Tue Sep 18 22:09:22 2001 EST
	    
```

## time

Outputs verbose timing statistics for executing a command.

**time ls -l /** gives something like this:

```bash
real    0m0.067s
 user    0m0.004s
 sys     0m0.005s
```

See also the very similar [[x9644#^TIMESREF|times]] command in the previous section.

> [!note]
> As of [[bashver2^#BASH2REF|version 2.0]] of Bash, **time** became a shell reserved word, with slightly altered behavior in a pipeline.

## touch

Utility for updating access/modification times of a file to current system time or other specified time, but also useful for creating a new file. The command **touch zzz** will create a new file of zero length, named zzz, assuming that zzz did not previously exist. Time-stamping empty files in this way is useful for storing date information, for example in keeping track of modification times on a project.

> [!note]
> The **touch** command is equivalent to **: >> newfile** or **>> newfile** (for ordinary files).

> [!tip]
> Before doing a [[basic#^CPREF|cp -u]] (_copy/update_), use **touch** to update the time stamp of files you don't wish overwritten.
>
> As an example, if the directory /home/bozo/tax_audit contains the files spreadsheet-051606.data, spreadsheet-051706.data, and spreadsheet-051806.data, then doing a **touch spreadsheet*.data** will protect these files from being overwritten by files with the same names during a **cp -u /home/bozo/financial_info/spreadsheet*data /home/bozo/tax_audit**.

## at

The **at** job control command executes a given set of commands at a specified time. Superficially, it resembles [[system-and-administrative-commands#^CRONREF|cron]], however, **at** is chiefly useful for one-time execution of a command set.

**at 2pm January 15** prompts for a set of commands to execute at that time. These commands should be shell-script compatible, since, for all practical purposes, the user is typing in an executable shell script a line at a time. Input terminates with a [[special-chars#^CTLDREF|Ctl-D]].

Using either the -f option or input redirection (<), **at** reads a command list from a file. This file is an executable shell script, though it should, of course, be non-interactive. Particularly clever is including the [[miscellaneous-commands#^RUNPARTSREF|run-parts]] command in the file to execute a different set of scripts.

```bash
bash$ at 2:30 am Friday < at-jobs.list
job 2 at 2000-10-27 02:30
	     
```

## batch

The **batch** job control command is similar to **at**, but it runs a command list when the system load drops below .8. Like **at**, it can read commands from a file with the -f option.

> The concept of _batch processing_ dates back to the era of mainframe computers. It means running a set of commands without user intervention.

## cal

Prints a neatly formatted monthly calendar to stdout. Will do current year or a large range of past and future years.

## sleep

This is the shell equivalent of a _wait loop_. It pauses for a specified number of seconds, doing nothing. It can be useful for timing or in processes running in the background, checking for a specific event every so often (polling), as in [[Example 32-6|Example 32-6]].

```bash
sleep 3     # Pauses 3 seconds.
```

> [!note]
> The **sleep** command defaults to seconds, but minute, hours, or days may also be specified.
>
> ```bash
> sleep 3 h   # Pauses 3 hours!
> ```

> [!note]
> The [[system-and-administrative-commands#^WATCHREF|watch]] command may be a better choice than **sleep** for running commands at timed intervals.

## usleep

_Microsleep_ (the _u_ may be read as the Greek _mu_, or _micro-_ prefix). This is the same as **sleep**, above, but "sleeps" in microsecond intervals. It can be used for fine-grained timing, or for polling an ongoing process at very frequent intervals.

```bash
usleep 30     # Pauses 30 microseconds.
```

This command is part of the Red Hat _initscripts / rc-scripts_ package.

> [!caution]
> The **usleep** command does not provide particularly accurate timing, and is therefore unsuitable for critical timing loops.

## hwclock, clock

The **hwclock** command accesses or adjusts the machine's hardware clock. Some options require _root_ privileges. The /etc/rc.d/rc.sysinit startup file uses **hwclock** to set the system time from the hardware clock at bootup.

The **clock** command is a synonym for **hwclock**.
