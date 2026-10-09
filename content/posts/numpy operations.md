---
title: "Essential NumPy Functions for Deep Learning & Machine Learning"
date: 2026-10-09
tags: ["NumPy", "Deep Learning", "Python", "Machine Learning"]
description: "A practical cheat sheet of essential NumPy functions used in vectorization, KNN, and Matrix Factorization."
---

When transitioning from basic Python loops to high-performance deep learning code, mastering NumPy's vectorized operations is crucial. Below is a curated list of essential NumPy functions, categorized by their use cases in machine learning algorithms like K-Nearest Neighbors (KNN), Cross-Validation, and Matrix Factorization.

## 1. Array Initialization & Creation

### `np.zeros()` / `np.zeros_like()`
Creates an array filled with zeros. `zeros_like` creates an array of zeros with the same shape and type as a given array.
```python
import numpy as np

# Create a 2x3 matrix of zeros
dists = np.zeros((2, 3))

# Create a zero matrix matching the shape of an existing matrix
matrix_input = np.array([[1, 2], [3, 4]])
e = np.zeros_like(matrix_input)
```

### `np.full()`
Returns a new array of given shape and type, filled with a specific value (e.g., np.nan).

```python

# Create a 3x3 matrix filled with NaN
R = np.full((3, 3), np.nan)

```

## 2. Mathematical & Reduction Operations
### `np.sum()` (and the `keepdims` trick)
Sums array elements over a given axis. Using keepdims=True is critical for maintaining matrix dimensions during broadcasting.

```python
X = np.array([[1, 2], [3, 4]])

# Sum along columns (axis=1), keeping the 2D shape: (2, 1)
sum_X = np.sum(np.square(X), axis=1, keepdims=True)
```

### `np.square()` & `np.sqrt()`
Element-wise square and square root operations.

```python
diff = np.array([-3, 4])
l2_dist = np.sqrt(np.sum(np.square(diff))) # Computes L2 norm: 5.0
```

### `np.mean()`
Computes the arithmetic mean. It can also be cleverly used on boolean arrays to calculate accuracy.

```Python
y_pred = np.array([1, 0, 1, 1])
y_true = np.array([1, 0, 0, 1])

# Calculate classification accuracy directly
accuracy = np.mean(y_pred == y_true) # Output: 0.75 (75%)
```

## 3. Matrix & Linear Algebra
### `np.dot()` & `np.matmul()`
Used for dot products and matrix multiplication. `np.matmul()` is preferred for matrix multiplications (and is strictly 2D or higher), while `np.dot()` is flexible for 1D dot products.

```Python
X = np.random.rand(5, 10)      # 5 test samples, 10 features
X_train = np.random.rand(20, 10) # 20 train samples, 10 features

# Matrix multiplication for L2 distance expansion: -2 * X * Y^T
dot_matrix = -2 * np.dot(X, X_train.T)

# Alternatively, using matmul
P = np.random.rand(10, 5)
Q = np.random.rand(20, 5)
R_pred = np.matmul(P, Q.T) # Shape: (10, 20)
```

## 4. Array Manipulation & Broadcasting
### `np.array_split()`
Splits an array into multiple sub-arrays (useful for K-fold cross-validation).

```Python
X_train = np.arange(10)
# Split into 5 folds
X_train_folds = np.array_split(X_train, 5) 
```

### `np.concatenate()`
Joins a sequence of arrays along an existing axis. Perfect for merging training folds.

```Python
# Merge all folds except the i-th one for cross-validation
i = 1
X_train_cv = np.concatenate(X_train_folds[:i] + X_train_folds[i+1:])
```

### `np.newaxis (Broadcasting)`
Increases the dimension of the existing array by one more dimension.

```Python
user_bias = np.array([1.0, 2.0, 3.0]) # Shape: (3,)

# Convert to column vector (3, 1) for broadcasting
user_bias_col = user_bias[:, np.newaxis] 
```

## 5. Searching, Sorting & Counting
### `np.argsort()`
Returns the indices that would sort an array. Essential for finding the K-Nearest Neighbors.

```Python
distances = np.array([0.5, 0.1, 0.9, 0.2])

# Get indices of the 2 smallest distances
nearest_idx = np.argsort(distances)[:2] # Output: [1, 3]
```

### `np.bincount()` & `np.argmax()`
`np.bincount` counts the number of occurrences of each value in an array of non-negative ints. `np.argmax` returns the index of the maximum value. Combined, they find the mode (majority vote).

```Python
# K nearest labels
closest_y = np.array([2, 0, 2, 1, 2])

# Find the majority vote (the most frequent label)
y_pred = np.bincount(closest_y).argmax() # Output: 2
```

### `np.where()`
Returns elements or indices chosen from x or y depending on a condition.

```Python
movies = np.array([101, 102, 103])
# Find the exact index of movie ID 102
idx = np.where(movies == 102)[0][0] # Output: 1
```

## 6. Handling Missing Data (NaNs)
### `np.isnan()` & `~` (Bitwise NOT)
Creates a boolean mask indicating NaN values. The ~ operator inverts it to isolate valid data.

```Python
arr = np.array([[1.0, np.nan], [3.0, 4.0]])
mask = ~np.isnan(arr) 

# Extract only valid values into a 1D array
valid_data = arr[mask] # Output: [1.0, 3.0, 4.0]
```

### `np.count_nonzero()`
Counts the number of True values in an array. Faster than np.sum() for boolean masks.

```Python
num_valid_ratings = np.count_nonzero(~np.isnan(arr)) # Output: 3
```