# Python Cheat Sheet

## Table of Contents
1. [Basics](#basics)
2. [Data Types](#data-types)
3. [Control Flow](#control-flow)
4. [Functions](#functions)
5. [Object-Oriented Programming](#object-oriented-programming)
6. [Modules and Packages](#modules-and-packages)
7. [File Handling](#file-handling)
8. [Exception Handling](#exception-handling)
9. [Comprehensions](#comprehensions)
10. [Lambda Functions](#lambda-functions)
11. [Decorators](#decorators)
12. [Generators](#generators)

## Basics
### Comments
```python
# Single line comment
"""
Multi-line comment
or docstring
"""
```

### Variables and Assignment
```python
x = 5
y, z = 3, 4
```

### Print Function
```python
print("Hello, World!")
print(f"x is {x}")
```

## Data Types
### Numbers
```python
integer = 42
float_num = 3.14
complex_num = 1 + 2j
```

### Strings
```python
string = "Hello, World!"
multiline = """
Multi-line
string
"""
```

### Lists
```python
my_list = [1, 2, 3]
my_list.append(4)
```

### Tuples
```python
my_tuple = (1, 2, 3)
```

### Sets
```python
my_set = {1, 2, 3}
my_set.add(4)
```

### Dictionaries
```python
my_dict = {"key": "value"}
my_dict["new_key"] = "new_value"
```

## Control Flow
### If-Else Statement
```python
if condition:
    # code
elif another_condition:
    # code
else:
    # code
```

### For Loop
```python
for item in iterable:
    # code
```

### While Loop
```python
while condition:
    # code
```

## Functions
```python
def function_name(parameter1, parameter2):
    # function body
    return result
```

## Object-Oriented Programming
```python
class ClassName:
    def __init__(self, attribute):
        self.attribute = attribute
    
    def method(self):
        # method body
```

## Modules and Packages
```python
import module_name
from module_name import function_name
from package_name import module_name
```

## File Handling
```python
with open("file.txt", "r") as file:
    content = file.read()

with open("file.txt", "w") as file:
    file.write("Content")
```

## Exception Handling
```python
try:
    # code that may raise an exception
except ExceptionType:
    # handle the exception
finally:
    # code that always executes
```

## Comprehensions
### List Comprehension
```python
squares = [x**2 for x in range(10)]
```

### Dictionary Comprehension
```python
square_dict = {x: x**2 for x in range(5)}
```

## Lambda Functions
```python
square = lambda x: x**2
```

## Decorators
```python
def decorator(func):
    def wrapper(*args, **kwargs):
        # do something before
        result = func(*args, **kwargs)
        # do something after
        return result
    return wrapper

@decorator
def my_function():
    pass
```

## Generators
```python
def my_generator():
    yield 1
    yield 2
    yield 3
```

For more detailed information, refer to the [official Python documentation](https://docs.python.org/3/).
