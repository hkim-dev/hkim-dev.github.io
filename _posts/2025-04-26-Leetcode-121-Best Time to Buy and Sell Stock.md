---
title: Leetcode 121 - Best Time to Buy and Sell Stock
date: 2025-04-26 20:00:00 -0400
categories: "Problem-Solving"
tags:
  - Leetcode
  - Python
toc: false
classes: wide
---

This problem is about finding the maximum possible profit, where each element represents the price of a stock on a given day. You are allowed to complete only one transaction - buy once and sell once.

## Problem Statement
Find the full problem statement of this Leetcode problem: [link](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/)

- **Given:** an array `prices`
- **Goal:** Return the maximum profit

## Thought Process
Initially, it felt like a simple two-pointer problem: one pointer for tracking minimum price, another for maximum price. Even though this approach seemed promising at first, I will discuss why it fails and walk through a case it does not handle properly.

```python
profit = 0
min_price = 10000
max_price = -1
for price in prices:
    if min_price > price:
        min_price = price
    elif max_price <= price:
        max_price = price
        profit = max(profit, max_price - min_price)

return profit
```

The above is the code I came up with at first. This idea worked for the examples given in the problem statement, but failed for a lot of cases I overlooked. The profit is calculated only in the `elif` statement, to make sure a transaction occurs only after a purchase. This code has a serious pitfall: it only accounts for the global maximum and therefore fails to capture possible local maximum profits.

Consider the following example:
`[3, 3, 5, 0, 0, 3, 1, 4]`

![prices](/assets/images/prices.png)

In this case, the local maximum is 5 at the index 2, however, the maximum profit is found at the index 7. This made me realize I needed to shift my perspective from a two-pointer approach to DP for a better solution. 



### Key Insight
It's important to remember that we are not looking for a global maximum price. Instead, after locking in a minimum purchase price, we are looking for the local maximum profit we can achieve afterwards. The strategy to track the minimum price so far still stands, however, the best profit should be computed at each point.
Even though there's no explicit DP table, the idea is actually dynamic programming because we store the previous state and update the optimal result at each step based on that state. This results in a space-optimized O(1) solution.

In short, we are:
- memorizing the lowest price as we move through the array,
- using it to calculate the maximum profit locally

## Final Solution

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        profit = 0
        min_price = 10000

        for price in prices:
            if min_price > price:
                min_price = price
                continue
            profit = max(profit, price - min_price)

        return profit
```

`continue` has been added under the `if` statement to ensure a calculation of the profit occurs after a purchase, but it could be omitted as it won't influence the output. Although it's labeled as an "Easy" problem, thinking about it in terms of dynamic programming helped me understand DP more intuitively.