---
title: "Using Python's vars() to Generate a Dictionary from Object Fields"
date: 2022-12-13 15:00:00 -0400
categories: "programming"
header:
  teaser: "assets/images/vscode-relative-import.png"
tags:
  - Python
toc: true
toc_sticky: true
---

# vars() function

Let’s consider the following example:

```python
class Foo:
	def __init__(self):
		self.a = 1
		self.b = 2
```

The vars() function in Python returns the **`__dict__`** attribute of an object, which is a dictionary containing the object's attributes and their values. In the example code provided, vars(foo) returns {'a': 1, 'b': 2} because foo is an instance of the Foo class, which has attributes a and b.

```python
foo = Foo()
print(vars(foo))  # Output: {'a': 1, 'b': 2}
```

This demonstrates how you can use the vars() function to inspect the attributes of an object dynamically. It's particularly useful when you need to introspect objects or manipulate their attributes programmatically.

# Modifying Object Attributes

You can also use the `vars()` function to modify object attributes dynamically. Let's say we want to update the value of attribute **`b`** in the **`foo`** object:

```python
vars(foo)['b'] = 42
print(vars(foo))  # Output: {'a': 1, 'b': 42}
```

# Adding New Attributes

Additionally, you can add new attributes to an object using the **`vars()`** function. Let's add a new attribute **`c`** to the **`foo`** object:

```python
vars(foo)['c'] = 3.14
print(vars(foo))  # Output: {'a': 1, 'b': 42, 'c': 3.14}
```