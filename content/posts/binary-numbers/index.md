+++
title = 'Binary Numbers'
date = 2024-05-06T12:44:25+02:00
draft = false
summary = 'Discover how a series of simple 0s and 1s form the backbone of digital calculations. This article breaks down binary representation, shows how to translate it into familiar numbers, and explores the fascinating concept of bit shifting for efficient manipulation within Go programs.'
+++

{{< figure src="image.png" style="min-width: 250px; max-width:400px" width="100%">}}

So basically each 1 or 0 in binary number means the power of 2 

{{< emphasize >}}One{{< / emphasize >}} stand for yes and zero for {{< emphasize >}}No{{< / emphasize >}}.
It means if this power of two is present inside of our number or not.

**Lets build a 42 as the sum powers of 2:**

32 + 8 + 2 = **42**  
32 is **2**^5   
8 is **2**^3  
2 is **2**^1 

- The first right number in binary stands for 2 to power of 0  
- The second right number in binary stands for 2 to power of 1   **...**

In our example we don't need 2^0 we need 2^1 this is second digit in binary number, but we don't skip the first digit we just set it as 0.
At this moment we have: 10

There also some binary operators 
**Shifting:** 
{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
n = 45
n = n << 1 # n is 90
{{< / highlight >}}

so we basically shift all numbers by 1 so it basically like multipliying by 2

{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
pow = 1
pow = pow << 2 
{{< / highlight >}}

So shifting **1** means that we shift it to some power of two 
for example if we shift 1 by 2 it means shift to 4

Based on this Go has [Uint and Int](/posts/uint-and-int) types

