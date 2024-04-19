+++
title = 'Hash Table Performance and Implementation'
date = 2024-03-26T15:34:47+05:30
draft = true
ShowToc = true
+++

Hash Table gives constant time performance when there are zero collisions.

*But that's impossible!*

So how can we quantify how "full" the table is?
This is called the **Load Factor**

## Load Factor

Load Factor is the ratio of the total number of elements and the number of slots in the hash table.

### Load Factor with different resolution strategies

#### Load Factor with different resolution strategies

1. With chaining, we can never fill the table i.e. we can always add a new node to list

In Chained hashing 

Load Factor  `α`  = `Average number of elements stored per linked list`

Time to resolve collision = `O(1 + α)`

2. With open addressing, we would eventually run out of space, as after each collision, we are probing to find a free slot and would fil that, and due to this writes would fail. 
> In both cases performance would go down much before operations start to degrade.

<!--  make graphs of timestamp 7:46 in excalidraw-->

### Which is the best strategy then?

Depending on the probing strategy, the cost of each probe changes, and it matters.

1. Probing is costly with chained hashing due to:
   1. Linear traversal of linked list
   2. Random memory access
2. Double hashing required us to evaluate 2 hash functions due to:
   1. CPU intensive and time consuming
   2. Random jump in array, and this is not cache  friendly.

### How to compare and benchmark the strategies?

If we are re-implementing Hash Tables, it is important to know how good we did, and hence analyze, benchmark and tune.

### Lookup time as a function of load

For your strategy identify how lookup time changes as a function of load factor against all strategies

1. Create a Hash Table of size 1024
2. Insert `n` elements varying in size.
3. Lookup 1000 random keys (high miss ratio)

We should see:
1. Performance for open addressing degrades as `α -> 1` i.e. when the slots start getting full.
2. Chained approach degrades gracefully.
3. Linear probing would be slower than double hashing
4. Probes of Double Hashing wil be shorter

**NOTE**: We can't conclude, Chained Hashing >> Open Addressing because chained hashing is not very cache friendly.

## How to make Cache perform better in Chained Hashing

To leverage cache in chained hashing, we can allocate pool for each slot so that the nodes of list are close to each other in memory.
<!-- make diagram from 21:50 -->

**Open Addressing outperforms chained hashing when tables are large enough**

So if Hash Table performance matters a lot, experiment with different strategies, parameters and algorithms to tune it so it works best for your use case.

## Resizing a Hash Table

*Performance of hash tables degrade as load factor increases* and hence, at a certain threshold, to maintain the performance of our hash table we have to resize.

**When should we resize?**

Resize when load factor is "too high" and it depends on the underlying structure and algorithm.

We cannot be too aggressive as we would waste too much time resizing, nor too lenient, which would degrade the performance.

A common decision is 
`resize when α = 0.5 i.e array is half filled`

But we ccan make this logic as complex as possible.

## How to resize?

1. Create a new array.
2. Copy elements from old to the new array
3. Delete old array