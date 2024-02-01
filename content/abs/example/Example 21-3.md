**Example 21-3.** Running parallel processes in subshells

```bash
(cat list1 list2 list3 | sort | uniq > list123) &
	(cat list4 list5 list6 | sort | uniq > list456) &
	# Merges and sorts both sets of lists simultaneously.
	# Running in background ensures parallel execution.
	#
	# Same effect as
	#   cat list1 list2 list3 | sort | uniq > list123 &
	#   cat list4 list5 list6 | sort | uniq > list456 &
	
	wait   # Don't execute the next command until subshells finish.
	
	diff list123 list456
```
