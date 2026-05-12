---
title: "Coin Change"
date: 2026-05-12T12:50:00+02:00
draft: false
toc: false
images:
tags:
  - algos
---

We are given an array of coin denominations. Using these coins, we need to make up the target **amount** and return the minimum number of coins required. Examples:

```python
coins = [1, 2, 5]
amount = 11
# Output: 3 (5 + 5 + 1)
```

```python
coins = [2, 6]
amount = 16
# Output: 4 (6 + 6 + 2 + 2)
```

The first thing that comes to mind is a greedy approach: always take the largest coin. This can fail though, so there is no guarantee it will work.

A reliable approach is to explore all possible branches and take the best result. This is DFS plus memoization, because otherwise the recursion will revisit the same subproblems over and over.

This is a classic DFS question. The slightly tricky part is deciding what value the DFS should return so that memoization works. We want `dfs(rest)` to return the minimum number of coins needed to reach `0` from `rest`. My initial attempt returned the total coins along the current path, which doesn't memoize cleanly.
```python
def coin_change(coins: list[int], amount: int) -> float:
    def dfs(rest) -> float:
        if rest == 0:
            return 0
        elif rest <= 0:
            return math.inf

        mc = math.inf

        for child in coins:
            count = dfs(rest - child)
            if count == math.inf:
                continue
            mc = min(mc, count + 1)
        return mc
    r = dfs(amount)
    return -1 if r == math.inf else r
```
Another helpful trick is to use `math.inf` as the default value, because you can always take `min(...)`. I first tried using `-1`, but it quickly gets messy because you need extra checks like `mc != -1` before taking the minimum.

After that, adding memoization is straightforward:

```python
def coin_change(coins: list[int], amount: int) -> float:
    memo: list[float] = [-1] * (amount + 1)

    def dfs(rest) -> float:
        if rest == 0:
            return 0
        elif rest <= 0:
            return math.inf

        if memo[rest] != -1:
            return memo[rest]

        # Using `inf` means we can always take `min(...)`.
        # With `-1`, we'd need extra checks before taking the minimum.
        mc = math.inf

        for child in coins:
            count = dfs(rest - child)
            if count == math.inf:
                continue
            mc = min(mc, count + 1)

        memo[rest] = mc
        return mc
    r = dfs(amount)
    return -1 if r == math.inf else r
```

What we get is essentially top-down DP. The main disadvantage compared to bottom-up DP is that this solution makes a lot of function calls, whereas a bottom-up approach only uses loops.

If you look at the `memo` table, it ends up looking very similar to what you'd build with bottom-up DP. This helped me understand the difference between the two approaches. Bottom-up does mostly the same thing as top-down; it just iterates amounts and asks: "can we reach this amount from a smaller one?" If yes, we fill the table with `dp[a - coin] + 1` (the `+1` accounts for the coin we just used).


```python
def coin_change(coins, amount):
    INF = amount + 1
    dp = [INF] * (amount + 1)
    dp[0] = 0

    for a in range(1, amount + 1):
        for coin in coins:
            if a - coin >= 0:
                # dp[a-coin] -- previous layer  + 1 current layer
                dp[a] = min(dp[a], dp[a - coin] + 1)

    return dp[amount] if dp[amount] != INF else -1
```

The most interesting line here is `dp[a] = min(dp[a], dp[a - coin] + 1)`. We update the current amount only if `dp[a - coin]` is not `INF`. If that holds, we know the remainder is reachable, so we can build `a` by adding one more coin.

PS: I didn't write the bottom-up approach myself; I was mostly curious to understand how it works.
