# Quantum
### By Quinn Okabayashi
Quantum is a Python extension written in C to efficiently compute matrix operations. 

This library must be built from source, but future plans involve providing prebuilt binaries through pip.

See bottom for build details.

# Docs
## Constructor
The Matrix constructor is implemented as follows:
```python
Matrix(data, dims: (int, int)) -> Matrix
```
The `data` argument can be one of the following types:
1. `number`
2. `List[number]`
3. `List[List[number]]`
4. `Matrix`

Where `number` is shorthand for `Union[int, float]`.

The `dims` argument is optional.

___
<details>
<summary>Constructing a Matrix from <code>number</code></summary>

Constructing a matrix from a constant defaults to a 1x1 matrix.
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
</details>

___
<details>
<summary>Constructing a Matrix from <code>List[number]</code></summary>

Constructing a Matrix from a list without providing `dims` returns the column vector representation.
```python
>>> Matrix([1,2,3,4])
[   1.000 ]
[   2.000 ]
[   3.000 ]
[   4.000 ]
```
Providing `dims` will return a Matrix of that shape, storing list values in row-major order.

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
</details>

___
<details>
<summary>Constructing a Matrix from <code>List[List[number]]</code></summary>

Constructing a Matrix from a 2D list is the easiest method for unit testing, since `dims` is optional.
```python
>>> Matrix([
...     [1,0],
...     [0,1]
... ])
[   1.000   0.000 ]
[   0.000   1.000 ]
```
If the 2D list isn't rectangular, a `ValueError` exception is raised.
- Note that the error message uses zero based indexing to indicate the row.
```python
>>> Matrix([
...     [1,2,3],
...     [1,2]  # should have 3 elements
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

</details>

___
<details>
<summary>Constructing a Matrix from <code>Matrix</code></summary>

Constructing a Matrix from another matrix will simply copy the contents. 
The `dims` argument is ignored in this case.
```python
>>> A = Matrix(5, dims=(2,3))
>>> Matrix(A)
[   5.000   5.000   5.000 ]
[   5.000   5.000   5.000 ]
```
</details>

___
## Constructing from random distributions

Quantum also provides methods to construct matrices with values from random distributions.

- `dims` and `seed` are optional arguments.
- Dimensions not provided by `dims` default to `1`.
- If `seed` is not provided, a time-based seed is used.

___
<details>
<summary>Constructing a matrix from a Gaussian distribution</summary>

The constructor for a matrix from a Gaussian distribution is implemented as follows:
```python
Matrix.gauss(
        mu: float = 0, 
        sigma: float = 1, 
        dims: (int, int), 
        seed: int
    ) -> Matrix
```

Example:
```python
>>> Matrix.gauss(sigma=0.2, dims=(2,3), seed=1)
[   0.108  -0.169   0.074 ]
[  -0.350   0.113  -0.070 ]
```
</details>

___
<details>
<summary>Constructing a matrix from a uniform distribution</summary>

The constructor for a matrix from a uniform distribution is implemented as follows:
```python
Matrix.uniform(
        lower: float = 0, 
        upper: float = 1, 
        dims: (int, int), 
        seed: int
    ) -> Matrix
```

If `lower` > `upper`, a `ValueError` exception is raised.
```python
>>> Matrix.uniform(lower=1, upper=0)
```
```
ValueError: lower bound cannot be greater than upper bound: (1.000, 0.000)
```

Example:
```python
>>> Matrix.uniform(lower=-1, upper=1, dims=(2,3), seed=1)
[   0.680  -0.211   0.566 ]
[   0.597   0.823  -0.605 ]
```
</details>

___
## Matrix fields

Every Matrix object provides the following read-only fields:

- `rows`: The integer number of rows in the matrix
- `cols`: The number of columns in the matrix
- `data`: A list of all the matrix elements, in row-major order

___
## Matrix methods

<details>
<summary><code>T</code> : Returns the transpose of the matrix.</summary>

* `T` uses a lazy copying method, so the returned matrix shares internal memory with the original until one of them is modified. This also means that getting the transpose of a matrix takes O(1) time. Each matrix also caches its transpose, so calls like `A.T.T.T.T` are inexpensive, and only create 1 new Matrix object.

</details>

<details>
<summary><code>map(closure: Function[[number], number])</code> Applies the <code>closure</code> function to each element in the matrix, modifying it inplace.</summary>

- If the matrix has a cached transpose `Matrix` object, `map()` will remove from its cache and also copy the data to a new array.

</details>    

___
## Number methods
- `+` : Returns the sum of two matrices
- `-` : Returns the difference of two matrices
- `*` : Returns the element-wise product of two matrices
    - A scalar can also be used as one argument for scalar multiplication
- `/` : Returns the element-wise quotient of two matrices
    - A scalar can also be used as the divisor for scalar division
- `@` : Returns the matrix product of two matrices.
- `==` : Returns `True` if the matrices are equivalent, otherwise `False`.
- `!=` : Returns `True` if the matrices are not equivalent, otherwise `False`.

___
# Build instructions

To compile the module on your machine, you need to configure a `pypath.h` file so the module can access the Python internals provided by the Python Developer tools.

If you already have the Python developer tools installed on your machine, skip to step 2.

## 1. Installing Python developer tools

### **Linux**
You should be able to install the dev tools with the following command:

```
$ sudo apt-get install python3.8-dev
```

### **OSX**
Installing Python through Homebrew automatically installs the dev tools

Homebrew installation instructions can be found here: [https://docs.python-guide.org/starting/install3/osx/](https://docs.python-guide.org/starting/install3/osx/)

### **Windows**

Use Google.

## 2. Compiling the module

1. Navigate to the root directory of this repository

2. Create the `pypath.h` file inside of `include/`
```
$ touch include/pypath.h
```

3. Locate the path to your `Python.h`. 

    - If you're using UNIX based system, the following command may be useful:
```
$ locate Python.h
```

3. Paste the following contents into the file, substituting your include path:
```c
#ifndef PYPATH_H
#define PYPATH_H

#include <path/to/your/Python.h>

#endif /* PYPATH_H */
```

4. In your terminal, navigate to the root of this repository and enter the following command:
```
$ python3 setup.py build
```
If you followed the directions correctly, you should see a Quantum.(device_info).so file show up in the root directory of the repository. Move this file into the same directory as your Python scripts to be able to import it directly.

