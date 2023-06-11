---
title: Python Official Tutorial 3
date: 2023-06-08 10:27:07
tags: [python, tech]
---

In this post basic python data structures will be included.

## Strings

### Backslahes

``` python
>>> 'doesn\'t'  # use \' to escape the single quote...
"doesn't"
>>> "doesn't"  # ...or use double quotes instead
"doesn't"
```

to display single quotes inside the string output.

``` python
>>> s = 'First line.\nSecond line.'  # \n means newline

>>> s  # without print(), \n is included in the output
'First line.\nSecond line.'

>>> print(s)  # with print(), \n produces a new line
First line.
Second line.
```

`print` will encode backslahes `\`. But variable will not.


### Raw string

``` python
>>> print('C:\some\name')  # here \n means newline!
C:\some
ame
>>> print(r'C:\some\name')  # note the r before the quote
C:\some\name
```

`raw string` with `r` before the string will disable encoding the backslashes `\`.

### Multiple lines

Use triple quotes `"""..."""` or `'''...'''` to display texts with multiple lines.

``` python
print("""
Usage: thingy [OPTIONS]
    -h                        Display this usage message
    -H hostname               Hostname to connect to
""")
```


### Concatenation

Operators can be used between strings.

``` python
>>> 3 * 'un' + 'ium'
'unununium'
```

Automatic concatenation
``` python
>>> 'Py' 'thon'
'Python'
```
can be useful when the strings are long. But will only work when both are strings.

Concatenate a variable and a string using `+`.

``` python
>>> prefix = 'py'
>>> prefix + 'thon'
'python'
```

### String Assignment

``` python
>>> word = 'python'
>>> word[0] = 'a'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'str' object does not support item assignment
```

Python strings are immutable and cannot be assigned with new values.

### Print

`print` has attribute `end` which will add to the end of the print result.

``` python
print('abc', end='---')
print('xyz')
# abc---xyz
```

`print` can print multiple objects with attribute `sep`.

``` python
i = 100
print('apple', i, 0.123, sep='----')
# apple----100----0.123
```

`*l` will print the list without square brackets. `.join` can concatenate all the elements inside the list.

``` python
l = ['a', 'b', 'c']

print(*l)
# a b c
print('-'.join(l))
# a-b-c
```

Insert variables into the string using `%` or `format`.

``` python
s = 'Alice'
i = 25

print('%s is %d years old' % (s, i))
# Alice is 25 years old

print('{} is {} years old'.format(s, i))
# Alice is 25 years old

print(f'{s} is {i} years old')
# Alice is 25 years old
```

The last method is also known as `f-strings`. `f-strings` can also be used in displaying numeric variables by certain number of digits.

``` python 
number = 0.45

print(f'{number:.4f} is {number:.2%}')
# 0.4500 is 45.00%
```


## Lists

### Concatenation

Similar to strings, a list variable and a list can be concatenated using `+`.
``` python
>>> lst = [1, 2, 3, 4]
>>> lst + [5, 6, 7, 8] 
[1, 2, 3, 4, 5, 6, 7, 8]
```


### Delete elements

The following is a quick way of deleting specific elements in a list.

``` python
>>> lst[1:3] = []
>>> lst
[1, 4]
```

`clear` will clear all the elements in the list.

``` python
>>> list1.clear()
>>> list1
[]
```

Equivalent to `del`.

``` python
>>> del(list1[:])
>>> list1
[]
```

### Using lists as queue

``` python
>>> from collections import deque
>>> queue = deque(["Eric", "John", "Michael"])
>>> queue.append("Terry")          
>>> queue.append("Graham")          
>>> queue.popleft()
'Eric'
>>> queue.popleft()    
'John'
```

### Other usage

`map` can be used with `lambda` function to create a list.

``` python
>>> numbers1 = [1, 2, 3]
>>> numbers2 = [4, 5, 6]
>>> list(map(lambda x, y: x + y, numbers1, numbers2))
[5, 7, 9]
```

`extend` can be used to extend list, but is an in-place modification.

``` python
>>> list1 = [1, 2]
>>> list2 = [3, 4]
>>> list1.extend(list2)
>>> print(list1)
[1, 2, 3, 4]
```

## Tuples

`tuple` nesting can be done as following,

``` python
>>> t = 12345, 54321, 'hello!'
>>> u = t, (1, 2, 3, 4, 5)
>>> u
((12345, 54321, 'hello!'), (1, 2, 3, 4, 5))
```

## Sets

`set` can be initialized using curly braces or the set() function with no duplicate elements.

``` python
>>> basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
>>> basket
{'orange', 'pear', 'banana', 'apple'}

>>> a = set('abracadabra')
>>> a
{'c', 'b', 'a', 'r', 'd'}
```

## Dict

There are several ways of initializing a new `dict`.

``` python
>>> tel = {'jack': 4098, 'sape': 4139}
>>> dict([('sape', 4139), ('guido', 4127), ('jack', 4098)])
>>> {x: x**2 for x in (2, 4, 6)}
>>> dict(sape=4139, guido=4127, jack=4098)
```