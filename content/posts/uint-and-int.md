+++
title = 'Uint and Int'
date = 2024-05-06T12:46:04+02:00
draft = false
summary = "Discover how Go uses 'int' and 'uint' to represent numbers. Explore how binary storage affects their capacity and why one handles negative values while the other doesn't."
+++

{{< blockquote >}}
In **Go** there a two types representing numbers (excluding floating point nubmers): uint and int. The maximum digital value you can store in **int** is 127.  In **uint**  is a 255.  Here is why ⬇️
{{< / blockquote >}}

Every programming language saves numbers in [Binary numbers](/posts/binary-numbers)

**Int** stands for all numbers positive and negative, when the **uint** in the opposite only for positive values from 0 to **max_value**.

So in **int** we spend one bit in front of every number for storing a sign of the number (1 for negative and 0 for positive). In **uint** we can use all 8 bits for the digits, that's why we can store more in **uint**.
