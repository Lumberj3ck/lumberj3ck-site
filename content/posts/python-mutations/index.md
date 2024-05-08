+++
title = 'Python Mutations'
date = 2024-05-05T11:36:52+02:00
draft = false
summary = " Understand mutability in Python. Learn how variables can impact each other and how to manage these changes for predictable code. Covers lists, dictionaries, strings, and more."
+++


Have you ever encountered unexpected results in your Python code when variables seemed to change in tandem? This article delves into the concepts of mutability and how changes in one variable can sometimes propagate to others. We'll explain why this behavior occurs and how to manage it effectively.
Let's explore this with a simple example:
{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
numbers = [2, 1, 3, 4, 7]
# points to the same object
numbers2 = numbers 
{{< / highlight  >}}


 {{< emphasize >}} numbers2 {{< / emphasize >}} is not a distinct object; it's a link to  {{< emphasize >}} numbers {{< / emphasize >}}. Any changes to `numbers` will reflect in `numbers2`.

Let's change {{< emphasize >}} numbers {{< / emphasize >}} list:
{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
numbers.pop()
# Output: 7
{{< / highlight  >}}


Now, check both lists:

{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
print(numbers)   # Output: [2, 1, 3, 4] 
print(numbers2)  # Output: [2, 1, 3, 4]
{{< / highlight  >}}



Here, changes in {{< emphasize >}} numbers {{< / emphasize >}} impact {{< emphasize >}} numbers2 {{< / emphasize >}} because they both point to the same memory location and the same object.

{{< blockquote >}}
**Note**: If you need to create a distinct object you need to use python **copy** module
{{< / blockquote >}}

However this works differently with immutable data types:
{{< highlight python "linenos=tables, linostart=1" >}}
name = "Walt"

# second_name points to the same location as the name
second_name = name 

# Here we create completely new object in memory  
second_name = "Jesse" 
print(name) # Walt
print(second_name)  # Jesse
{{< / highlight >}}

This behaviour may cause unpredictable erorrs if you don't know about it.


💡 **Two Types of Changes in Python:**

1. **Mutation:** Alters an object's state. Example: modifying a list.
2. **Change of Possession:** Links to a completely different object.

### **1. Mutation:**

Mutation involves altering an object's state. This works for mutable data types like **lists**, **sets**, and **dictionaries** etc.
{{< blockquote >}}
**Note**: You can understand that data type is mutable, if data type has "alter" methods.   
For example:  **dir(list)**  (append method);  **dir(set)**    (pop method).

{{< / blockquote >}}


### Example 1 - Mutation in Lists:

{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
# Mutable data type: List
numbers = [1, 2, 3, 4]

# Mutation: Changing the list in-place
numbers.append(5)

# Output: [1, 2, 3, 4, 5]
print(numbers)

{{< / highlight  >}}

### Example 2 - Mutation in Dictionaries:

{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
# Mutable data type: Dictionary
person = {'name': 'Alice', 'age': 25}

# Mutation: Modifying the dictionary in-place
person['age'] = 26

# Output: {'name': 'Alice', 'age': 26}
print(person)

{{< / highlight  >}}

### **2. Change of Possession:**

Change of possession involves linking to a completely different object. This is more common with immutable data types.

### Example 1 - Change of Possession with Strings:
{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
# Immutable data type: String
greeting = "Hello"

# Change of Possession: Creating a new string
greeting = greeting + ", World!"

# Output: Hello, World!
print(greeting)
{{< / highlight  >}}


### Example 2 - Change of Possession with Tuples:
{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
# Immutable data type: Tuple
coordinates = (3, 4)

# Change of Possession: Creating a new tuple
coordinates = (5, 6)

# Output: (5, 6)
print(coordinates)

{{< / highlight  >}}  


### Python Operators: 🤔 

**is**: Checks if two variables point to the same memory location.  
**==**: Checks if two variables store the same data.

{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
import copy

numbers = [1, 2, 3]
# here we create a new object with 
# the same data as in numbers
copy_of_numbers = copy.copy(numbers)

# just a link
link_to_numbers = numbers

numbers is copy_of_numbers # False they're two separate obj
numbers is link_to_numbers # True they're same
{{< / highlight >}}


🛠️ **Mutability in Python:**

- **Mutable:** Lists, Sets, Dicts, Arrays, etc.
- **Immutable:** Integers, Tuples, Strings, Booleans, Floats, Bytes, etc.
