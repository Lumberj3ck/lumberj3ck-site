---
title: "All Valid Permutes"
date: 2026-05-03T16:32:50+02:00
draft: false
toc: false
images:
tags:
  - algos 
---

Solving [this](https://leetcode.com/problems/permutations/description/) problem. The problem is as simple as it sounds; we are given a list of characters, we have to return a list of all valid permutations of these characters. Each element has to be used only once. This is typical backpropagation/dfs question, whenever the question is to generate something or aggregate across multiple branches we can try to draw state tree. 

{{< figure src="state-tree.png" style="min-width: 250px; max-width:400px; margin:0;" width="100%" alt="State tree diagram">}}

So the initial idea is to go through each available on this level letters and dfs from it. The only problem might be how to count left letters. 
For that the simplest what we can do is try `s[:i]+s[i+1:]`. We have to remember that in python `s[:i]` does not include `i` and because of that we can get this beauty `s[:i] + s[i:]`; what we get is the same full string `s`.
That is why we have to move the higher border which includes `i`.


{{< highlight python >}}
def permutations(letters: str) -> list[str]:
    n = len(letters)
    res = []
    def dfs(path, lett):
        if len(path) == n:
            res.append(path)
            return  

        for i in range(len(lett)):
            dfs(path + lett[i], lett[:i]+lett[i+1:])


    dfs("", letters)
    return res
{{< / highlight  >}}

The problem with this solution is that we have to allocate memory at each dfs branching for both of our arguments. To solve problem with second parameter we have to understand an idea, that we actually don't need to remember all unique left letters combinations. We can select one character to do dfs and just remember that we don't want to use it in all beneath branches. When we come back to this level we will remove it from restricted and can use it again. The DS for this could be **set()**, **map**, but for us enough to have simply a list, we know the index of the letter and by indexing in array we still get constant time.   
Solving problem with the `path` parameter is actually trivial, the problem is that strings are immutable and this way when we pass in a new param we allocate a new string. To do that we have to use some mutable DS like list. This is pattern used in many dfs questions, somehow there is also some look-alikes with the first approach we used. We will add characters on dfs when we come back from dfs we will remove the character, this way our maximum length might be `len(letters)`.  

{{< highlight python >}}
def permute_with_better_memory(nums: List[int]) -> List[List[int]]:
    letters = nums
    n = len(letters)
    res = []
    used = [False] * len(letters)
    def dfs(path: List[int]):
        if len(path) == n:
            res.append(path[:])
            return  

        for i in range(len(letters)):
            if used[i]:
                continue

            path.append(letters[i])
            # In all branches beneath it won't be available
            used[i] = True
            dfs(path)
            path.pop()

            used[i] = False

    dfs([])
    return res
{{< / highlight  >}}

