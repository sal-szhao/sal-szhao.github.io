---
title: Python Official Tutorial 1
date: 2023-06-07 11:24:38
tags: [python, tech]
---

In this blog, I will follow the official Python [tutorial](https://docs.python.org/3/tutorial/). Being a coder using Python for several years, I'd like to see if there is anything omitting which I could make up for.

## Some flags

``` bash
$ python -c <command>
$ python -c "print(3)"
```

`-c` flag will allow running python codes directly in the terminal.

``` bash
$ python -m <module-name>
$ python -m timeit '3 + 2'
```

`-m` flag will allow using modules directly in the terminal.

## Source Code Encoding

### Encoding

```
# -*- coding: uft-8 -*-
```

to declare the encoding of the file.

### Executable script

``` 
#!/usr/bin/env python3
```

Add the line to the beginning of `.py` file (should be the first line). Then run

``` bash
$ chmod +x python-interpreter.py
$ ./python-interpreter.py
```

`chmod +x` will turn `.py` file into an excetuable file. Use `./` to run the executable file.

## Python I/O 

### Argument passing

``` python 
import sys

print('sys.argv: ', sys.argv)
print('sys.argv[0]: ', sys.argv[0])
print('sys.argv[1]: ', sys.argv[1])
```

`sys.argv` will record all the inputs from the command line.

``` bash
$ python python-interpreter.py 1 2
```

Add the inputs behind the file name.

Notice that `sys.argv[0]` will always be the file name. The output will be 

```
sys.argv:  ['python-interpreter.py', '1', '2']
sys.argv[0]:  python-interpreter.py
sys.argv[1]:  1
```

### Input from command line

``` python
name = input('Please enter your name: ')
print(name)
```

The built-in function `input()` will allow users to input a string and store in the variable.

### Reading and writing to files

The first method is to use `open` keyword. Notice that file needs to be closed by calling `close()`, otherwise texts may not be written into the disk properly.

``` python
f = open('workfile', 'w', encoding="utf-8")
f.close()
```

The second method is to use `with` keyword, which is a better practice becuase it will automatically close the file after executed.

``` python
with open('workfile', encoding="utf-8") as f:
    read_data = f.read()
```

There are also different ways of reading the content in the file object.

``` python
# Read the whole contents in the file.
f.read()
# Read a single line.
f.readline()
# Iterate through the file.
for line in f:
    print(line, end='')
```

### JSON file I/O

One can transform an object to `json` by `json.dumps`.

``` python
import json

x = [1, 'simple', 'list']
print(json.dumps(x))
# '[1, "simple", "list"]'
```

Or dump/load the json into/from a specific file by `json.dump()` and `json.load()`.

``` python
json.dump(x, f)
x = json.load(f)
```


## Zip function

`zip` can be used to aggregate iterables.

``` python
names = ['Alice', 'Bob', 'Charlie']
ages = [24, 50, 18]
print(list(zip(names, ages)))
# [('Alice', 24), ('Bob', 50), ('Charlie', 18)]
```

## Looping techniques

Achieve reverse order of `range`.

``` python
print(list(range(9, 0, -1)))
# [9, 8, 7, 6, 5, 4, 3, 2, 1]
print(list(reversed(range(1, 10, 1))))
# [9, 8, 7, 6, 5, 4, 3, 2, 1]
```

## Comparisons

Comparisons can be chained. Eg. `a < b == c` will check `a < b` and `b == c`.

``` python
print(1 < 2 == 2)
# True
```

## Exceptions Handling

### Exceptions chaining
``` python
try:
    open("database.sqlite")
except OSError:
    raise RuntimeError("unable to handle error")
```

will give the result

```
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
FileNotFoundError: [Errno 2] No such file or directory: 'database.sqlite'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
RuntimeError: unable to handle error
```

### Clean-up actions

`finally` will be executed even when exceptions are raised.

``` python 
try:
    raise KeyboardInterrupt
finally:
    print('Goodbye, world!')
```
