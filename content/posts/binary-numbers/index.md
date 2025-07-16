+++
title = 'Binary Numbers'
date = 2024-04-01T12:44:25+02:00
draft = false
summary = 'Discover how a series of simple 0s and 1s form the backbone of digital calculations. This article breaks down binary representation, shows how to translate it into familiar numbers, and explores the fascinating concept of bit shifting for efficient manipulation within Go programs.'
+++

{{< figure src="medium.webp" srcset="medium.webp 640w, large.webp 1280w" style="min-width: 250px; max-width:400px" width="100%">}}

So basically each 1 or 0 in binary number means the power of 2 

{{< emphasize >}}One{{< / emphasize >}} stand for yes and zero for {{< emphasize >}}No{{< / emphasize >}}.
It means if this power of two is present inside of our number or not.

**Lets build a 42 as the sum powers of 2:**

32 + 8 + 2 = **42**  
32 is **2**^5   
8 is **2**^3  
2 is **2**^1 

Starting from the rightmost digit, the weights are: 2^0, 2^1, 2^2, 2^3, 2^4, and so on.
The first right number in binary stands for 2 to power of 0  
The second right number in binary stands for 2 to power of 1   **...**

In our example we don't need 2^0 we need 2^1 this is second digit in binary number, but we don't skip the first digit we just set it as 0.
At this moment we have: {{< emphasize >}}'00000010' {{< / emphasize >}}. Then let's fill in third and fifth digits: {{< emphasize >}}'00101010' {{< / emphasize >}}. 

If we want get normal number out of binary ('00101010'):  
In this case, our weights will be: 2^0, 2^1, 2^2, 2^3, 2^4, and 2^5 (though the leftmost two digits are 0 and won't contribute).
Multiply each digit by its weight:
**2^1 + 2^3 + 2^5 = 42**


## Binary operators
**Shifting:** 
{{< highlight python >}}
n = 45
n = n << 1 # n is 90
{{< / highlight >}}

so we basically shift all numbers by 1 so it basically like multipliying by 2

{{< highlight python >}}
pow = 1
pow = pow << 2 
{{< / highlight >}}

Because **1** in binary is 2^0 or '00000001'. If we shift one to left we will increase power of two. **For example:**  
If we shift to the left by one ({{< emphasize >}}<< 1 {{< / emphasize >}}): '00000010', We will get 2^1. If we shift by 2 ({{< emphasize >}}<< 2 {{< / emphasize >}}): '0000100', We will get 2^2.
Shifting **1** means that we shift it to some power of two 
for example if we shift 1 by 2 it means shift to 4

## Use case
We can use binary operators for flags. For example, if in our application we will have following permissions: 
1. permission to write
2. permission to read
3. permission to execute
4. permission to update users  

We can say that permission to write would be first digit from the right, permission for read would be second digit from the right. If user would have this permission then we set 1, if not 0. This user would have all permissions {{< emphasize >}}'00001111' {{< / emphasize >}}. This user will have everything but execute, {{< emphasize >}}'00001101'{{< / emphasize >}}. 

Based on this Go has [Uint and Int](/posts/uint-and-int) types

