---
title: 27. Arrays
---


Newer versions of Bash support one-dimensional arrays. Array elements may be initialized with the **`variable[xx]`** notation. Alternatively, a script may introduce the entire array by an explicit **declare -a variable** statement. To dereference (retrieve the contents of) an array element, use *curly bracket* notation, that is, **`${element[xx]}`**.

![[Example 27-1|Example 27-1]]

As we have seen, a convenient way of initializing an entire array is the `array=( element1 element2 ... elementN )` notation.

```bash
base64_charset=( {A..Z} {a..z} {0..9} + / = )
               #  Using extended brace expansion
               #+ to initialize the elements of the array.                
               #  Excerpted from vladz's "base64.sh" script
               #+ in the "Contributed Scripts" appendix.
```

> Bash permits array operations on variables, even if the variables are not explicitly declared as arrays.
>
> ```bash
> string=abcABC123ABCabc
> echo ${string[@]}               # abcABC123ABCabc
> echo ${string[*]}               # abcABC123ABCabc 
> echo ${string[0]}               # abcABC123ABCabc
> echo ${string[1]}               # No output!
>                                 # Why?
> echo ${#string[@]}              # 1
>                                 # One element in the array.
>                                 # The string itself.
> 
> # Thank you, Michael Zick, for pointing this out.
> ```
>
> Once again this demonstrates that [[variables-and-parameters#^BVUNTYPED|Bash variables are untyped]].

![[Example 27-2|Example 27-2]]

Array variables have a syntax all their own, and even standard Bash commands and operators have special options adapted for array use.

![[Example 27-3|Example 27-3]]

Many of the standard [[manipulating-variables#^STRINGMANIP|string operations]] work on arrays.

![[Example 27-4|Example 27-4]]

[[command-substitution#^COMMANDSUBREF|Command substitution]] can construct the individual elements of an array.

![[Example 27-5|Example 27-5]]

In an array context, some Bash [[internal-commands-and-builtins|builtins]] have a slightly altered meaning. For example, [[internal-commands-and-builtins#^UNSETREF|unset]] deletes array elements, or even an entire array.

![[Example 27-6|Example 27-6]]

As seen in the previous example, either **`${array_name[@]}`** or **`${array_name\[*]}`** refers to *all* the elements of the array. Similarly, to get a count of the number of elements in an array, use either **`${#array_name[@]}`** or **`${#array_name[*]}`**. **`${#array_name}`** is the length (number of characters) of **`${array_name[0]}`**, the first element of the array.

![[Example 27-7|Example 27-7]]

The relationship of **`${array_name[@]}`** and **`${array_name[*]}`** is analogous to that between [[another-look-at-variables#^APPREF|$@ and $*]]. This powerful array notation has a number of uses.

```bash
# Copying an array.
array2=( "${array1[@]}" )
# or
array2="${array1[@]}"
#
#  However, this fails with "sparse" arrays,
#+ arrays with holes (missing elements) in them,
#+ as Jochen DeSmet points out.
# ------------------------------------------
  array1[0]=0
# array1[1] not assigned
  array1[2]=2
  array2=( "${array1[@]}" )       # Copy it?

echo ${array2[0]}      # 0
echo ${array2[2]}      # (null), should be 2
# ------------------------------------------



# Adding an element to an array.
array=( "${array[@]}" "new element" )
# or
array[${#array[*]}]="new element"

# Thanks, S.C.
```

> [!tip]
> The **array=( element1 element2 ... elementN )** initialization operation, with the help of [[command-substitution#^COMMANDSUBREF|command substitution]], makes it possible to load the contents of a text file into an array.
>
> ```bash
> #!/bin/bash
> 
> filename=sample_file
> 
> #            cat sample_file
> #
> #            1 a b c
> #            2 d e fg
> 
> 
> declare -a array1
> 
> array1=( `cat "$filename"`)                #  Loads contents
> #         List file to stdout              #+ of $filename into array1.
> #
> #  array1=( `cat "$filename" | tr '\n' ' '`)
> #                            change linefeeds in file to spaces. 
> #  Not necessary because Bash does word splitting,
> #+ changing linefeeds to spaces.
> 
> echo ${array1[@]}            # List the array.
> #                              1 a b c 2 d e fg
> #
> #  Each whitespace-separated "word" in the file
> #+ has been assigned to an element of the array.
> 
> element_count=${#array1[*]}
> echo $element_count          # 8
> ```

Clever scripting makes it possible to add array operations.

![[Example 27-8|Example 27-8]]

> [!note]
> Adding a superfluous **declare -a** statement to an array declaration may speed up execution of subsequent operations on the array.

![[Example 27-9|Example 27-9]]

![[Example 27-10|Example 27-10]]

--

Arrays permit deploying old familiar algorithms as shell scripts. Whether this is necessarily a good idea is left for the reader to decide.

![[Example 27-11|Example 27-11]]

--

Is it possible to nest arrays within arrays?

```bash
#!/bin/bash
# "Nested" array.

#  Michael Zick provided this example,
#+ with corrections and clarifications by William Park.

AnArray=( $(ls --inode --ignore-backups --almost-all \
	--directory --full-time --color=none --time=status \
	--sort=time -l ${PWD} ) )  # Commands and options.

# Spaces are significant . . . and don't quote anything in the above.

SubArray=( ${AnArray[@]:11:1}  ${AnArray[@]:6:5} )
#  This array has six elements:
#+     SubArray=( [0]=${AnArray[11]} [1]=${AnArray[6]} [2]=${AnArray[7]}
#      [3]=${AnArray[8]} [4]=${AnArray[9]} [5]=${AnArray[10]} )
#
#  Arrays in Bash are (circularly) linked lists
#+ of type string (char *).
#  So, this isn't actually a nested array,
#+ but it's functionally similar.

echo "Current directory and date of last status change:"
echo "${SubArray[@]}"

exit 0
```

--

Embedded arrays in combination with [[bash-version-2-3-and-4#^VARREFNEW|indirect references]] create some fascinating possibilities

![[Example 27-12|Example 27-12]]

--

Arrays enable implementing a shell script version of the *Sieve of Eratosthenes*. Of course, a resource-intensive application of this nature should really be written in a compiled language, such as C. It runs excruciatingly slowly as a script.

![[Example 27-13|Example 27-13]]

![[Example 27-14|Example 27-14]]

Compare these array-based prime number generators with alternatives that do not use arrays, [[Example 16-46|Example 16-46]].

--

Arrays lend themselves, to some extent, to emulating data structures for which Bash has no native support.

![[Example 27-15|Example 27-15]]

--

Fancy manipulation of array "subscripts" may require intermediate variables. For projects involving this, again consider using a more powerful programming language, such as Perl or C.

![[Example 27-16|Example 27-16]]

--

Bash supports only one-dimensional arrays, though a little trickery permits simulating multi-dimensional ones.

![[Example 27-17|Example 27-17]]

A two-dimensional array is essentially equivalent to a one-dimensional one, but with additional addressing modes for referencing and manipulating the individual elements by *row* and *column* position.

For an even more elaborate example of simulating a two-dimensional array, see [[Example A-10|Example A-10]].

--

For more interesting scripts using arrays, see:

- [[Example 12-3|Example 12-3]]
- [[Example 16-46|Example 16-46]]
- [[Example A-22|Example A-22]]
- [[Example A-44|Example A-44]]
- [[Example A-41|Example A-41]]
- [[Example A-42|Example A-42]]
