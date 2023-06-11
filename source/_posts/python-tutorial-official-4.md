---
title: Python Official Tutorial 4
date: 2023-06-08 14:07:39
tags: [python, tech]
---

This post is mainly set up for `classes` in python.

## Namespace and Scope

`local` variables vs. `non-local` variables vs. `global` variables.

``` python
def scope_test():
    def do_local():
        spam = "local spam"

    def do_nonlocal():
        nonlocal spam
        spam = "nonlocal spam"

    def do_global():
        global spam
        spam = "global spam"

    spam = "test spam"
    do_local()
    print("After local assignment:", spam)
    do_nonlocal()
    print("After nonlocal assignment:", spam)
    do_global()
    print("After global assignment:", spam)

scope_test()
print("In global scope:", spam)
```

The result is 

``` python
After local assignment: test spam
After nonlocal assignment: nonlocal spam
After global assignment: nonlocal spam
In global scope: global spam
```

Note how the local assignment (which is default) didn’t change scope_test's binding of spam. The nonlocal assignment changed scope_test's binding of spam, and the global assignment changed the module-level binding.

## Data attributes

Data attributes need not be declared; like local variables, they spring into existence when they are first assigned to.

``` python
x.counter = 1
while x.counter < 10:
    x.counter = x.counter * 2
print(x.counter)
# 16
```

## Class and Instance Variabies

Class variables are shared among all instances of the class.

``` python
class Dog:

    tricks = []             # mistaken use of a class variable

    def add_trick(self, trick):
        self.tricks.append(trick)

d = Dog()
e = Dog()
d.add_trick('roll over')
e.add_trick('play dead')
print(d.tricks)                # unexpectedly shared by all dogs
# ['roll over', 'play dead']
```

Instance variable is unique to each instance.

``` python
class Dog:

    def __init__(self):
        self.tricks = []    # creates a new empty list for each dog

    def add_trick(self, trick):
        self.tricks.append(trick)

d = Dog()
e = Dog()
d.add_trick('roll over')
e.add_trick('play dead')
print(d.tricks)
# ['roll over']
print(e.tricks)
# ['play dead']
```

If the same attribute name occurs in both an instance and in a class, then the instance variable is prioritized. 

``` python
w1 = Warehouse()
print(w1.purpose, w1.region)
# storage west
w2 = Warehouse()
w2.region = 'east'
print(w2.purpose, w2.region)
# storage east
```


In this case, assigning `region` will be treated as an instance variable.


## Odds and Ends

`dataclasses` are close to 'struct', which can be easier to implement.

``` python
from dataclasses import dataclass

@dataclass
class Employee:
    name: str
    dept: str
    salary: int
```


## Generator

`Generators` are written like regular functions but use the `yield` statement whenever users want to return data. 

``` python
def reverse(data):
    for index in range(len(data)-1, -1, -1):
        yield data[index]

for char in reverse('golf'):
    print(char, end='')

# flog
```





