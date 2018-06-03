---
layout: post
title: "Dynamic Programming"
description: "Dynamic Programming"
categories: [Algorithm]
tags: [Algorithm]
redirect_from:
  - /2018/06/02/
---

## Dynamic Programming
1. 空间优化
2. 局部优化和全局最优实现时间优化
3. 记忆化搜索

动态规划的4要点
- 状态
a) 最优解/Max/Min
b) Yes/No
c) Count(*)
- 方程(状态之间的联系)
- 初始化
- 答案

1. 动态规划的空间规划
House Robber
[website](https://www.lintcode.com/problem/house-robber/description)
- 状态`f[i]`
`f[i]` 表示前i个房子中，偷到的最大价值
- `f[i] = max(f[i-1], f[i-2]+A[i])`
- `f[0] = A[0]`, `f[1] = max(A[0],A[1])`
- `f[n-1]`


一维优化特点`f[i]`是由`f[i-1]`和`f[i-2]`决定，这样可以转化为`f[i%2]`由`f[(i-1)%2]`和`f[(i-2)%2]`决定
- 状态state
`f[i]`表示前i个方子中，**偷了第i个房子的**，偷到的最大价值
- 方程
`f[i] = max(f[i-2], f[i-3]) + A[i]`
- 初始化
`f[0] = A[0]; f[1] = A[1]; f[2] = A[0] + A[2];`
- 答案
`max{f[i]}`

可以转化为`f[i%3]`由`f[(i-2)%3]`和`f[(i-3)%3]`决定

Maximal Square
[website](https://www.lintcode.com/problem/maximal-square/description)
- 状态
`f[i][j]` 表示以`i`,`j`为正方形右下角可以扩展的最大边长
- 方程
```
if matrix[i][j] == 1
	f[i][j] = min(f[i-1][j], f[i][j-1], f[i-1][j-1]) + 1;
if matrix[i][j] == 0
	f[i][j] = 0
```
- 初始化
```
f[i][0] = matrix[i][0]
f[0][j] = matrix[0][j]
```
- 答案
`max{f[i][j]}`

可以用`dp[2][n]`的矩阵即可
```
if matrix[i][j] == 1
	f[i%2][j] = min(f[(i-1)%2][j], f[i%2][j-1], f[(i-1)%2][j-1]) + 1;
if matrix[i][j] = 0
	f[i%2][j] = 0
```

2. 局部最优和全局最优

maximum subarray 
[website](https://www.lintcode.com/problem/maximum-subarray/description)
use presum array
```
for (j = 0->n)
	for (i = 0->j)
		presum[j+1] - presum[i](add dummy 0)
```
1. 状态
`local[i]`表示包括第i个元素能够找到的最大值(一定要取第i个元素能够取得的最大值)
`global[i]`表示全局前i个元素中能够找到的最大值
2. 方程
`local[i] = Max(nums[i], local[i-1]+nums[i]);`
`global[i] = Max(local[i], global[i-1]);`
3. 初始化
`loacl[0] = global[0] = nums[0]`
4. 答案
`global[n-1]`

maximum product subarray
[website](https://www.lintcode.com/problem/maximum-product-subarray/description)
use preproduct array
最初加一个1
```
for (j = 0->n)
	for (i = 0->j)
		preproduct[j+1] / preproduct[i]
```
need to record max, min
1. 状态
`min[i]` 表示前i个数包括第i个数找到的最小乘积
`max[i]`表示前i个数包括第i个数找到的最大乘积
2. 方程
```
min[i] = Min(nums[i], Min(min[i-1] * nums[i], max[i-1] * nums[i]));
max[i] = Max(nums[i], Max(max[i-1] * nums[i], min[i-1] * nums[i]));
global[i] = Max(global[i-1], max[i]);
```
3. 初始化
`min[0] = max[0] = nums[0];`
4. 答案
`gloabl[n-1]`

Best time to buy and sell stocks IV 
[website](https://www.lintcode.com/problem/best-time-to-buy-and-sell-stock-iv/description)

1. 状态
`f[i][j]`表示前i天进行j次交易能够获得的最大收益
2. 方程
`f[i][j] = max{f[x][j-1] + profit(x+1, i)} {x=0->i-1}`
3. 初始化
`f[i][0] = 0, f[0][i] = -INT_MAX`
4. 答案
`f[n-1][k]`
 
 A second method:
 1. 状态
`mustSell[i][j]`表示前i天，最多进行j次交易，前i天必须sell的最大获益
`globalbest[i][j]`表示前i天，最多进行j次交易，第i天可以不sell的最大获益
2. 方程
```
gain = prices[i] - prices[i-1]
mustsell[i][j] = Max(globalbest[i-1][j-1] + gain, mustsell[i-1][j]+gain)
globalbest[i][j] = Max(globalbest[i-1][j], mustsell[i][j])
```
3. 初始化
```
mustsell[0][i] = globalbest[0][j] = 0;
```
4. 答案
```
globalbest[n-1][k]
```




3. 记忆化搜索

longest increasing continuous 