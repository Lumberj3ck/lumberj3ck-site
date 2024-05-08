+++
title = 'Python Scope'
date = 2024-05-03T11:36:52+02:00
draft = false
summary = "Learn how Python handles variables in different scopes. Avoid errors and write predictable code by understanding how functions access and modify variables."
+++
This is a note about python Scopes.

So there is a global scope everything which is outside of a function and there also local scope which is inside of a function. 

{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
# they are threating each other as the different namespaces
temp = 'global'
def change():
    temp = 'local'
    print(temp) # 'local'
print(temp) # 'global'

def change():
    global temp
    temp = 'local'
print(temp) # local
{{< / highlight >}}

If we want to overwrite this behaviour we should use **global** operator

The same behaviour we would see if we try to change a variable which is defined inside of a function. 

{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
def main():
    temp = 'main local'
    def wrapper():
        temp = 'wrapper local'
        print("Inside of a wrapper", temp)
    wrapper() # Inside of a wrapper local temp
    print("Inside of main", temp) # Inside of main value
{{< / highlight >}}

But if we want to access a variable, which is instantiated inside of a local scope (inside of a function) we need to use a **local** operator

{{< highlight python "linenos=table,hl_lines=,linenostart=1" >}}
def main():
    temp = 'main local'
    def wrapper():
        local temp 
        # We used a local operator,
        # now we can impact the variable defined inside of main 
        temp = 'wrapper local'
{{< / highlight >}}
