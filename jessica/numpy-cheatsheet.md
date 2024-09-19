# NumPy Cheat Sheet

## Table of Contents
1. [Array Creation](#array-creation)
2. [Array Indexing](#array-indexing)
3. [Array Slicing](#array-slicing)
4. [Array Operations](#array-operations)
5. [Array Functions](#array-functions)
6. [Linear Algebra](#linear-algebra)
7. [Random Numbers](#random-numbers)
8. [Reshaping and Transposing](#reshaping-and-transposing)
9. [Stacking and Splitting](#stacking-and-splitting)
10. [Input and Output](#input-and-output)

## Array Creation
```python
import numpy as np

a = np.array([1, 2, 3])
b = np.zeros((3, 3))
c = np.ones((2, 2))
d = np.eye(3)
e = np.arange(10)
f = np.linspace(0, 1, 5)
```

## Array Indexing
```python
a = np.array([[1, 2, 3], [4, 5, 6]])
print(a[0, 1])  # 2
print(a[1, -1])  # 6
```

## Array Slicing
```python
a = np.array([[1, 2, 3, 4], [5, 6, 7, 8]])
print(a[:, 1:3])  # [[2 3], [6 7]]
```

## Array Operations
```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

print(a + b)  # [5 7 9]
print(a * b)  # [4 10 18]
print(a ** 2)  # [1 4 9]
```

## Array Functions
```python
a = np.array([[1, 2, 3], [4, 5, 6]])

print(np.sum(a))  # 21
print(np.mean(a))  # 3.5
print(np.max(a))  # 6
print(np.min(a))  # 1
print(np.std(a))  # 1.707825127659933
```

## Linear Algebra
```python
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

print(np.dot(a, b))  # [[19 22], [43 50]]
print(np.linalg.inv(a))  # [[-2.   1. ], [ 1.5 -0.5]]
print(np.linalg.det(a))  # -2.0
```

## Random Numbers
```python
print(np.random.rand(3, 3))  # Random 3x3 array from uniform distribution
print(np.random.randn(3, 3))  # Random 3x3 array from normal distribution
print(np.random.randint(1, 10, (3, 3)))  # Random 3x3 integer array from 1 to 9
```

## Reshaping and Transposing
```python
a = np.arange(12)
print(a.reshape(3, 4))
print(a.reshape(3, -1))  # -1 means "whatever is needed"

b = np.array([[1, 2, 3], [4, 5, 6]])
print(b.T)  # Transpose
```

## Stacking and Splitting
```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

print(np.vstack((a, b)))  # Vertical stack
print(np.hstack((a, b)))  # Horizontal stack

c = np.array([[1, 2, 3, 4], [5, 6, 7, 8]])
print(np.hsplit(c, 2))  # Horizontal split
print(np.vsplit(c, 2))  # Vertical split
```

## Input and Output
```python
np.save('array.npy', a)  # Save array to file
loaded_array = np.load('array.npy')  # Load array from file

np.savetxt('array.txt', a)  # Save array to text file
loaded_array = np.loadtxt('array.txt')  # Load array from text file
```

For more detailed information, refer to the [official NumPy documentation](https://numpy.org/doc/stable/).
