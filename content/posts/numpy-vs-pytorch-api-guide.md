---
title: "NumPy vs. PyTorch: A Practical API Comparison"
date: 2026-10-10
slug: "numpy-vs-pytorch-api-guide"
description: "A practical guide to common NumPy and PyTorch APIs, covering broadcasting, indexing, reshaping, statistics, memory sharing, and automatic differentiation."
tags: ["Python", "NumPy", "PyTorch"]
draft: false
---

NumPy and PyTorch have remarkably similar APIs for working with multidimensional data. If you know how to create, index, reshape, and calculate with a NumPy array, much of that knowledge transfers directly to a PyTorch tensor.

However, migrating code takes more than replacing `np` with `torch`. Similar names can hide differences in default data types, return values, memory sharing, and statistical conventions. PyTorch also adds device management and automatic differentiation, which affect how you should write and modify tensor operations.

This guide compares common operations on ordinary NumPy arrays and dense PyTorch tensors. It focuses on the array operations shared by both libraries; PyTorch's neural network and training APIs are outside its scope. Examples assume a modern PyTorch version that supports the `correction` argument for variance and standard deviation, and that its default floating-point dtype has not been changed.

```python
import numpy as np
import torch
```

Throughout the tables, `a` and `b` refer to NumPy arrays, while `t` and `u` refer to PyTorch tensors. Each example should be read independently.

## 1. Core objects and basic properties

NumPy's core object is `ndarray`. PyTorch's core object is `Tensor`.

```python
a = np.array([[1, 2, 3], [4, 5, 6]])
t = torch.tensor([[1, 2, 3], [4, 5, 6]])
```

| Purpose | NumPy | PyTorch |
|---|---|---|
| Core type | `np.ndarray` | `torch.Tensor` |
| Shape | `a.shape` | `t.shape` or `t.size()` |
| Number of dimensions | `a.ndim` | `t.ndim` or `t.dim()` |
| Total number of elements | `a.size` | `t.numel()` |
| Length of a particular dimension | `a.shape[0]` | `t.shape[0]` or `t.size(0)` |
| Data type | `a.dtype` | `t.dtype` |
| Extract a Python scalar from a one-element object | `a.item()` | `t.item()` |
| Convert to a Python list | `a.tolist()` | `t.tolist()` |
| Device | Ordinary arrays reside on the CPU | `t.device` |
| Gradient tracking | No built-in equivalent | `t.requires_grad` |

One naming trap is `size`:

```python
a.size       # 6: total element count

t.size()     # torch.Size([2, 3]): shape
t.numel()    # 6: total element count
```

PyTorch tensors can run on supported accelerators, such as CUDA GPUs, and participate in a computation graph for automatic differentiation. Ordinary NumPy arrays provide neither capability directly.

## 2. Creating arrays and tensors

| Purpose | NumPy | PyTorch |
|---|---|---|
| Create from a list | `np.array([1, 2, 3])` | `torch.tensor([1, 2, 3])` |
| Reuse existing data when possible | `np.asarray(data)` | `torch.as_tensor(data)` |
| All zeros | `np.zeros((2, 3))` | `torch.zeros((2, 3))` |
| All ones | `np.ones((2, 3))` | `torch.ones((2, 3))` |
| Fill with a value | `np.full((2, 3), 7)` | `torch.full((2, 3), 7)` |
| Uninitialized storage | `np.empty((2, 3))` | `torch.empty((2, 3))` |
| Identity matrix | `np.eye(3)` | `torch.eye(3)` |
| Sequence with a fixed step | `np.arange(0, 10, 2)` | `torch.arange(0, 10, 2)` |
| A fixed number of evenly spaced points | `np.linspace(0, 1, 5)` | `torch.linspace(0, 1, 5)` |
| Zeros with matching shape and dtype | `np.zeros_like(a)` | `torch.zeros_like(t)` |
| Ones with matching shape and dtype | `np.ones_like(a)` | `torch.ones_like(t)` |
| A value with matching shape and dtype | `np.full_like(a, 7)` | `torch.full_like(t, 7)` |

`empty` does **not** initialize every element to zero. Write to its elements before relying on their contents.

`arange` uses an exclusive stop bound. For predictable floating-point sample counts, prefer `linspace`. Both libraries include the endpoints in `linspace` by default, but NumPy also supports `endpoint=False`; PyTorch's `linspace` has no corresponding parameter.

PyTorch creation functions can also accept a device and gradient settings:

```python
t = torch.zeros(
    (2, 3),
    dtype=torch.float32,
    device="cpu",
    requires_grad=True,
)
```

## 3. Data types and conversion

| Purpose | NumPy | PyTorch |
|---|---|---|
| 32-bit floating point | `np.float32` | `torch.float32` |
| 64-bit floating point | `np.float64` | `torch.float64` |
| 32-bit integer | `np.int32` | `torch.int32` |
| 64-bit integer | `np.int64` | `torch.int64` |
| Boolean | `np.bool_` | `torch.bool` |
| Convert to float32 | `a.astype(np.float32)` | `t.to(torch.float32)` or `t.float()` |
| Convert to int64 | `a.astype(np.int64)` | `t.to(torch.int64)` or `t.long()` |

### Default floating-point types differ

```python
np.array([1.0, 2.0]).dtype
# dtype('float64')

torch.tensor([1.0, 2.0]).dtype
# torch.float32
```

Similarly, `np.zeros(...)` defaults to `float64`, while `torch.zeros(...)` uses PyTorch's default floating-point dtype, normally `float32`.

Specify the dtype explicitly when comparing results or migrating code:

```python
a = np.array([1, 2, 3], dtype=np.float32)
t = torch.tensor([1, 2, 3], dtype=torch.float32)
```

Type promotion rules are not identical across the two libraries, so matching input dtypes is particularly useful when mixing integers, floating-point values, and Python scalars.

### Integer means behave differently

NumPy automatically produces a floating-point mean from an integer array:

```python
np.array([1, 2, 3]).mean()
# 2.0
```

PyTorch's `mean` requires a floating-point or complex input, or an appropriate explicit dtype:

```python
t = torch.tensor([1, 2, 3])

# t.mean()  # Raises an error for this integer tensor.
t.float().mean()
# tensor(2.)
```

Conversion also has memory implications: `a.astype(...)` copies by default, whereas `t.to(...)` can return the original tensor when the requested dtype and device already match.

## 4. Indexing, slicing, and boolean masks

Basic indexing syntax is almost identical.

| Purpose | NumPy | PyTorch |
|---|---|---|
| First row | `a[0]` | `t[0]` |
| Second column | `a[:, 1]` | `t[:, 1]` |
| First two rows | `a[:2]` | `t[:2]` |
| Last row | `a[-1]` | `t[-1]` |
| Every other element along the first dimension | `a[::2]` | `t[::2]` |
| Selected rows | `a[[0, 2]]` | `t[[0, 2]]` |
| Boolean selection | `a[a > 0]` | `t[t > 0]` |
| Conditional assignment | `a[a < 0] = 0` | `t[t < 0] = 0` |
| Add a leading dimension | `a[None, ...]` | `t[None, ...]` |

Combine elementwise conditions with `&`, `|`, and `~`, and put each comparison in parentheses:

```python
a[(a > 0) & (a < 10)]
t[(t > 0) & (t < 10)]
```

Python's `and` and `or` do not perform elementwise boolean operations.

### Negative slice steps are a difference

NumPy supports reversing an array with a negative step:

```python
a[::-1]
```

PyTorch does not support the equivalent `t[::-1]`. Use:

```python
torch.flip(t, dims=[0])
```

Their memory behavior differs too: NumPy's `flip` returns a view, whereas `torch.flip` copies data.

Basic slicing generally returns a view in both libraries. Reading with advanced integer or boolean indexing generally returns a copy. Assigning through an index, such as `t[t < 0] = 0`, still modifies the original tensor and can be subject to autograd restrictions.

## 5. `where` and broadcasting

The three-argument forms have matching roles:

```python
np.where(condition, x, y)
torch.where(condition, x, y)
```

Each result element comes from `x` where the condition is true, and from `y` where it is false. In PyTorch, use a boolean tensor for `condition`.

**The condition, `x`, and `y` do not need identical shapes. All three must be broadcastable to one common shape.**

### A one-dimensional condition can select from a two-dimensional input

```python
condition = np.array([True, False, True])
x = np.array([[1, 2, 3], [4, 5, 6]])
y = np.array([10, 20, 30])

np.where(condition, x, y)
# array([[ 1, 20,  3],
#        [ 4, 20,  6]])
```

The PyTorch version follows the same broadcasting rules:

```python
condition = torch.tensor([True, False, True])
x = torch.tensor([[1, 2, 3], [4, 5, 6]])
y = torch.tensor([10, 20, 30])

torch.where(condition, x, y)
# tensor([[ 1, 20,  3],
#         [ 4, 20,  6]])
```

The shapes combine as follows:

```text
condition: (3,)
x:         (2, 3)
y:         (3,)
result:    (2, 3)
```

A scalar is also allowed for either branch:

```python
np.where(np.array([True, False, True]), np.array([1, 2, 3]), 0)
torch.where(torch.tensor([True, False, True]), torch.tensor([1, 2, 3]), 0)
```

### The broadcasting rule

Compare shapes from the rightmost dimension toward the left. Corresponding sizes must be equal, or one of them must be `1`. Treat missing leading dimensions as size `1`.

| Shape A | Shape B | Compatible? | Result shape |
|---|---|---|---|
| `(2, 3)` | `(3,)` | Yes | `(2, 3)` |
| `(2, 3)` | `(2, 1)` | Yes | `(2, 3)` |
| `(3, 1)` | `(1, 4)` | Yes | `(3, 4)` |
| `(2, 3)` | `(2,)` | No | Error |

For example, a condition of shape `(3,)` cannot be combined with branches of shape `(2,)`:

```python
# Both calls raise an error because the shapes are incompatible.
# np.where(np.array([True, False, True]), [1, 2], [10, 20])
# torch.where(
#     torch.tensor([True, False, True]),
#     torch.tensor([1, 2]),
#     torch.tensor([10, 20]),
# )
```

Both branches must be compatible even if every condition value selects the same branch. Also, `where` is not a lazy Python conditional: expressions passed as `x` and `y` are evaluated before the function selects elements.

### Calling `where` with only a condition

With one argument, both libraries return a tuple of index arrays or tensors, one per dimension:

```python
np.where(np.array([True, False, True]))
# (array([0, 2]),)

torch.where(torch.tensor([True, False, True]))
# (tensor([0, 2]),)
```

Related operations have a subtle default return-format difference:

| Purpose | NumPy | PyTorch |
|---|---|---|
| Conditional selection | `np.where(c, x, y)` | `torch.where(c, x, y)` |
| Nonzero indices grouped by dimension | `np.nonzero(a)` | `torch.nonzero(t, as_tuple=True)` |
| Nonzero coordinates grouped by element | `np.argwhere(a)` | `torch.nonzero(t)` |

The coordinate comparison in the last row describes ordinary inputs with at least one dimension; zero-dimensional inputs have special shape conventions.

## 6. Reshaping, adding dimensions, and transposing

| Purpose | NumPy | PyTorch |
|---|---|---|
| Change shape | `a.reshape(2, 3)` | `t.reshape(2, 3)` |
| Infer one dimension | `a.reshape(2, -1)` | `t.reshape(2, -1)` |
| Flatten | `a.flatten()` | `t.flatten()` |
| Flatten while avoiding a copy when possible | `a.ravel()` | `t.reshape(-1)` |
| Insert a dimension | `np.expand_dims(a, axis=0)` | `t.unsqueeze(0)` |
| Remove all size-1 dimensions | `a.squeeze()` | `t.squeeze()` |
| Remove a specified size-1 dimension | `a.squeeze(axis=1)` | `t.squeeze(dim=1)` |
| Swap two dimensions | `a.swapaxes(0, 1)` | `t.transpose(0, 1)` |
| Reorder all dimensions | `a.transpose(2, 0, 1)` | `t.permute(2, 0, 1)` |
| Transpose a two-dimensional matrix | `a.T` | `t.T` |
| Move a dimension | `np.moveaxis(a, 0, -1)` | `torch.movedim(t, 0, -1)` |

### `transpose` does not mean exactly the same thing

NumPy's `transpose` can specify a complete permutation. PyTorch's `transpose` swaps exactly two dimensions; use `permute` for a full permutation.

```python
a = np.zeros((2, 3, 4))
t = torch.zeros((2, 3, 4))

a.transpose(2, 0, 1).shape   # (4, 2, 3)
t.permute(2, 0, 1).shape     # torch.Size([4, 2, 3])

t.transpose(0, 2).shape      # torch.Size([4, 3, 2])
```

Use `.T` for two-dimensional matrices in portable examples. For higher-dimensional tensors, state the intended permutation explicitly.

### PyTorch's `view` and `reshape` differ

`t.view(...)` changes shape only when the existing strides allow a view. It cannot copy data to make an incompatible layout work.

`t.reshape(...)` returns a view when possible and can copy otherwise. It is often the simpler choice when you do not specifically require shared storage.

```python
t = torch.arange(6).reshape(2, 3).transpose(0, 1)

t.reshape(-1)               # Works; may copy.
# t.view(-1)                # Fails for this layout.
t.contiguous().view(-1)     # Works after obtaining a contiguous layout.
```

NumPy's `a.view()` is not the equivalent of PyTorch's shape-changing `t.view(...)`. NumPy uses it to create a view, optionally reinterpreting the dtype or array subclass.

### Flattening and squeezing have their own edge cases

- NumPy's `flatten()` always copies. PyTorch's `flatten()` may return the original tensor, a view, or a copy.
- NumPy's `ravel()` produces a contiguous flattened array. A PyTorch `reshape(-1)` can return a noncontiguous one-dimensional view when its strides allow it.
- If a specified `squeeze` dimension has a size other than `1`, NumPy raises an error; PyTorch leaves that dimension unchanged.

## 7. Elementwise operations

These APIs are among the easiest to transfer between libraries.

| Purpose | NumPy | PyTorch |
|---|---|---|
| Arithmetic | `a + b`, `a - b`, `a * b`, `a / b` | Same operators |
| Power | `a ** 2`, `np.power(a, 2)` | `t ** 2`, `torch.pow(t, 2)` |
| Absolute value | `np.abs(a)` | `torch.abs(t)` |
| Square root | `np.sqrt(a)` | `torch.sqrt(t)` |
| Exponential | `np.exp(a)` | `torch.exp(t)` |
| Natural logarithm | `np.log(a)` | `torch.log(t)` |
| Sine and cosine | `np.sin(a)`, `np.cos(a)` | `torch.sin(t)`, `torch.cos(t)` |
| Floor and ceiling | `np.floor(a)`, `np.ceil(a)` | `torch.floor(t)`, `torch.ceil(t)` |
| Round to the nearest integer | `np.round(a)` | `torch.round(t)` |
| Restrict values to an interval | `np.clip(a, 0, 1)` | `torch.clamp(t, 0, 1)` |
| Elementwise maximum | `np.maximum(a, b)` | `torch.maximum(t, u)` |
| Elementwise minimum | `np.minimum(a, b)` | `torch.minimum(t, u)` |
| Test for NaN | `np.isnan(a)` | `torch.isnan(t)` |
| Test for finite values | `np.isfinite(a)` | `torch.isfinite(t)` |
| Elementwise approximate equality | `np.isclose(a, b)` | `torch.isclose(t, u)` |
| Overall approximate equality | `np.allclose(a, b)` | `torch.allclose(t, u)` |

For ordinary arrays and tensors, `*` means elementwise multiplication. Use `@` for matrix multiplication:

```python
a * b   # Elementwise multiplication.
a @ b   # Matrix multiplication.

t * u   # Elementwise multiplication.
t @ u   # Matrix multiplication.
```

## 8. Reductions and statistics

NumPy usually names the reduction dimension `axis` and the flag that retains it `keepdims`. The corresponding PyTorch names are usually `dim` and `keepdim`.

| Purpose | NumPy | PyTorch |
|---|---|---|
| Sum | `a.sum(axis=0)` | `t.sum(dim=0)` |
| Mean | `a.mean(axis=0)` | `t.mean(dim=0)` |
| Product | `a.prod(axis=0)` | `t.prod(dim=0)` |
| Maximum values only | `a.max(axis=0)` | `torch.amax(t, dim=0)` |
| Minimum values only | `a.min(axis=0)` | `torch.amin(t, dim=0)` |
| Index of the maximum | `a.argmax(axis=0)` | `t.argmax(dim=0)` |
| Index of the minimum | `a.argmin(axis=0)` | `t.argmin(dim=0)` |
| Standard deviation with denominator N | `a.std(axis=0, ddof=0)` | `t.std(dim=0, correction=0)` |
| Variance with denominator N | `a.var(axis=0, ddof=0)` | `t.var(dim=0, correction=0)` |
| Cumulative sum | `a.cumsum(axis=0)` | `t.cumsum(dim=0)` |
| All elements are true | `a.all(axis=0)` | `t.all(dim=0)` |
| At least one element is true | `a.any(axis=0)` | `t.any(dim=0)` |

### What does `axis=0` or `dim=0` mean?

It identifies the dimension being reduced, which is removed from the result unless you request that it be retained.

```python
a = np.array([[1., 2., 3.], [4., 5., 6.]])
t = torch.tensor([[1., 2., 3.], [4., 5., 6.]])

a.sum(axis=0)   # array([5., 7., 9.])
t.sum(dim=0)    # tensor([5., 7., 9.])

a.sum(axis=1)   # array([6., 15.])
t.sum(dim=1)    # tensor([6., 15.])
```

For this two-dimensional input, reducing dimension `0` combines rows and produces one value per column. Reducing dimension `1` combines columns and produces one value per row.

Keep the reduced dimension when you want later operations to broadcast along the same axis:

```python
a.sum(axis=1, keepdims=True).shape   # (2, 1)
t.sum(dim=1, keepdim=True).shape     # torch.Size([2, 1])
```

### `max` and `min` return different things

NumPy's `a.max(axis=1)` returns values only. PyTorch's `t.max(dim=1)` returns both values and indices:

```python
values, indices = t.max(dim=1)
# values:  tensor([3., 6.])
# indices: tensor([2, 2])
```

To get only the values:

```python
t.max(dim=1).values
torch.amax(t, dim=1)
```

`torch.amax` also supports reducing multiple dimensions at once. Under autograd, it can distribute gradients differently from `max(dim=...)` when several elements tie for the maximum.

Without a dimension argument, `t.max()` returns only the single overall maximum. The analogous distinctions apply to `min` and `amin`.

### Standard deviation and variance use different defaults

NumPy defaults to `ddof=0`, using N as the variance denominator. PyTorch defaults to `correction=1`, using N minus 1.

```python
a = np.array([1., 2., 3.])
t = torch.tensor([1., 2., 3.])

a.std()                  # Approximately 0.8165.
t.std()                  # tensor(1.)
t.std(correction=0)       # Approximately 0.8165.
```

The same default difference applies to `var`. Specify the correction explicitly when matching results.

### Even-length medians differ

NumPy averages the two middle values. `torch.median` returns the lower of the two middle values.

```python
np.median([1., 2., 3., 4.])
# 2.5

t = torch.tensor([1., 2., 3., 4.])
torch.median(t)
# tensor(2.)

torch.quantile(t, 0.5)
# tensor(2.5000)
```

For a supported floating-point dtype, `torch.quantile(t, 0.5)` uses linear interpolation by default and provides the usual midpoint behavior for an even number of values.

## 9. Concatenation, stacking, splitting, and repetition

| Purpose | NumPy | PyTorch |
|---|---|---|
| Join along an existing dimension | `np.concatenate([a, b], axis=0)` | `torch.cat([t, u], dim=0)` |
| Join along a new dimension | `np.stack([a, b], axis=0)` | `torch.stack([t, u], dim=0)` |
| Horizontal stacking | `np.hstack([a, b])` | `torch.hstack([t, u])` |
| Vertical stacking | `np.vstack([a, b])` | `torch.vstack([t, u])` |
| Split into a given number of sections, allowing unequal sizes | `np.array_split(a, 3, axis=0)` | `torch.tensor_split(t, 3, dim=0)` |
| Repeat individual elements or slices | `np.repeat(a, 2, axis=0)` | `torch.repeat_interleave(t, 2, dim=0)` |
| Tile the whole input | `np.tile(a, (2, 3))` | `t.repeat(2, 3)` |
| Broadcast to a target shape | `np.broadcast_to(a, shape)` | `t.expand(shape)` |

### Concatenation preserves rank; stacking adds a dimension

If both inputs have shape `(2, 3)`:

```text
concatenate / cat along dimension 0 -> (4, 3)
stack along dimension 0             -> (2, 2, 3)
```

Inputs to `stack` must have the same shape. Inputs to `cat` must match outside the concatenation dimension for ordinary nonempty inputs.

### `split` is not a direct name-for-name replacement

```python
a = np.arange(6)
t = torch.arange(6)

np.split(a, 2)
# 2 sections, each containing 3 elements.

torch.split(t, 2)
# Sections of size 2: 3 sections in total.

torch.tensor_split(t, 2)
# 2 sections, each containing 3 elements.
```

With an integer section count, `np.split` requires an equal division, while `torch.tensor_split` allows unequal section sizes. The closer counterpart to `torch.tensor_split` is `np.array_split`.

Passing a list introduces another distinction: NumPy's `split` treats it as split indices, while `torch.split` treats it as section sizes. Use `torch.tensor_split` when you want split-index semantics.

### `repeat` also has different meanings

```python
a = np.array([1, 2])
t = torch.tensor([1, 2])

np.repeat(a, 2)                 # array([1, 1, 2, 2])
torch.repeat_interleave(t, 2)   # tensor([1, 1, 2, 2])

np.tile(a, 2)                   # array([1, 2, 1, 2])
t.repeat(2)                    # tensor([1, 2, 1, 2])
```

Broadcasting avoids physically repeating the data. `np.broadcast_to` returns a read-only view; `t.expand` returns a view that can have multiple logical elements sharing one memory location. Clone an expanded tensor before modifying elements independently.

## 10. Sorting, searching, and linear algebra

| Purpose | NumPy | PyTorch |
|---|---|---|
| Sorted values | `np.sort(a)` | `torch.sort(t).values` |
| Sorting indices | `np.argsort(a)` | `torch.argsort(t)` |
| Unique values | `np.unique(a)` | `torch.unique(t)` |
| Insertion positions in sorted input | `np.searchsorted(a, v)` | `torch.searchsorted(t, v)` |
| Largest k elements | Usually combine `argsort` or `argpartition` with indexing | `torch.topk(t, k)` |
| Matrix multiplication | `a @ b`, `np.matmul(a, b)` | `t @ u`, `torch.matmul(t, u)` |
| Dot product of two 1-D vectors | `np.dot(a, b)` | `torch.dot(t, u)` |
| Vector or matrix norm | `np.linalg.norm(a)` | `torch.linalg.norm(t)` |
| Solve a linear system | `np.linalg.solve(a, b)` | `torch.linalg.solve(t, u)` |
| Matrix inverse | `np.linalg.inv(a)` | `torch.linalg.inv(t)` |
| Determinant | `np.linalg.det(a)` | `torch.linalg.det(t)` |
| Singular value decomposition | `np.linalg.svd(a)` | `torch.linalg.svd(t)` |
| Eigenvalues and eigenvectors | `np.linalg.eig(a)` | `torch.linalg.eig(t)` |
| Einstein summation | `np.einsum(...)` | `torch.einsum(...)` |

### PyTorch sorting returns values and indices

```python
t = torch.tensor([30, 10, 20])

values, indices = torch.sort(t)
# values:  tensor([10, 20, 30])
# indices: tensor([1, 2, 0])
```

`torch.topk` also returns values and indices. Separately, note that NumPy's `np.sort(a)` returns a sorted copy, while the array method `a.sort()` sorts in place and returns `None`.

### Prefer `@` or `matmul` for matrix multiplication

`np.dot` accepts one-dimensional, two-dimensional, and higher-dimensional inputs, with behavior depending on their ranks. `torch.dot` accepts only two one-dimensional tensors.

For ordinary or batched matrix multiplication, use `@` or `matmul` in both libraries:

```text
A.shape = (5, 2, 3)
B.shape = (5, 3, 4)

(A @ B).shape = (5, 2, 4)
```

The last two dimensions describe each matrix; preceding dimensions are batch dimensions and follow broadcasting rules.

## 11. Random numbers

For new NumPy code, create a random number generator explicitly:

```python
rng = np.random.default_rng(42)
```

| Purpose | NumPy | PyTorch |
|---|---|---|
| Uniform values in `[0, 1)` | `rng.random((2, 3))` | `torch.rand((2, 3))` |
| Standard normal values | `rng.standard_normal((2, 3))` | `torch.randn((2, 3))` |
| Random integers with an exclusive upper bound | `rng.integers(0, 10, size=(2, 3))` | `torch.randint(0, 10, (2, 3))` |
| Random permutation of `0` through `n - 1` | `rng.permutation(n)` | `torch.randperm(n)` |

Set PyTorch's default random generators' seed with:

```python
torch.manual_seed(42)
```

Or use an independent generator:

```python
g = torch.Generator().manual_seed(42)
t = torch.rand((2, 3), generator=g)
```

The same seed does **not** produce matching sequences in NumPy and PyTorch. It also does not guarantee that all PyTorch operations are deterministic across devices or versions.

## 12. Copies, views, and conversion between libraries

| Operation | Typical behavior |
|---|---|
| `b = a` or `u = t` | Adds another Python reference to the same object |
| `a.copy()` | Copies the NumPy data |
| `t.clone()` | Copies tensor data and preserves an autograd connection when gradient tracking applies |
| `t.detach()` | Removes the autograd connection but shares storage |
| `t.detach().clone()` | Removes the autograd connection and copies the data |
| `torch.tensor(a)` | Copies data |
| `torch.from_numpy(a)` | Shares storage between a supported NumPy array and a CPU tensor |
| `torch.as_tensor(a)` | Shares data when possible; may copy for conversion |
| `t.numpy()` | Shares storage when the tensor satisfies the direct conversion requirements |

### NumPy to PyTorch

```python
a = np.array([1., 2., 3.], dtype=np.float32)

t1 = torch.from_numpy(a)  # Shares storage.
t2 = torch.tensor(a)      # Copies data.

a[0] = 99

print(t1)
# tensor([99.,  2.,  3.])

print(t2)
# tensor([1., 2., 3.])
```

Sharing requires a supported dtype and layout. For example, a NumPy array reversed with `a[::-1]` has negative strides, which `torch.from_numpy` does not support. Copy that array first:

```python
reversed_tensor = torch.from_numpy(a[::-1].copy())
```

### PyTorch to NumPy

For an ordinary tensor with a NumPy-compatible dtype and layout, a common conversion is:

```python
a = t.detach().cpu().numpy()
```

The steps are:

1. `detach()` obtains a tensor without an autograd connection.
2. `cpu()` obtains a tensor on the CPU.
3. `numpy()` exposes that CPU tensor as a NumPy array.

If `t` is already on the CPU, this result can still share storage with it. `detach()` does not make a copy. A GPU-to-CPU transfer creates separate CPU storage, which the resulting NumPy array can then share.

To obtain an independent NumPy copy:

```python
a = t.detach().cpu().numpy().copy()
```

Some tensor dtypes and layouts require an additional conversion, and tensors with unresolved conjugate or negative view flags need those flags resolved before direct NumPy conversion.

### Copying data and preserving gradients are separate choices

For an existing tensor, `torch.tensor(t)` constructs a detached copy. Use `t.clone()` when you want a copy that remains connected to the computation graph, or `t.detach().clone()` when you deliberately want independent data without that connection.

## 13. Devices and automatic differentiation

PyTorch's device and autograd APIs have no direct equivalents in ordinary NumPy array operations.

### Moving data between devices

```python
t = torch.tensor([1., 2., 3.])

if torch.cuda.is_available():
    t = t.to("cuda")

t = t.cpu()
```

Tensors participating in the same operation generally need to be on the same device. Device moves and dtype conversions are commonly expressed through `to`:

```python
t = t.to(device="cpu", dtype=torch.float64)
```

### Computing gradients

```python
x = torch.tensor([1., 2., 3.], requires_grad=True)

loss = (x ** 2).sum()
loss.backward()

print(x.grad)
# tensor([2., 4., 6.])
```

The scalar loss is the sum of the squared elements. Its derivative with respect to each element is twice that element.

| Purpose | PyTorch |
|---|---|
| Enable gradient tracking at creation | `requires_grad=True` |
| Enable it on an existing eligible tensor | `t.requires_grad_()` |
| Run backpropagation | `loss.backward()` |
| Read accumulated gradients | `t.grad` |
| Detach a tensor from its graph | `t.detach()` |
| Disable gradient recording in a block | `with torch.no_grad():` |
| Clear gradients managed by an optimizer | `optimizer.zero_grad()` |

Gradients accumulate by default. Training loops usually clear them before computing the next batch's gradients. By default, gradients are retained in `.grad` for leaf tensors that require gradients; intermediate tensors need `retain_grad()` if you want to inspect their `.grad` values after backpropagation.

Only floating-point and complex tensors can require gradients. Converting data to NumPy and calculating there does not preserve PyTorch's computation graph.

### In-place operations need care around autograd

Many PyTorch methods ending in `_` modify the tensor in place:

```python
t = torch.tensor([1., 2., 3.])
t.add_(1)   # Modifies t itself.
t.zero_()   # Fills t itself with zeros.
```

An in-place operation can invalidate data needed for backward computation, or be disallowed on a leaf tensor that requires gradients. Code that works as a NumPy assignment may therefore need a different form in a differentiable PyTorch computation.

## 14. A complete example: standardizing features

Suppose each row is a sample and each column is a feature. To standardize each feature, subtract its column mean and divide by its column standard deviation.

### NumPy

```python
a = np.array([
    [1, 2, 3],
    [4, 5, 6],
], dtype=np.float32)

mean = a.mean(axis=0, keepdims=True)
std = a.std(axis=0, keepdims=True, ddof=0)

z_numpy = (a - mean) / (std + 1e-8)
```

### PyTorch

```python
t = torch.tensor([
    [1, 2, 3],
    [4, 5, 6],
], dtype=torch.float32)

mean = t.mean(dim=0, keepdim=True)
std = t.std(dim=0, keepdim=True, correction=0)

z_torch = (t - mean) / (std + 1e-8)
```

Both produce corresponding results close to:

```text
[[-1., -1., -1.],
 [ 1.,  1.,  1.]]
```

For these CPU tensors, compare the results with:

```python
np.allclose(z_numpy, z_torch.numpy())
# True
```

The visible parameter changes are small: `axis` becomes `dim`, and `keepdims` becomes `keepdim`. Specifying `correction=0` is essential to match NumPy's standard deviation convention.

## Quick migration reference

| NumPy concept or API | Common PyTorch counterpart |
|---|---|
| `ndarray` | `Tensor` |
| `axis` | Usually `dim` |
| `keepdims` | `keepdim` |
| `a.size` | `t.numel()` |
| `concatenate` | `cat` |
| `expand_dims` | `unsqueeze` |
| `transpose` with a full axis permutation | `permute` |
| `swapaxes` | `transpose` |
| `astype` | `to(dtype=...)` |
| `copy` | `clone`, with a separate decision about gradient tracking |
| `tile` | `repeat` |
| `repeat` | `repeat_interleave` |
| `array_split` | `tensor_split` |
| `max(axis=...)`, values only | `amax(dim=...)` |
| Default `std` and `var` | Use `correction=0` to match the denominator |

When migrating a function, check its return structure, default parameters, accepted dtypes, and memory behavior. For PyTorch, also check the device and whether gradients need to flow through the operation.

## Official documentation

- [NumPy API reference](https://numpy.org/doc/stable/reference/index.html)
- [PyTorch API reference](https://docs.pytorch.org/docs/stable/torch.html)
- [NumPy broadcasting guide](https://numpy.org/doc/stable/user/basics.broadcasting.html)
- [PyTorch broadcasting semantics](https://docs.pytorch.org/docs/stable/notes/broadcasting.html)
- [PyTorch autograd mechanics](https://docs.pytorch.org/docs/stable/notes/autograd.html)
