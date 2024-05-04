+++
title = 'Python Variables' 
date = 2022-05-04T10:24:53+02:00
draft = false
summary = 'Set is muttable cause we have methods like add or remove while frozenset is not. You can check this by calling dir from object and if there is some methods which mutate state it is mutable data type.'
+++

{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
numbers = [2, 1, 3, 4, 7]
numbers2 = numbers
name = "Trey"
{{< / highlight  >}}

Numbers2 is a link to numbers, so the difference that numbers2 is not totaly different object in memory it is just the same object which is linked to numbers. So whexn numbers is mutating we can see it also affects on numbers2.

{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
>>> numbers.pop()
7
>>> numbers
[2, 1, 3, 4]
>>> numbers2
[2, 1, 3, 4]
{{< / highlight  >}}

Here we can see changes in numbers not because variable changed but because of object changed.
## Two types of changing in Python 
{{< blockquote >}}
1. Mutation  ( changes objects state )
2. Change posession ( changes link to completely different object )
{{< / blockquote >}}

Is  {{< emphasize >}}operator{{< / emphasize >}} in Python checks if two variables is pointing to the same cell in memory

**==** operator in Python checks if two variables stores the same data 

Mutation is working only for mutable data types in Python, for example for int or string it will not . 

| Mutable | Immutable    |
| ------- | ------------ |
| List    | Integer      |
| Set     | Tupple       |
| Dict    | String       |
| Array   | Boolean      |
| ---     | Frozen set   |
| ---     | Float, Bytes |

Set is muttable cause we have methods like add or remove while frozenset is not. 
You can check this by calling dir from object and if there is some methods which mutate state it is mutable data type.

If type can be hashed than this data type is immutable

In this code:
- We declare the {{< emphasize >}} score {{< / emphasize >}} variable normally in main() with the line score := 20.
- Then in the line incrementScore(&score) we use the reference operator & to get a pointer to the score variable, and pass this pointer as the argument to incrementScore(). Remember, a pointer just contains a memory address – in this case it's the memory address of the score variable.
- When incrementScore() is executed, the parameter s contains a copy of this pointer. But this copy still holds the same memory address – the memory address of the score variable.
- In the line newScore := *s + 10 we use the dereference operator *s to 'read through' and get the underlying value at that memory address, and add ten to it.
- Then in the next line *s = newScore we use the dereference operator again to 'write through' and set newScore as the value at that memory address.
- The end result is that we've mutated the value at the memory address of the score variable. So when the program executes the final line of code, we get the output "The score is 30".
