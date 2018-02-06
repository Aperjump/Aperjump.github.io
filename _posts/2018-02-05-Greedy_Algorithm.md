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
Let `O` be the optimal set of intervals, and we compare `|A|` and `|O|`, since there may be multiple optimal solutions. This compare means the requests in set `A` is as much as elements in `O`. 
The core idea of the proof is to show that for each step, our choice can be better than solution `O`.(We call this **step ahead**)
