---
title: 36.6. Optimizations
---


Most shell scripts are quick 'n dirty solutions to non-complex problems. As such, optimizing them for speed is not much of an issue. Consider the case, though, where a script carries out an important task, does it well, but runs too slowly. Rewriting it in a compiled language may not be a palatable option. The simplest fix would be to rewrite the parts of the script that slow it down. Is it possible to apply principles of code optimization even to a lowly shell script?

Check the loops in the script. Time consumed by repetitive operations adds up quickly. If at all possible, remove time-consuming operations from within loops.

Use [[../commands/internal-commands-and-builtins|builtin]] commands in preference to system commands. Builtins execute faster and usually do not launch a subshell when invoked.

Avoid unnecessary commands, particularly in a [[special-chars.html#PIPEREF|pipe]].

```bash
cat "$file" | grep "$word"

grep "$word" "$file"

#  The above command-lines have an identical effect,
#+ but the second runs faster since it launches one fewer subprocess.
```

The [[basic.html#CATREF|cat]] command seems especially prone to overuse in scripts.

> Disabling certain Bash options can speed up scripts.
>
> As Erik Brandsberg points out:
>
> If you don't need [Unicode](bashver4.html#UNICODEREF) support, you can get potentially a 2x or more improvement in speed by simply setting the **LC_ALL** variable.
>
> ```bash
>    export LC_ALL=C
> 
>    [specifies the locale as ANSI C,
>    thereby disabling Unicode support]
> 
> [In an example script ...]
> 
> Without [Unicode support]:
> erik@erik-desktop:~/capture$ time ./cap-ngrep.sh
> live2.pcap > out.txt
> 
>   real        0m20.483s
>   user        1m34.470s
>   sys         0m12.869s
> 
> With [Unicode support]:
> erik@erik-desktop:~/capture$ time ./cap-ngrep.sh
> live2.pcap > out.txt
> 
>   real        0m50.232s
>   user        3m51.118s
>   sys         0m11.221s
> 
> A large part of the overhead that is optimized is, I believe,
> regex match using [[ string =~ REGEX ]],
> but it may help with other portions of the code as well.
> I hadn't [seen it] mentioned that this optimization helped
> with Bash, but I had seen it helped with "grep,"
> so why not try?
> ```

> [!note]
> Certain operators, notably [[moreadv.html#EXPRREF|expr]], are very inefficient and might be replaced by [[dblparens.html|double parentheses]] arithmetic expansion. See [[contributed-scripts.html#TESTEXECTIME|Example A-59]].
>
> ```bash
> Math tests
> 
> math via $(( ))
> real          0m0.294s
> user          0m0.288s
> sys           0m0.008s
> 
> math via expr:
> real          1m17.879s   # Much slower!
> user          0m3.600s
> sys           0m8.765s
> 
> math via let:
> real          0m0.364s
> user          0m0.372s
> sys           0m0.000s
> ```
>
> [[tests.html#IFTHEN|Condition testing]] constructs in scripts deserve close scrutiny. Substitute [[testbranch.html#CASEESAC1|case]] for [[tests.html#IFTHEN|if-then]] constructs and combine tests when possible, to minimize script execution time. Again, refer to [[contributed-scripts.html#TESTEXECTIME|Example A-59]].
>
> ```bash
> Test using "case" construct:
> real          0m0.329s
> user          0m0.320s
> sys           0m0.000s
> 
> 
> Test with if [], no quotes:
> real          0m0.438s
> user          0m0.432s
> sys           0m0.008s
> 
> 
> Test with if [], quotes:
> real          0m0.476s
> user          0m0.452s
> sys           0m0.024s
> 
> 
> Test with if [], using -eq:
> real          0m0.457s
> user          0m0.456s
> sys           0m0.000s
> ```

> [!note] 
> Erik Brandsberg recommends using [[bashver4.html#ASSOCARR|associative arrays]] in preference to conventional numeric-indexed arrays in most cases. When overwriting values in a numeric array, there is a significant performance penalty vs. associative arrays. Running a test script confirms this. See [[contributed-scripts.html#ASSOCARRTEST|Example A-60]].
> 
> ```bash
> Assignment tests
> 
> Assigning a simple variable
> real          0m0.418s
> user          0m0.416s
> sys           0m0.004s
> 
> Assigning a numeric index array entry
> real          0m0.582s
> user          0m0.564s
> sys           0m0.016s
> 
> Overwriting a numeric index array entry
> real          0m21.931s
> user          0m21.913s
> sys           0m0.016s
> 
> Linear reading of numeric index array
> real          0m0.422s
> user          0m0.416s
> sys           0m0.004s
> 
> Assigning an associative array entry
> real          0m1.800s
> user          0m1.796s
> sys           0m0.004s
> 
> Overwriting an associative array entry
> real          0m1.798s
> user          0m1.784s
> sys           0m0.012s
> 
> Linear reading an associative array entry
> real          0m0.420s
> user          0m0.420s
> sys           0m0.000s
> 
> Assigning a random number to a simple variable
> real          0m0.402s
> user          0m0.388s
> sys           0m0.016s
> 
> Assigning a sparse numeric index array entry randomly into 64k cells
> real          0m12.678s
> user          0m12.649s
> sys           0m0.028s
> 
> Reading sparse numeric index array entry
> real          0m0.087s
> user          0m0.084s
> sys           0m0.000s
> 
> Assigning a sparse associative array entry randomly into 64k cells
> real          0m0.698s
> user          0m0.696s
> sys           0m0.004s
> 
> Reading sparse associative index array entry
> real          0m0.083s
> user          0m0.084s
> sys           0m0.000s
> ```

Use the [[timedate.html#TIMREF|time]] and [[x9644.html#TIMESREF|times]] tools to profile computation-intensive commands. Consider rewriting time-critical code sections in C, or even in assembler.

Try to minimize file I/O. Bash is not particularly efficient at handling files, so consider using more appropriate tools for this within the script, such as [[awk.html#AWKREF|awk]] or [[wrapper.html#PERLREF|Perl]].

Write your scripts in a modular and coherent form, [^1] so they can be reorganized and tightened up as necessary. Some of the optimization techniques applicable to high-level languages may work for scripts, but others, such as _loop unrolling_, are mostly irrelevant. Above all, use common sense.

For an excellent demonstration of how optimization can dramatically reduce the execution time of a script, see [[mathc.html#MONTHLYPMT|Example 16-47]].

[^1]: This usually means liberal use of [[./functions|functions]].
