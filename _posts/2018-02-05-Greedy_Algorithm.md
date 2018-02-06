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
Let $O$ be the optimal set of intervals, and we compare $|A|$ and $|O|$, since there may be multiple optimal solutions. This compare means the requests in set $A$ is as much as elements in $O$. 
The core idea of the proof is to show that for each step, our choice can be better than solution $O$.(We call this **step ahead**)Let $i_1, ..., i_k$ be the set of requests in $A$, and $j_1, ..., j_m$ in set $O$. 
We want to show that each of intervals in $A$ finishes at least as soon as the corresponding interval in the set $O$. 
**statement 1: for all indices $r \leq k$, we have $f(i_r) \leq f(j_r)$**
**Proof**: mathematical induction
(1) for $r = 1$ holds
(2) suppose $f(i_{r-1}) \leq f(j_{r-1})$, since $f(j_{r-1}) \leq s(j_r)$, so $f(i_{r-1}) \leq s(j_r)$. Thus the interval $j_r$ is in the set of available intervals. And the algo selects the smallest finish time, thus $f(i_r) \leq f(j_r)$. 
**statement 2: the greedy algorithm returns an optimal set $A$**.
**Proof**: contradiction
If $A$ is not optimal, then an optimal set $O$ must have more requests, and $m > k$, since we already have $f(i_k) \leq f(j_k)$, there is another request $j_{k+1}$ in $O$. But this request finish after $f(i_k)$, thus our algorithm can still pick this request, thus contractdicts $m>k$. 
