---
title: "36.3. Tests and Comparisons: Alternatives"
---


For tests, the [[tests#^DBLBRACKETS|[[ | ]]]] construct may be more appropriate than **[[other-comparison-operators#^ICOMPARISON1| ]**. Likewise, [arithmetic comparisons]] might benefit from the [[operations-and-related-topics.html|(( ))]] construct.

```bash
a=8

# All of the comparisons below are equivalent.
test "$a" -lt 16 && echo "yes, $a < 16"         # "and list"
/bin/test "$a" -lt 16 && echo "yes, $a < 16" 
[ "$a" -lt 16 ] && echo "yes, $a < 16" 
[[ $a -lt 16 ]] && echo "yes, $a < 16"          # Quoting variables within
(( a < 16 )) && echo "yes, $a < 16"             # [[ ]] and (( )) not necessary.

city="New York"
# Again, all of the comparisons below are equivalent.
test "$city" \< Paris && echo "Yes, Paris is greater than $city"
                                  # Greater ASCII order.
/bin/test "$city" \< Paris && echo "Yes, Paris is greater than $city" 
[ "$city" \< Paris ] && echo "Yes, Paris is greater than $city" 
[[ $city < Paris ]] && echo "Yes, Paris is greater than $city"
                                  # Need not quote $city.

# Thank you, S.C.
```
