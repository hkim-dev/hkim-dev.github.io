---
title: "Understanding Python's 'ImportError: Attempted Relative Import With No Known Parent Package' Error"
date: 2024-02-27 15:00:00 -0400
categories: "programming"
header:
  teaser: "assets/images/vscode-relative-import.png"
tags:
  - Python
# classes: wide
toc: true
---


![vscode-captured](/assets/images/vscode-relative-import.png)

**TL;DR: Use absolute imports or run your script as a module using `python -m`!**
<br>

# Background
Creating a local Python package was a blast until I ran into this error: **ImportError: Attempted Relative Import With No Known Parent Package**. Trying to make sense of articles and StackOverflow posts about it felt like decoding hieroglyphs. In this article, I'll break down what causes this error and how to fix it, using a simple example that anyone can follow.

## Script, Module, and Package
1. Module: A file containing Python code meant to be imported into scripts or other modules.
2. Package: A directory containing Python modules.
3. Script: A Python program meant to be directly executed.

> **_NOTE_:** `__init__.py` used to be required to be part of a package but not anymore from Python 3.3 onwards. See [PEP 420](https://peps.python.org/pep-0420/) for more information.

## Relative Imports in Python

To know what the error means, we need to know what *relative imports* are in Python. Having been introduced in [PEP 328](https://peps.python.org/pep-0328/), they are a way to import modules that are located relative to the current module's position. Relative imports are primarily intended for use within a package and subpackage hierarchy. They simplify import statements like the following:

- `from ..module import object`

The two leading dots mean that the module to be imported is located in a parent directory. One dot (e.g., `.module`) would mean that it is located in the same directory.

# Example
Now let's consider a simple package structure:

```md
package/
│
├── __init__.py
├── module.py
└── tests/
    ├── __init__.py
    └── test_module.py
```

Let's say `module.py` contains some functions that you want to test, and you've created a test module `test_module.py` within the `tests` directory.

module.py has the following code:
```python
# module.py

def add(a, b):
    return a + b
```

test_module.py contains a test written for the function in module.py:
```python
# test_module.py

from ..module import add

def test_add():
    print(add(2, 3))
```

What will happen when you try to directly run the test script (i.e., `python subpackage/module2.py`)? You'll get the dreaded "ImportError: Attempted Relative Import With No Known Parent Package" error. Why? Remember that relative imports are resolved based on the [`__package__`](https://peps.python.org/pep-0366/#proposed-change) attribute. The attribute provides **information about the package to which a module belongs.** `__package__` is set to `None` when a .py file is run as a script. With `__package__` being None, Python doesn't recognize that `module2.py` is part of a package and rather treats it as a standalone script. Hence the *no known parent package* error!

> **_NOTE 1:_** `sys.path` contains the module search path initialized when Python starts. Please read more about it in this link: https://docs.python.org/3/library/sys_path_init.html

> **_NOTE 2:_** You can add search paths to [`sys.path`](https://docs.python.org/3/library/sys.html#sys.path). Another alternative is modifying the [PYTHONPATH](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONPATH) environment variable. Read more about them in the linked documentations. While modifying those can be a workaround for the introduced error, I personally don't find it advisable as it affects the entire Python runtime environment. My two cents here is that manual modifications to either `sys.path` or `PYTHONPATH` make it difficult to manage the code.


# Solutions

## Solution 1: Absolute Import

The first solution I suggest is to use absolute imports in the test scripts and run them outside of the package directory.

Modify `test_module.py` like the following:
```python
# test_module.py

from package.module import add

def test_add():
    print(add(2, 3))
```

Note the import statement does not contain leading dots. And let's add a new script in the parent directory of the package.

```md
project/
├── package/
│   ├── __init__.py
│   ├── module.py
│   └── tests/
│       ├── __init__.py
│       └── test_module.py
└── script.py
```

Run the test module in `script.py`:
```python
# script.py

import sys
from package.tests.test_module import test_add

print(sys.path)
print(__name__)
print(__package__)

def main():
    test_add()

if __name__ == "__main__":
    main()
```

```console
hyeminkim@Hyemins-MacBook-Air python_relative_import % python script.py
['/Users/hyeminkim/python_relative_import', '/Users/hyeminkim/anaconda3/lib/python311.zip', '/Users/hyeminkim/anaconda3/lib/python3.11', '/Users/hyeminkim/anaconda3/lib/python3.11/lib-dynload', '/Users/hyeminkim/anaconda3/lib/python3.11/site-packages', '/Users/hyeminkim/anaconda3/lib/python3.11/site-packages/aeosa']
__main__
None
5
```

Note that `__package__` is set to None. Python will correctly locate the package in the current working directory specified in `sys.path`.

## Solution 2: Run it as a Module

To directly run `test_module` using relative imports, run it as a module using the `-m` option without the file extension '.py'.
Let's change `test_module.py` back to the old script with some new prints.

```python
# test_module.py
import sys
from ..module import add

print(sys.path)
print(__name__)
print(__package__)

def test_add():
    print(add(2, 3))
```

```console
hyeminkim@Hyemins-MacBook-Air python_relative_import % python -m package.tests.test_module
['/Users/hyeminkim/python_relative_import', '/Users/hyeminkim/anaconda3/lib/python311.zip', '/Users/hyeminkim/anaconda3/lib/python3.11', '/Users/hyeminkim/anaconda3/lib/python3.11/lib-dynload', '/Users/hyeminkim/anaconda3/lib/python3.11/site-packages', '/Users/hyeminkim/anaconda3/lib/python3.11/site-packages/aeosa']
__main__
package.tests
```

The `__package__` attribute is set to `package.tests`, providing Python with the package context necessary for the relative import of `module`.


# Wrapping Up

You've reached the end of this article! Good job! :confetti_ball: As much as importing in Python can be confusing, I believe understanding how it works gives you a clearer view of Python's module/package system. I also recommend exploring the docs linked in this article for deeper understanding. Happy coding! :raised_hands: