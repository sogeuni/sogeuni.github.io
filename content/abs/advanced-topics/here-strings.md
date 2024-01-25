---
title: 19.1. Here Strings
---


> A _here string_ can be considered as a stripped-down form of a _here document_.  
> It consists of nothing more than **COMMAND <<< $WORD**,  
> where $WORD is expanded and fed to the stdin of **COMMAND**.

As a simple example, consider this alternative to the [[internal-commands-and-builtins#^ECHOGREPREF|echo-grep]] construction.

```bash
# Instead of:
if echo "$VAR" | grep -q txt   # if [[ $VAR = *txt* ]]
# etc.

# Try:
if grep -q "txt" <<< "$VAR"
then   #         ^^^
   echo "$VAR contains the substring sequence \"txt\""
fi
# Thank you, Sebastian Kaminski, for the suggestion.
```

Or, in combination with [[internal-commands-and-builtins#^READREF|read]]:

```bash
String="This is a string of words."

read -r -a Words <<< "$String"
#  The -a option to "read"
#+ assigns the resulting values to successive members of an array.

echo "First word in String is:    ${Words[0]}"   # This
echo "Second word in String is:   ${Words[1]}"   # is
echo "Third word in String is:    ${Words[2]}"   # a
echo "Fourth word in String is:   ${Words[3]}"   # string
echo "Fifth word in String is:    ${Words[4]}"   # of
echo "Sixth word in String is:    ${Words[5]}"   # words.
echo "Seventh word in String is:  ${Words[6]}"   # (null)
                                                 # Past end of $String.

# Thank you, Francisco Lobo, for the suggestion.
```

It is, of course, possible to feed the output of a _here string_ into the stdin of a [[loops-and-branches#^LOOPREF00|loop]].

```bash
# As Seamus points out . . .

ArrayVar=( element0 element1 element2 {A..D} )

while read element ; do
  echo "$element" 1>&2
done <<< $(echo ${ArrayVar[*]})

# element0 element1 element2 A B C D
```

###### Example 19-13. Prepending a line to a file

```bash
#!/bin/bash
# prepend.sh: Add text at beginning of file.
#
#  Example contributed by Kenny Stauffer,
#+ and slightly modified by document author.


E_NOSUCHFILE=85

read -p "File: " file   # -p arg to 'read' displays prompt.
if [ ! -e "$file" ]
then   # Bail out if no such file.
  echo "File $file not found."
  exit $E_NOSUCHFILE
fi

read -p "Title: " title
cat - $file <<<$title > $file.new

echo "Modified file is $file.new"

exit  # Ends script execution.

  from 'man bash':
  Here Strings
  	A variant of here documents, the format is:
  
  		<<<word
  
  	The word is expanded and supplied to the command on its standard input.


  Of course, the following also works:
   sed -e '1i\
   Title: ' $file
```

###### Example 19-14. Parsing a mailbox

```bash
#!/bin/bash
#  Script by Francisco Lobo,
#+ and slightly modified and commented by ABS Guide author.
#  Used in ABS Guide with permission. (Thank you!)

# This script will not run under Bash versions -lt 3.0.


E_MISSING_ARG=87
if [ -z "$1" ]
then
  echo "Usage: $0 mailbox-file"
  exit $E_MISSING_ARG
fi

mbox_grep()  # Parse mailbox file.
{
    declare -i body=0 match=0
    declare -a date sender
    declare mail header value


    while IFS= read -r mail
#         ^^^^                 Reset $IFS.
#  Otherwise "read" will strip leading & trailing space from its input.

   do
       if [[ $mail =~ ^From  ]]   # Match "From" field in message.
       then
          (( body  = 0 ))           # "Zero out" variables.
          (( match = 0 ))
          unset date

       elif (( body ))
       then
            (( match ))
            #  echo "$mail"
            #  Uncomment above line if you want entire body
            #+ of message to display.

   elif [[ $mail ]]; then
      IFS=: read -r header value <<< "$mail"
      #                          ^^^  "here string"

      case "$header" in
      [Ff][Rr][Oo][Mm] ) [[ $value =~ "$2" ]] && (( match++ )) ;;
      # Match "From" line.
      [Dd][Aa][Tt][Ee] ) read -r -a date <<< "$value" ;;
      #                                  ^^^
      # Match "Date" line.
      [Rr][Ee][Cc][Ee][Ii][Vv][Ee][Dd] ) read -r -a sender <<< "$value" ;;
      #                                                    ^^^
      # Match IP Address (may be spoofed).
      esac

       else
          (( body++ ))
          (( match  )) &&
          echo "MESSAGE ${date:+of: ${date[*]} }"
       #    Entire $date array             ^
          echo "IP address of sender: ${sender[1]}"
       #    Second field of "Received" line    ^

       fi


    done < "$1" # Redirect stdout of file into loop.
}


mbox_grep "$1"  # Send mailbox file to function.

exit $?

# Exercises:
# ---------
# 1) Break the single function, above, into multiple functions,
#+   for the sake of readability.
# 2) Add additional parsing to the script, checking for various keywords.



$ mailbox_grep.sh scam_mail
  MESSAGE of Thu, 5 Jan 2006 08:00:56 -0500 (EST) 
  IP address of sender: 196.3.62.4
```

Exercise: Find other uses for _here strings_, such as, for example, [[math-commands#^GOLDENRATIO|feeding input to _dc_]].
