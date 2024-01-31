---
title: O.2. Writing Scripts
---


Write a script to carry out each of the following tasks.

**EASY**

**Self-reproducing Script**

Write a script that backs itself up, that is, copies itself to a file named backup.sh.

Hint: Use the [[external-filters-programs-and-commands#^CATREF|cat]] command and the appropriate [[othertypesv#^SCRNAMEPARAM|positional parameter]].

**Home Directory Listing**

Perform a recursive directory listing on the user's home directory and save the information to a file. Compress the file, have the script prompt the user to insert a USB flash drive, then press **ENTER**. Finally, save the file to the flash drive after making certain the flash drive has properly mounted by parsing the output of [[system-and-administrative-commands#^DFREF|df]]. Note that the flash drive must be _unmounted_ before it is removed.

**Converting [[loops-and-branches#^FORLOOPREF1|for]] loops to [[loops-and-branches#^WHILELOOPREF|while]] and [[loops-and-branches#^UNTILLOOPREF|until]] loops**

Convert the _for loops_ in [[Example 11-1|Example 11-1]] to _while loops_. Hint: store the data in an [[arrays#^ARRAYREF|array]] and step through the array elements.

Having already done the "heavy lifting," now convert the loops in the example to _until loops_.

**Changing the line spacing of a text file**

Write a script that reads each line of a target file, then writes the line back to stdout, but with an extra blank line following. This has the effect of _double-spacing_ the file.

Include all necessary code to check whether the script gets the necessary command-line argument (a filename), and whether the specified file exists.

When the script runs correctly, modify it to _triple-space_ the target file.

Finally, write a script to remove all blank lines from the target file, _single-spacing_ it.

**Backwards Listing**

Write a script that echoes itself to stdout, but _backwards_.

**Automatically Decompressing Files**

Given a list of filenames as input, this script queries each target file (parsing the output of the [[file-and-archiving-commands#^FILEREF|file]] command) for the type of compression used on it. Then the script automatically invokes the appropriate decompression command (**gunzip**, **bunzip2**, **unzip**, **uncompress**, or whatever). If a target file is not compressed, the script emits a warning message, but takes no other action on that particular file.

**Unique System ID**

Generate a "unique" 6-digit hexadecimal identifier for your computer. Do _not_ use the flawed [[system-and-administrative-commands#^HOSTIDREF|hostid]] command. Hint: **[[file-and-archiving-commands#^MD5SUMREF|md5sum]] [[important-files#^DATAFILESREF1|/etc/passwd]]**, then select the first 6 digits of output.

**Backup**

Archive as a "tarball" (*.tar.gz file) all the files in your home directory tree (/home/your-name) that have been modified in the last 24 hours. Hint: use [[external-filters-programs-and-commands#^FINDREF|find]].

Optional: you may use this as the basis of a _backup_ script.

**Checking whether a process is still running**

Given a [[special-characters#^PROCESSIDREF|process ID]] (_PID_) as an argument, this script will check, at user-specified intervals, whether the given process is still running. You may use the [[system-and-administrative-commands#^PPSSREF|ps]] and [[external-filters-programs-and-commands#^SLEEPREF|sleep]] commands.

**Primes**

Print (to stdout) all prime numbers between 60000 and 63000. The output should be nicely formatted in columns (hint: use [[internal-commands-and-builtins#^PRINTFREF|printf]]).

**Lottery Numbers**

One type of lottery involves picking five different numbers, in the range of 1 - 50. Write a script that generates five pseudorandom numbers in this range, _with no duplicates_. The script will give the option of echoing the numbers to stdout or saving them to a file, along with the date and time the particular number set was generated. (If your script consistently generates _winning_ lottery numbers, then you can retire on the proceeds and leave shell scripting to those of us who have to work for a living.)

**INTERMEDIATE**

**Integer or String**

Write a script [[functions|function]] that determines if an argument passed to it is an integer or a string. The function will return TRUE (0) if passed an integer, and FALSE (1) if passed a string.

Hint: What does the following expression return when $1 is _not_ an integer?

expr $1 + 0

**[[special-characters#^ASCIIDEF|ASCII]] to Integer**

The _atoi_ function in **C** converts a string character to an integer. Write a shell script function that performs the same operation. Likewise, write a shell script function that does the inverse, mirroring the **C** _itoa_ function which converts an integer into an ASCII character.

**Managing Disk Space**

List, one at a time, all files larger than 100K in the /home/username directory tree. Give the user the option to delete or compress the file, then proceed to show the next one. Write to a logfile the names of all deleted files and the deletion times.

**Banner**

Simulate the functionality of the deprecated [[miscellaneous-commands#^BANNERREF|banner]] command in a script.

**Removing Inactive Accounts**

Inactive accounts on a network server waste disk space and may become a security risk. Write an administrative script (to be invoked by _root_ or the [[system-and-administrative-commands#^CRONREF|cron daemon]]) that checks for and deletes user accounts that have not been accessed within the last 90 days.

**Enforcing Disk Quotas**

Write a script for a multi-user system that checks users' disk usage. If a user surpasses a preset limit (500 MB, for example) in her /home/username directory, then the script automatically sends her a "pigout" warning e-mail.

The script will use the [[system-and-administrative-commands#^DUREF|du]] and [[communications-commands#^COMMMAIL1|mail]] commands. As an option, it will allow setting and enforcing quotas using the [[system-and-administrative-commands#^QUOTAREF|quota]] and [[system-and-administrative-commands#^SETQUOTAREF|setquota]] commands.

**Logged in User Information**

For all logged in users, show their real names and the time and date of their last login.

Hint: use [[system-and-administrative-commands#^WHOREF|who]], [[system-and-administrative-commands#^LASTLOGREF|lastlog]], and parse [[important-files#^DATAFILESREF1|/etc/passwd]].

**Safe Delete**

Implement, as a script, a "safe" delete command, sdel.sh. Filenames passed as command-line arguments to this script are not deleted, but instead [[file-and-archiving-commands#^GZIPREF|gzipped]] if not already compressed (use [[file-and-archiving-commands#^FILEREF|file]] to check), then moved to a ~/TRASH directory. Upon invocation, the script checks the ~/TRASH directory for files older than 48 hours and [[external-filters-programs-and-commands#^RMREF|permanently deletes]] them. (An better alternative might be to have a second script handle this, periodically invoked by the [[system-and-administrative-commands#^CRONREF|cron daemon]].)

_Extra credit:_ Write the script so it can handle files and directories [[external-filters-programs-and-commands#^RMRECURS|recursively]]. This would give it the capability of "safely deleting" entire directory structures.

**Making Change**

What is the most efficient way to make change for $1.68, using only coins in common circulations (up to 25c)? It's 6 quarters, 1 dime, a nickel, and three cents.

Given any arbitrary command-line input in dollars and cents ($*.??), calculate the change, using the minimum number of coins. If your home country is not the United States, you may use your local currency units instead. The script will need to parse the command-line input, then change it to multiples of the smallest monetary unit (cents or whatever). Hint: look at [[Example 24-8|Example 24-8]].

**Quadratic Equations**

Solve a _quadratic_ equation of the form _Ax^2 + Bx + C = 0_. Have a script take as arguments the coefficients, **A**, **B**, and **C**, and return the solutions to five decimal places.

Hint: pipe the coefficients to [[math-commands#^BCREF|bc]], using the well-known formula, _x = ( -B +/- sqrt( B^2 - 4AC ) ) / 2A_.

**Table of Logarithms**

Using the [[math-commands#^BCREF|bc]] and [[internal-commands-and-builtins#^PRINTFREF|printf]] commands, print out a nicely-formatted table of eight-place natural logarithms in the interval between 0.00 and 100.00, in steps of .01.

Hint: _bc_ requires the -l option to load the math library.

**Unicode Table**

Using [[Example T-1|Example T-1]] as a template, write a script that prints to a file a complete [[bash-version-4#^UNICODEREF|Unicode]] table.

Hint: Use the -e option to [[internal-commands-and-builtins#^ECHOREF|echo]]: **echo -e '\uXXXX'**, where _XXXX_ is the Unicode numerical character designation. This requires [[bash-version-4#^BASH42|version 4.2]] or later of Bash.

**Sum of Matching Numbers**

Find the sum of all five-digit numbers (in the range 10000 - 99999) containing _exactly two_ out of the following set of digits: { 4, 5, 6 }. These may repeat within the same number, and if so, they count once for each occurrence.

Some examples of _matching numbers_ are 42057, 74638, and 89515.

**Lucky Numbers**

A _lucky number_ is one whose individual digits add up to 7, in successive additions. For example, 62431 is a _lucky number_ (6 + 2 + 4 + 3 + 1 = 16, 1 + 6 = 7). Find all the _lucky numbers_ between 1000 and 10000.

**Craps**

Borrowing the ASCII graphics from [[Example A-40|Example A-40]], write a script that plays the well-known gambling game of _craps_. The script will accept bets from one or more players, roll the dice, and keep track of wins and losses, as well as of each player's bankroll.

**Tic-tac-toe**

Write a script that plays the child's game of _tic-tac-toe_ against a human player. The script will let the human choose whether to take the first move. The script will follow an optimal strategy, and therefore never lose. To simplify matters, you may use ASCII graphics:

```bash
   o | x |
   ----------
     | x |
   ----------
     | o |
     
   Your move, human (row, column)?
```

**Alphabetizing a String**

Alphabetize (in ASCII order) an arbitrary string read from the command-line.

**Parsing**

Parse [[important-files#^DATAFILESREF1|/etc/passwd]], and output its contents in nice, easy-to-read tabular form.

**Logging Logins**

Parse /var/log/messages to produce a nicely formatted file of user logins and login times. The script may need to run as _root_. (Hint: Search for the string "LOGIN.")

**Pretty-Printing a Data File**

Certain database and spreadsheet packages use save-files with the fields separated by commas, commonly referred to as _comma-separated values_ or CSVs. Other applications often need to parse these files.

Given a data file with comma-separated [[special-characters#^FIELDREF|fields]], of the form:

|   |
|---|
|Jones,Bill,235 S. Williams St.,Denver,CO,80221,(303) 244-7989
Smith,Tom,404 Polk Ave.,Los Angeles,CA,90003,(213) 879-5612
...|

Reformat the data and print it out to stdout in labeled, evenly-spaced columns.

**Justification**

Given ASCII text input either from stdin or a file, adjust the word spacing to right-justify each line to a user-specified line-width, then send the output to stdout.

**Mailing List**

Using the [[communications-commands#^COMMMAIL1|mail]] command, write a script that manages a simple mailing list. The script automatically e-mails the monthly company newsletter, read from a specified text file, and sends it to all the addresses on the mailing list, which the script reads from another specified file.

**Generating Passwords**

Generate pseudorandom 8-character passwords, using characters in the ranges [0-9], [A-Z], [a-z]. Each password must contain at least two digits.

**Monitoring a User**

You suspect that one particular user on the network has been abusing her privileges and possibly attempting to hack the system. Write a script to automatically monitor and log her activities when she's signed on. The log file will save entries for the previous week, and delete those entries more than seven days old.

You may use [[system-and-administrative-commands#^LASTREF|last]], [[system-and-administrative-commands#^LASTLOGREF|lastlog]], and [[system-and-administrative-commands#^LASTCOMMREF|lastcomm]] to aid your surveillance of the suspected fiend.

**Checking for Broken Links**

Using [[communications-commands#^LYNXREF|lynx]] with the -traversal option, write a script that checks a Web site for broken links.

**DIFFICULT**

**Testing Passwords**

Write a script to check and validate passwords. The object is to flag "weak" or easily guessed password candidates.

A trial password will be input to the script as a command-line parameter. To be considered acceptable, a password must meet the following minimum qualifications:

- Minimum length of 8 characters
    
- Must contain at least one numeric character
    
- Must contain at least one of the following non-alphabetic characters: @, #, $, %, &, *, +, -, =
    

Optional:

- Do a dictionary check on every sequence of at least four consecutive alphabetic characters in the password under test. This will eliminate passwords containing embedded "words" found in a standard dictionary.
    
- Enable the script to check all the passwords on your system. These do not reside in [[important-files#^DATAFILESREF1|/etc/passwd]].
    

This exercise tests mastery of [[regexp#^REGEXREF|Regular Expressions]].

**Cross Reference**

Write a script that generates a _cross-reference_ (_concordance_) on a target file. The output will be a listing of all word occurrences in the target file, along with the line numbers in which each word occurs. Traditionally, _linked list_ constructs would be used in such applications. Therefore, you should investigate [[arrays#^ARRAYREF|arrays]] in the course of this exercise. [[Example 16-12|Example 16-12]] is probably _not_ a good place to start.

**Square Root**

Write a script to calculate square roots of numbers using _Newton's Method_.

The algorithm for this, expressed as a snippet of Bash [[assorted-tips#^PSEUDOCODEREF|pseudo-code]] is:

|   |
|---|
|#  (Isaac) Newton's Method for speedy extraction
#+ of square roots.

guess = $argument
#  $argument is the number to find the square root of.
#  $guess is each successive calculated "guess" -- or trial solution --
#+ of the square root.
#  Our first "guess" at a square root is the argument itself.

oldguess = 0
# $oldguess is the previous $guess.

tolerance = .000001
# To how close a tolerance we wish to calculate.

loopcnt = 0
# Let's keep track of how many times through the loop.
# Some arguments will require more loop iterations than others.


while [ ABS( $guess $oldguess ) -gt $tolerance ]
#       ^^^^^^^^^^^^^^^^^^^^^^^ Fix up syntax, of course.

#      "ABS" is a (floating point) function to find the absolute value
#+      of the difference between the two terms.
#             So, as long as difference between current and previous
#+            trial solution (guess) exceeds the tolerance, keep looping.

do
   oldguess = $guess  # Update $oldguess to previous $guess.

#  =======================================================
   guess = ( $oldguess + ( $argument / $oldguess ) ) / 2.0
#        = 1/2 ( ($oldguess **2 + $argument) / $oldguess )
#  equivalent to:
#        = 1/2 ( $oldguess + $argument / $oldguess )
#  that is, "averaging out" the trial solution and
#+ the proportion of argument deviation
#+ (in effect, splitting the error in half).
#  This converges on an accurate solution
#+ with surprisingly few loop iterations . . .
#+ for arguments > $tolerance, of course.
#  =======================================================

   (( loopcnt++ ))     # Update loop counter.
done|

It's a simple enough recipe, and _seems_ at first glance easy enough to convert into a working Bash script. The problem, though, is that Bash has [[operations-and-related-topics#^NOFLOATINGPOINT|no native support for floating point numbers]]. So, the script writer needs to use [[math-commands#^BCREF|bc]] or possibly [[awk#^AWKREF|awk]] to convert the numbers and do the calculations. It could get rather messy . . .

**Logging File Accesses**

Log all accesses to the files in /etc during the course of a single day. This information should include the filename, user name, and access time. If any alterations to the files take place, that will be flagged. Write this data as tabular (tab-separated) formatted records in a logfile.

**Monitoring Processes**

Write a script to continually monitor all running processes and to keep track of how many child processes each parent spawns. If a process spawns more than five children, then the script sends an e-mail to the system administrator (or _root_) with all relevant information, including the time, PID of the parent, PIDs of the children, etc. The script appends a report to a log file every ten minutes.

**Strip Comments**

Strip all comments from a shell script whose name is specified on the command-line. Note that the initial [[sha-bang#^SHABANGREF|#! line]] must not be stripped out.

**Strip HTML Tags**

Strip all the HTML tags from a specified HTML file, then reformat it into lines between 60 and 75 characters in length. Reset paragraph and block spacing, as appropriate, and convert HTML tables to their approximate text equivalent.

**XML Conversion**

Convert an XML file to both HTML and text format.

Optional: A script that converts Docbook/SGML to XML.

**Chasing Spammers**

Write a script that analyzes a spam e-mail by doing DNS lookups on the IP addresses in the headers to identify the relay hosts as well as the originating ISP. The script will forward the unaltered spam message to the responsible ISPs. Of course, it will be necessary to filter out _your own ISP's IP address_, so you don't end up complaining about yourself.

As necessary, use the appropriate [[communications-commands#^COMMUNINFO1|network analysis commands]].

For some ideas, see [[Example 16-41|Example 16-41]] and [[Example A-28|Example A-28]].

Optional: Write a script that searches through a list of e-mail messages and deletes the spam according to specified filters.

**Creating man pages**

Write a script that automates the process of creating [[external-filters-programs-and-commands#^MANREF|man pages]].

Given a text file which contains information to be formatted into a _man page_, the script will read the file, then invoke the appropriate [[external-filters-programs-and-commands#^GROFFREF|groff]] commands to output the corresponding _man page_ to stdout. The text file contains blocks of information under the standard _man page_ headings, i.e., NAME, SYNOPSIS, DESCRIPTION, etc.

[[Example A-39|Example A-39]] is an instructive first step.

**Hex Dump**

Do a hex(adecimal) dump on a binary file specified as an argument to the script. The output should be in neat tabular [[special-characters#^FIELDREF|fields]], with the first field showing the address, each of the next 8 fields a 4-byte hex number, and the final field the ASCII equivalent of the previous 8 fields.

The obvious followup to this is to extend the hex dump script into a disassembler. Using a lookup table, or some other clever gimmick, convert the hex values into 80x86 op codes.

**Emulating a Shift Register**

Using [[Example 27-15|Example 27-15]] as an inspiration, write a script that emulates a 64-bit shift register as an [[arrays#^ARRAYREF|array]]. Implement functions to _load_ the register, _shift left_, _shift right_, and _rotate_ it. Finally, write a function that interprets the register contents as eight 8-bit ASCII characters.

**Calculating Determinants**

Write a script that calculates determinants [[local-variables#^RECURSIONREF0|^1] by [recursively]] expanding the _minors_. Use a 4 x 4 determinant as a test case.

**Hidden Words**

Write a "word-find" puzzle generator, a script that hides 10 input words in a 10 x 10 array of random letters. The words may be hidden across, down, or diagonally.

Optional: Write a script that _solves_ word-find puzzles. To keep this from becoming too difficult, the solution script will find only horizontal and vertical words. (Hint: Treat each row and column as a string, and search for substrings.)

**Anagramming**

Anagram 4-letter input. For example, the anagrams of _word_ are: _do or rod row word_. You may use /usr/share/dict/linux.words as the reference list.

**Word Ladders**

A "word ladder" is a sequence of words, with each successive word in the sequence differing from the previous one by a single letter.

For example, to "ladder" from _mark_ to _vase_:

|   |
|---|
|mark --> park --> part --> past --> vast --> vase
         ^           ^       ^      ^           ^|

Write a script that solves word ladder puzzles. Given a starting and an ending word, the script will list all intermediate steps in the "ladder." Note that _all_ words in the sequence must be legitimate dictionary words.

**Fog Index**

The "fog index" of a passage of text estimates its reading difficulty, as a number corresponding roughly to a school grade level. For example, a passage with a fog index of 12 should be comprehensible to anyone with 12 years of schooling.

The Gunning version of the fog index uses the following algorithm.

1. Choose a section of the text at least 100 words in length.
    
2. Count the number of sentences (a portion of a sentence truncated by the boundary of the text section counts as one).
    
3. Find the average number of words per sentence.
    
    AVE_WDS_SEN = TOTAL_WORDS / SENTENCES
    
4. Count the number of "difficult" words in the segment -- those containing at least 3 syllables. Divide this quantity by total words to get the proportion of difficult words.
    
    PRO_DIFF_WORDS = LONG_WORDS / TOTAL_WORDS
    
5. The Gunning fog index is the sum of the above two quantities, multiplied by 0.4, then rounded to the nearest integer.
    
    G_FOG_INDEX = int ( 0.4 * ( AVE_WDS_SEN + PRO_DIFF_WORDS ) )
    

Step 4 is by far the most difficult portion of the exercise. There exist various algorithms for estimating the syllable count of a word. A rule-of-thumb formula might consider the number of letters in a word and the vowel-consonant mix.

A strict interpretation of the Gunning fog index does not count compound words and proper nouns as "difficult" words, but this would enormously complicate the script.

**Calculating PI using Buffon's Needle**

The Eighteenth Century French mathematician de Buffon came up with a novel experiment. Repeatedly drop a needle of length _n_ onto a wooden floor composed of long and narrow parallel boards. The cracks separating the equal-width floorboards are a fixed distance _d_ apart. Keep track of the total drops and the number of times the needle intersects a crack on the floor. The ratio of these two quantities turns out to be a fractional multiple of PI.

In the spirit of [[Example 16-50|Example 16-50]], write a script that runs a Monte Carlo simulation of _Buffon's Needle_. To simplify matters, set the needle length equal to the distance between the cracks, _n = d_.

Hint: there are actually two critical variables: the distance from the center of the needle to the nearest crack, and the inclination angle of the needle to that crack. You may use [[math-commands#^BCREF|bc]] to handle the calculations.

**Playfair Cipher**

Implement the Playfair (Wheatstone) Cipher in a script.

The Playfair Cipher encrypts text by substitution of _digrams_ (2-letter groupings). It is traditional to use a 5 x 5 letter scrambled-alphabet _key square_ for the encryption and decryption.

|   |
|---|
|C O D E S
   A B F G H
   I K L M N
   P Q R T U
   V W X Y Z

Each letter of the alphabet appears once, except "I" also represents
"J". The arbitrarily chosen key word, "CODES" comes first, then all
the rest of the alphabet, in order from left to right, skipping letters
already used.

To encrypt, separate the plaintext message into digrams (2-letter
groups). If a group has two identical letters, delete the second, and
form a new group. If there is a single letter left over at the end,
insert a "null" character, typically an "X."

THIS IS A TOP SECRET MESSAGE

TH IS IS AT OP SE CR ET ME SA GE



For each digram, there are three possibilities.
-----------------------------------------------

1) Both letters will be on the same row of the key square:
   For each letter, substitute the one immediately to the right, in that
   row. If necessary, wrap around left to the beginning of the row.

or

2) Both letters will be in the same column of the key square:
   For each letter, substitute the one immediately below it, in that
   row. If necessary, wrap around to the top of the column.

or

3) Both letters will form the corners of a rectangle within the key square:
   For each letter, substitute the one on the other corner the rectangle
   which lies on the same row.


The "TH" digram falls under case #3.
G H
M N
T U           (Rectangle with "T" and "H" at corners)

T --> U
H --> G


The "SE" digram falls under case #1.
C O D E S     (Row containing "S" and "E")

S --> C  (wraps around left to beginning of row)
E --> S

=========================================================================

To decrypt encrypted text, reverse the above procedure under cases #1
and #2 (move in opposite direction for substitution). Under case #3,
just take the remaining two corners of the rectangle.


Helen Fouche Gaines' classic work, ELEMENTARY CRYPTANALYSIS (1939), gives a
fairly detailed description of the Playfair Cipher and its solution methods.|

This script will have three main sections

1. Generating the _key square_, based on a user-input keyword.
    
2. Encrypting a _plaintext_ message.
    
3. Decrypting encrypted text.
    

The script will make extensive use of [[arrays#^ARRAYREF|arrays]] and [[functions|functions]]. You may use [[Example A-56|Example A-56]] as an inspiration.

--

Please do not send the author your solutions to these exercises. There are more appropriate ways to impress him with your cleverness, such as submitting bugfixes and suggestions for improving the book.

|[[writing-scripts#^AEN25254|[1]]]|For all you clever types who failed intermediate algebra, a _determinant_ is a numerical value associated with a multidimensional _matrix_ ([[arrays#^ARRAYREF|array]] of numbers).

|   |
|---|
|For the simple case of a 2 x 2 determinant:

  |a  b|
  |b  a|

The solution is a*a - b*b, where "a" and "b" represent numbers.||

