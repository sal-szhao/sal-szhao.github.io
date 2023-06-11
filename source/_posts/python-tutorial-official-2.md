---
title: Python Official Tutorial 2
date: 2023-06-07 17:09:41
tags: [python, tech]
---

This blog post will contain function-related and module-related codes in python.

## Functions

### Default values

**Warning**: The default value is evaluated only once. This makes a difference when the default is a mutable object such as a list, dictionary, or instances of most classes. (Default is like a once initialization)

``` python
a = 1

def f(a, L=[]):
    L.append(a)
    return L

print(f(1))
print(f(2))
print(f(3))
```

will print

```
[1]
[1, 2]
[1, 2, 3]
```

The solution will be

``` python
def f(a, L=None):
    if L is None: L = []
    L.append(a)
    return L
```

### Keyword arguments


`keyword` arguments should appear after `positional` arguments.

``` python
def parrot(voltage, state='a stiff', action='voom', type='Norwegian Blue'):
    print("-- This parrot wouldn't", action, end=' ')
    print("if you put", voltage, "volts through it.")
    print("-- Lovely plumage, the", type)
    print("-- It's", state, "!")
```

The following will work.

``` python
parrot('a thousand', state='pushing up the daisies')
```

The following will not work.

``` python
parrot(voltage=5.0, 'dead')
```


### Special parameters

Formal parameter `*name` receives a `tuple` containing `positional` arguments.

Formal parameter `**name` receives a `dict` containing `keyword` arguments.

``` python
def cheeseshop(*positions, **keywords):
    for pos in positions:
        print(pos)
    print("-" * 40)
    for kw in keywords:
        print(kw, ":", keywords[kw])

cheeseshop("It's very runny, sir.",
           "It's really very, VERY runny, sir.",
           shopkeeper="Michael Palin",
           client="John Cleese",
           sketch="Cheese Shop Sketch")
```

The result will be

``` python
It's very runny, sir.
It's really very, VERY runny, sir.
----------------------------------------
shopkeeper : Michael Palin
client : John Cleese
sketch : Cheese Shop Sketch
```

Arguments before `/` will be `positional-only` arguments.

Arguments after `*` will be `keyword-only` arguments.

The following is a sample of how to use `/` and `*` in function arguments.
``` python
def combined_example(pos_only, /, standard, *, kwd_only):
    print(pos_only, standard, kwd_only)
```

### Function annotations

`__annotations__` attribute will print the input type and output type of the function.

``` python
def f(ham: str, eggs: str = 'eggs') -> str:
    print("Annotations:", f.__annotations__)

f('spam')
```

will yield the result:

```
Annotations: {'ham': <class 'str'>, 'eggs': <class 'str'>, 'return': <class 'str'>}
```

## Modules and Packages

### Import Modules

There are serveral ways if importing modules from local python scripts.

``` python
>>> import fibo
>>> from fibo import fib, fib2
>>> from fibo import *
>>> import fibo as fib
```

### Search paths

When importing modules, python will search `sys.builtin_module_names`. If not found, then it will search `fibo.py` in a list of directories given by `sys.path`.


### Compiled cache

To speed up loading modules, Python caches the compiled version of each module in the `__pycache__` directory under the name module.version.pyc.

### Standard modules

First and second prompts:
``` python 
>>> import sys

>>> sys.ps1
'>>> '
>>> sys.ps2
'... '
```

Add to system path:

``` python
>>> import sys
>>> sys.path.append('/ufs/guido/lib/python')
```

`dir()` function will show the names that a module defines.

``` python
>>> import fibo
>>> dir(fibo)
['__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'fib', 'fib2']
```

There are also other `os` commands as following.

``` python
>>> import os                                     
>>> os.getcwd()                             # Return the current working directory
>>> os.chdir('/server/accesslogs')          # Change current working directory
>>> os.system('mkdir today')                # Run the command mkdir in the system shell
```


### Import packages

Packages are a way of structuring Python’s module namespace by using “dotted module names”. Packages should have separate folders with `__init__.py` file.

``` python
>>> from fibo.fibo import fib, fib2
>>> print(fib(1000))
0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 
```