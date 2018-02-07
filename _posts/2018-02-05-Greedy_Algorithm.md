---
layout: post
title: "Greedy Algorithm"
description: "Greedy Algorithm"
categories: [Algorithm]
tags: [Algorithm]
redirect_from:
  - /2018/02/05/
---

## 1. Greedy Algorithm Proofs

### 1.1 The Greedy Algorithm Stays Ahead
Scheduling Problem: 
Suppose there are `n` requests, each has its `s(i)` start time and `f(i)` end time, we need to come up with an algo to accept the most requests.
The greedy rule which can work out this problem is to choose the earliest finish request in each iteration. 
```
// copy from book algorithm design
Initially get R to be the set of all requests, and let A be empty
while (!R.isempty())
	choose a request `i` that has the smallest finishing time.
	A.append(i)
	delete all requests from R that are not compatible with request i
end
return A
```
**Proof**

Let $O$ be the optimal set of intervals, and we compare $|A|$ and $|O|$, since there may be multiple optimal solutions. 
This compare means the requests in set $A$ is as much as elements in $O$. 

The core idea of the proof is to show that for each step, our choice can be better than solution $O$.(We call this **step ahead**)

Let $i_1, ..., i_k$ be the set of requests in $A$, and $j_1, ..., j_m$ in set $O$. 
We want to show that each of intervals in $A$ finishes at least as soon as the corresponding interval in the set $O$.

>**statement 1: for all indices $r \leq k$, we have $f(i_r) \leq f(j_r)$**

**Proof**: mathematical induction
(1) for $r = 1$ holds
(2) suppose $f(i_{r-1}) \leq f(j_{r-1})$, since $f(j_{r-1}) \leq s(j_r)$, so $f(i_{r-1}) \leq s(j_r)$. Thus the interval $j_r$ is in the set of available intervals. 
And the algo selects the smallest finish time, thus $f(i_r) \leq f(j_r)$. 

>**statement 2: the greedy algorithm returns an optimal set $A$**.

**Proof**: contradiction
If $A$ is not optimal, then an optimal set $O$ must have more requests, and $m > k$, since we already have $f(i_k) \leq f(j_k)$, there is another request $j_{k+1}$ in $O$. 

But this request finish after $f(i_k)$, thus our algorithm can still pick this request, thus contractdicts $m>k$. 
A related problem: if we have many identical resources available and we wish to schedule all the requests using as few resources as possible. 

![greedy_1](https://raw.githubusercontent.com/Aperjump/Aperjump.github.io/master/_picture/2018-02-05-Greedy_Algorithm/greedy_1.PNG)

Suppose we define the **depth** of a set of the interval to be the maximum number that pass over any single point on the time-line, we can calim: 

*In any instance of Interval Partitioning, the number of resources needed is at least the depth of the set of intervals*

We can now design a greedy algo that schedules all intervals using a number of resources equal to depth. 

The analysis of our algo will illustrate another general approach to proving optimallity: 
one finds a simple, "structural" bound asserting that every possible solution must have at least a certain value, and then one shows that the algorith under consideration always achieves this bound. 
algo:
Let $d$ be the depth of the set of intervals. 
```
Sort the intervals by start time
Let I_1, I_2, ..., I_n denote the intervals in this order
for j = 1:n
    for each interval I_t that p
    recedes I_j in sorted order and overlaps it 
        exclude the label of I_t from consideration for I_j
    if there is any label that has not been excluded then assign it to label I_j
    else 
        leave I_j unlabled
```
>**statement 1: no interval ends up unlabeled**
Consider one interval $I_j$, and suppose there are t intervals earlier in the sorted order that overlap it. 
These t intervals with $I_j$ form a set of $t+1$ intervals that all pass over a common point in time, which is bounded by $t+1 \leq d$.

>**statement 2: no two overlapping intervals are assigned to the same label**
If two are the same, during iteration, this label will be excluded. 

### 1.2 Exchange Argument
A list of requests, each only has a dealine $d_i$, and requires a contiguous time interval $t_i$, but it is willing to be scheduled at any time before the deadline. 
we can find following relationship:
$$ f(i) = s(i) + t_i$$, and we can define *lateness* as $l_i = f(i) - d_i$. 

Our goal in this optimization problem will be to schedule all requests, using nonoverlapping intervals to minimize the maximum latemness, $L = max_il_i$

We can sort the jobs in increasing order of their deadlines $d_i$. 
```
Order the jobs in order of their deadlines
Assume for simplicity of notation that d_1 <= ... <= d_n
Initially, f = s
Consider the jobs i = 1, ..., n in this order
Assign job j to the time interval from s(i) = f to f(i) = f + t_i
```

>**statement 1**: There is an optimal schedule with no idle time

Our plan here is to gradually modify $O$, preserving its optimality at each step but eventually transforming it into a schedule that is identical to the schedule $A$ found by the greedy algorithm. 
This approach we call it **exchange argument**
We say that a schedule $A'$ has an nversion if a job $i$ with deadline $d_i$ is scheduled before another job $j$ with earlier deadline $d_j \leq d_j$. 

>**statement 2**: All schedules with no inversions and no idle time have the same maximum lateness.

>**statement 3** There is an optimal schedule that has no inversions and no idle time. 

If $O$ has an inversion, then there is a pair of jobs $i$ and $j$ such that $j$ is $d_j < d_i$
If this exists, we can swap $i$ and $j$, and after swap this solution has a maximum lateness no larger than $O$.

>**statement 4** The schedule A produced by the greedy algo has optimal maximum lateness L.

### 1.3 Optimal Caching
Consider a set of $U$ of $n$ pieces of data stored in main memory, we also have a cache that can hold $k<n$ pieces of data at any one time. Assuming that cache initially holds some set of $k$ items. A sequence of data items $D=d_1, d_2, ..., d_m$ drawn from U is presented to us. 

When item $d_i$ is presented, we can access it very quickly if it is already in the cache, otherwise, we are required to bring it from main memory into cache and if the cache is full, to evict some other piece of data that is currently in the cache to make room for $d_i$. 

```
When d_i needs to be brough into the cache,
    evict the item that is needed the farthest into the future
```
When it is time to evict item, choose the one that will be used as late as possible. 
If d item is not the cache, bring it in step $i$, if there is a request for it.**schedule reduced**
One can imagine an algo that produced schedules which are not reduced, by bringring initems in steps when they are not requested. 

>**statement 1** $\bar{S}$ is a reduced schedule that brings in at most as many items as the schedule $S$

Consider an arbitrart seq $D$ of memory references, let $S_{FF}$ denote the schedule produced by farthest in future, let $S^*$ denote a schedule that incurs the minimum possible number of misses. 
>**statement 2** : Let $S$ be a reduced schedule that makes the same eviction decisions as $S_{FF}$ through the first $j$ items in the sequence, for a number $j$. Then there is a reduced schedule $S'$ that makes the same eviction decisions as $S_{FF}$ through the first $j+1$ items, and incurs no more misses than $S$ does. 

This proof is verbose in book **Algorithm Design**, so I will omit it here. 
### 1.4 Shorted Path in a Graph
We have a directed graph $G=(V,E)$, with a designated start node $s$, and we asssume $s$ has a path to every node in $G$. Each edge $e$ has a length $l_e \geq 0$. For a path $P$, the length of $P$ -denoted $l(P)$ is the sum of the lengths of all edges in $P$. Our goal is to determine the shortest ath from s to every other node in the graqph. 
```
Dijkstra's Algo(G, l)
Let S be the set of explored nodes
    For each $u \in S$, we explore a distance $d(u)$
Initially $S = {s}$, $d(s) = 0$
While $S\neq V$
    Select a node $v \in S$ with at least one eduge from $S$ for which $d'(v) = min_{e = (u,v);u \in S}(d(u) + l_e)$ is as small as possible
    Add $v$ to $S$ and define $d(v) = d'(v)$
EndWhile
```
**Proof**
>**statement 1**: Consider the set S at any point in the algorithm's execution. For each $u \in S$, the path $P_u$ is a shortest $s-u$ path.

Proof by induction:
When $|S| = 1$ is easy, suppose $|S| = k$, and we add new points $v$. Let $(u,v)$ be the final edge on our $P_v$. Consider another $s-v$ path, there must be some point $y$ not in set $S$, let $x$ be the node just before leave $S$. At iteration $k+1$, algo must consider adding $y$ to $S$ and reject $v$. This eans that there is no path from $s$ to $y$ through $x$ which is shorter than $(s,u) + (u,v)$

Using a priority queue, Dijkstra's Algo can be implementented on a graph with n nodes and m edges to run in $O(m)$ time, plus the time for $n$ `ExtractMin` and $m$ `ChangeKey` operations. 

### 1.6 Minimum Spanning Tree
We have a set of locations $V = \{v_1, v_2, ... , v_n\}$, for certain pairs$(v_i,v_j)$, we may build a direct link between $v_i$ and $v_j$ for a certain cost $c(v_i,v_j) > 0$. The problem is to find a subset of edges so that the graph is connected, and the total cost is as small as possible. We call a subset $T \subseteq E$ a spanning tree of $G$ if $(V, T)$ is a tree. 

(1) Kruskal's Algorithm:
Insert edges from E in order of increasing cost, if we meet a cycle discard that edge and move on.

(2) Prim's Algorithm: 
Mainatin a set $S \subseteq V$ on which a spanning tree has been constructed so far. In each iteration, we grow $S$ by one node, adding the node that minimize the cost $min_{e = (u,v):u \in S}C_e$ and include the edge $e = (u,v)$ in the spanning tree.

(3) Reverse Delete Algorithm:
Start with the full graph $(V,E)$ and begin deleting edges in order of decreasing cost. As we get to each edge $e$, we delete it as long as doing so would not actually disconnect the graph we currently have. 

>**statement 1** Assume that all edge costs are distinct. Let $S$ be any subset of nodes that is neither empty nor equal to all of $V$, and let edge $e = (v,w)$ be the minimum cost edge with one end in $S$ and the other in $V-S$. Then every minimum spanning tree contains the edge $e$. 

**Proof**: We can find another edge $e'$ satisfies the same property but $e' \geq e$, and we can swap $e$ for $e'$, and this time the overall connecting cost decreases. 
The difficulty is to prove the new tree is still connected(the two vertice in the set is connected)

>**statement 2** Kruskal's Algo produces a minimum spanning tree of $G$

Consider any edge $e = (v,w)$ added by Krukal's Algo, and let $S$ be the set of all nodes to which $v$ has a path at the moment just before $e$ is added. Clearly $v \in S$, but $w \notin S$, since adding $e$ does not create a cycle. Moreover, no edge from $S$ TO $V-S$ has been encountered, this $e$ satisfy the following property. 
