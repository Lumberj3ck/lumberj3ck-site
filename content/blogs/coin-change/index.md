---
title: "Coin Change"
date: 2026-05-12T12:50:00+02:00
draft: false
toc: false
images:
tags:
  - algos
---

We are given array of numbers using this numbers we have to valid split the **amount** and return the minimum amount of numbers is required for split. Example:

```bash
coins = [1,2,5]
amount = 11
# Output 3  with 5, 5, 1 
```

```bash
coins = [2,6]
amount = 16
# Output 4  with 6, 6, 2, 2
```

First thing comes to mind in here is being greedy and taking the biggest value coin, but it will fail there is no posibility to have this work. The only posible thing is to go through all posible branches and aggregate on the results. It is dfs, plus we will have to have memoization, because there will be cases where branching is too deep. 
This is the default dfs question, however because of the memoization we have to return values to the previous calls for agggregation, this is where it might get tricky a bit. We have to decide from that we are going to return the minimum amount of coins needed to reach 0 (or the whole number, if you go otherwise), my initial approach returned total amount of coins in the path and because of that it didn't work with the memo.
```python
def coin_change(coins: list[int], amount: int) -> float:
    def dfs(rest, c)->float:
        if rest == 0:
            return 0
        elif rest <=0:
            return math.inf

        if memo[rest] != -1:
            return memo[rest]

        mc = math.inf

        for child in coins:
            count = dfs(rest-child, c+1)
            if count == math.inf:
                continue
            mc = min(mc, count+1)
        return mc
    r = dfs(amount, 0)
    return -1 if r == math.inf else r
```
The other helpfull thing is to pick `math.inf` as default value, because we always simply check if what returned is `inf`, otherwise always min. I did it otherwise on my first try and tried with default value `-1`, but this quickly gets into mess, when you have to check if `mc != -1` and only then check min. 
After that adding memo is trivial:

```python
def coin_change(coins: list[int], amount: int) -> float:
    memo:list[float] = [-1]*(amount+1)

    def dfs(rest, c)->float:
        if rest == 0:
            return 0
        elif rest <=0:
            return math.inf

        if memo[rest] != -1:
            return memo[rest]

        # better to go with the inf then we always need to check for min
        # with -1 we have to first check if -1 and then min 
        mc = math.inf

        for child in coins:
            count = dfs(rest-child, c+1)
            if count == math.inf:
                continue
            mc = min(mc, count+1)

        memo[rest] = mc
        return mc
    r = dfs(amount, 0)
    return -1 if r == math.inf else r
```

What we get is essentially Top to bottom DP, the only disadvantage of this approach in from of bottom up DP is that this solution makes a lot of calls, where as bottom up approach only utilises for loops. If you take a look into the memo we got it will be almost the same as we would get out of bottom up DP. Here I clearly understood this differntiation of this two approaches. Bottom Up does mostly the same thing as Top Bottom, it just checks "can we from current amount get to 0?", if yes, we will fill current amount in the table and place the value `dp[0] + 1`, with this `+1` we will account for our current coin we substracted.


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

Here the most interesting thing is this `dp[a] = min(dp[a], dp[a - coin] + 1)`, we update current amount only if we find jump from current amount to the other, for example `dp[amount - coin] != INF` if this condition holds we can be sure that from here we can use this coin and next path has been already calculated.  
PS I didn't wrote bottom up approach myself, I was mostly curious to know how it works.


