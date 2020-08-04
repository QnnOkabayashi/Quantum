# Quantum Matrix Library
#### By Quinn Okabayashi

Quantum is a library built for optimized matrix operations in Python. For compilation details, see bottom.

# Docs

## Constructor
The constructor is implemented in the following way:
```python
Matrix(data, dims: (int, int)) -> Matrix
```
The `data` argument can be one of the following types:
1. `number`
2. `List[number]`
3. `List[List[number]]`
4. `Matrix`

The `dims` argument is optional.

### Constructing a matrix from `number`

Constructing a matrix from only a constant defaults to a 1x1 matrix.
```python
>>> Matrix(0)
[   0.000 ]
```
You can specify the shape with the `dims` argument.
```python
>>> Matrix(0, dims=(3,4))
[   0.000   0.000   0.000   0.000 ]
[   0.000   0.000   0.000   0.000 ]
[   0.000   0.000   0.000   0.000 ]
```
Leaving the second dim empty will default to 1 column.
```python
>>> Matrix(0, dims=(3,))
[   0.000 ]
[   0.000 ]
[   0.000 ]
```
### Constructing a matrix from `List[number]`

Constructing a matrix from a list without providing `dims` returns a column vector representation.
```python
>>> Matrix([1,2,3,4])
[   1.000 ]
[   2.000 ]
[   3.000 ]
[   4.000 ]
```
Providing `dims` will return a matrix of that shape, storing list values in row-major order.

```python
>>> Matrix([1,2,3,4], dims=(2,2))
[   1.000   2.000 ]
[   3.000   4.000 ]
```
If the list length doesn't match the product of `dims`, a `ValueError` exception is raised.
```python
>>> Matrix([1,2,3], dims=(2,2))
```
```
ValueError: matrix with dims=(2, 2) cannot be created from 3 value(s)
```
### Constructing a matrix from `List[List[number]]`

Constructing a matrix from a 2D list is the easiest method for unit testing, since `dims` is optional.
```python
>>> Matrix([
...     [1,0],
...     [0,1]
... ])
[   1.000   0.000 ]
[   0.000   1.000 ]
```
If the 2D list isn't rectangular, a `ValueError` exception is raised.
- Note that the row index in the error message is base 0
```python
>>> Matrix([
...     [1,2,3],
...     [1,2]
... ])
```
```
ValueError: expected 3 element(s) in row 1, but found 2 element(s)
```
You may also specify `dims`, which is useful for asserting the shape of the matrix when the list contents are not immediately visible.
```python
>>> data = [[1,0],[0,1]]
.
.
.
>>> Matrix(data, dims=(2,2))
[   1.000   0.000 ]
[   0.000   1.000 ]
```
If `dims` is provided but doesn't match the shape of the 2D list, a `ValueError` exception is raised.
```python
>>> data = [[1,0],[0,1]]
.
.
.
>>> Matrix(data, dims=(3,2))
```
```
ValueError: dims of given 2D array (2, 2) don't match dims specified (3, 2)
```

### Constructing a matrix from `Matrix`

Constructing a Matrix from another Matrix will simply copy the contents. 
The `dims` argument is ignored in this case.
```python
>>> A = Matrix(5, dims=(2,3))
>>> Matrix(A)
[   5.000   5.000   5.000 ]
[   5.000   5.000   5.000 ]
```

## Constructing from random distributions

Quantum also provides methods to construct matrices with values from random distributions.

- `dims` and `seed` are optional arguments.
- Dimensions not provided by `dims` default to `1`.
- If `seed` is not provided, a time-based seed is used.

The constructor for a matrix from a Gaussian/normal distribution is implemented as follows:
```python
Matrix.gauss(
        mu: float, 
        sigma: float, 
        dims: (int, int), 
        seed: int
    ) -> Matrix
```
Arguments `mu` and `sigma` default to `0` and `1`, respectively.

The constructor for a matrix from a uniform distribution is implemented as follows:
```python
Matrix.uniform(
        lower: float, 
        upper: float, 
        dims: (int, int), 
        seed: int
    ) -> Matrix
```
Arguments `lower` and `upper` default to `0` and `1` respectively.

If `lower` > `upper`, a `ValueError` exception is raised.
```python
>>> Matrix.uniform(lower=1, upper=0)
```
```
ValueError: lower bound cannot be greater than upper bound: (1.000, 0.000)
```



## Matrix fields

Every matrix instance provides the following read-only fields:

- `rows`: The number of rows in the matrix
- `cols`: The number of columns in the matrix
- `data`: An array of all the matrix elements, in row-major order

## Matrix methods

- `T` : Returns the transpose of the matrix.
- > `T` uses a lazy copying method, so the returned matrix shares internal memory with the original until one of them is modified. This also means that getting the transpose of a matrix takes O(1) time. Each matrix also caches its transpose, so calls like `A.T.T.T.T` are inexpensive, and only create 1 new object.
- `map(closure)`: Applies the `closure` function to each element in the matrix, modifying it inplace.
- > `map(closer)` will unbind any existing transpose objects. If the transpose was accessed, this ___will___ make a copy of the internal memory, which may be expensive.

## Number methods
- `+` : Returns the sum of two matrices
- `-` : Returns the difference of two matrices
- `*` : Returns the element-wise product of two matrices. Can also use a scalar.
- `/` : Returns the element-wise quotient of two matrices. Can also use a scalar as the divisor.
- `@` : Returns the matrix product of two matrices.
- `==` : Returns `True` if the matrices are equivalent, else `False`.
- `!=` : Returns `True` if the matrices are not equivalent, else `False`.


# Compilation details

To build the module on your machine, your computer must have Python developer tools installed (see below).

`pypath.h` allows the module to include a general filepath, so the user doesn't have to modify internal code.

## 1. Installing Python developer tools

### **Linux**
You should be able to install the dev tools with the following command

```
sudo apt-get install python3.8-dev
```

### **OSX**
Installing Python through Homebrew automatically installs the dev tools

Homebrew installation instructions can be found here: [https://docs.python-guide.org/starting/install3/osx/](https://docs.python-guide.org/starting/install3/osx/)

### **Windows**

Use Google.

## 2. Configuring pypath.h

In `include/`, create a file named `pypath.h`

**(Linux/OSX):** It may be useful to run the locate command in the terminal to find your `Python.h`

```
locate Python.h
```

Paste the following contents into the file, substituting your include path:

```c
#ifndef PYPATH_H
#define PYPATH_H

#include <path/to/your/Python.h>

#endif /* PYPATH_H */
```
