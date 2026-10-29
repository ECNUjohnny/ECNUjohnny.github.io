+++

authors = "Johnny"
title = "more ops in pytorch"
date = "2026-10-29"
tags = [
    "python",
    "pytorch",
]

+++


# PyTorch Tensor Operations: `torch.gather`, `torch.triu`, and `masked_fill`

This guide explains three useful PyTorch operations with examples, expected outputs, and common pitfalls.

## 1. `torch.gather`

### Purpose

`torch.gather` selects elements from a tensor using an index tensor **along a specified dimension**. The index tensor describes which element to select at each output position.

### Syntax

```python
torch.gather(input, dim, index)
```

- `input`: The source tensor.
- `dim`: The dimension along which values are selected.
- `index`: A `torch.long` tensor containing the indices to select.
- **Output shape:** The output has the same shape as `index`.

`input` and `index` must have the same number of dimensions. They do not broadcast against each other. For every dimension other than `dim`, the corresponding size of `index` must not exceed the size of `input`.

### Example: Gather values from each row

```python
import torch

x = torch.tensor([
    [10, 11, 12],
    [20, 21, 22],
])

index = torch.tensor([
    [2, 1],
    [0, 2],
])

result = torch.gather(x, dim=1, index=index)
print(result)
```

Output:

```text
tensor([[12, 11],
        [20, 22]])
```

Because `dim=1`, PyTorch selects columns within each row:

- Row 0: select columns 2 and 1, giving `[12, 11]`.
- Row 1: select columns 0 and 2, giving `[20, 22]`.

The result has shape `(2, 2)`, which is exactly the shape of `index`.

### Example: Select the score for each class label

A common machine-learning use case is selecting the predicted score corresponding to each sample's target class.

```python
logits = torch.tensor([
    [0.1, 0.8, 0.1],
    [0.7, 0.2, 0.1],
])  # Shape: (batch_size=2, num_classes=3)

labels = torch.tensor([1, 0])  # One class index per sample

selected_scores = logits.gather(dim=1, index=labels.unsqueeze(1))
selected_scores = selected_scores.squeeze(1)

print(selected_scores)
```

Output:

```text
tensor([0.8000, 0.7000])
```

`labels.unsqueeze(1)` changes the shape from `(2,)` to `(2, 1)`, so it has the same number of dimensions as `logits`. The `gather` result has shape `(2, 1)`; `squeeze(1)` removes that singleton dimension and produces shape `(2,)`.

### Common pitfalls

- **`index` should normally have dtype `torch.long`.** Its values are integer indices, not Boolean conditions.
- **`index` determines the output shape.** `gather` does not automatically return one value per row unless the index tensor is shaped that way.
- **Dimensions matter.** For a tensor shaped `(batch_size, num_classes)`, use `dim=1` to select a class within each sample. Use `dim=0` to select along the batch dimension.
- **Do not confuse `gather` with ordinary advanced indexing.** `gather` performs element selection along one chosen dimension while preserving the index tensor's shape.

---

## 2. `torch.triu`

### Purpose

`torch.triu` returns the upper-triangular part of a matrix. Elements outside the selected upper triangle are set to zero. It also supports batches of matrices, such as tensors shaped `(batch_size, rows, columns)`.

### Syntax

```python
torch.triu(input, diagonal=0)
```

- `input`: A 2-D matrix or a batch of matrices.
- `diagonal`: Which diagonal to use as the boundary. The default is `0`, the main diagonal.

For a matrix element at row `i` and column `j`, the element is retained when:

```text
j - i >= diagonal
```

Otherwise, it is set to zero.

### Example: Keep the main diagonal and everything above it

```python
import torch

x = torch.tensor([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
])

print(torch.triu(x))
```

Output:

```text
tensor([[1, 2, 3],
        [0, 5, 6],
        [0, 0, 9]])
```

### Understanding `diagonal`

```python
print(torch.triu(x, diagonal=1))
```

Output:

```text
tensor([[0, 2, 3],
        [0, 0, 6],
        [0, 0, 0]])
```

With `diagonal=1`, the main diagonal is excluded; only elements strictly above it are retained.

```python
print(torch.triu(x, diagonal=-1))
```

Output:

```text
tensor([[1, 2, 3],
        [4, 5, 6],
        [0, 8, 9]])
```

With `diagonal=-1`, one diagonal below the main diagonal is also retained.

| `diagonal` value | Effect |
|---:|---|
| `1` | Keep elements strictly above the main diagonal. |
| `0` | Keep the main diagonal and everything above it. |
| `-1` | Also keep one diagonal below the main diagonal. |

### Typical use: Build a causal attention mask

In autoregressive models, a position should not attend to future positions. A Boolean upper-triangular matrix can mark those future positions:

```python
seq_len = 4
future_mask = torch.triu(
    torch.ones(seq_len, seq_len, dtype=torch.bool),
    diagonal=1,
)

print(future_mask)
```

Output:

```text
tensor([[False,  True,  True,  True],
        [False, False,  True,  True],
        [False, False, False,  True],
        [False, False, False, False]])
```

Here, `True` marks a future position that should be blocked. The `diagonal=1` setting is important because it leaves the main diagonal unmasked, allowing each position to attend to itself.

### Common pitfalls

- `torch.triu` **sets excluded elements to zero**; it does not remove them or change the tensor's shape.
- For a Boolean input, excluded elements become `False`, which is useful for constructing masks.
- A positive `diagonal` moves the boundary upward; a negative value includes more diagonals below the main diagonal.

---

## 3. `Tensor.masked_fill`

### Purpose

`masked_fill` replaces elements at positions where a Boolean mask is `True` with a specified value.

### Syntax

```python
output = input.masked_fill(mask, value)
```

- `input`: The tensor to modify logically.
- `mask`: A Boolean tensor whose shape is broadcastable to the shape of `input`.
- `value`: The value used to replace masked elements.
- **Return value:** A new tensor; the original tensor is not modified in-place.

The in-place variant is `input.masked_fill_(mask, value)` (note the trailing underscore).

### Example: Replace selected elements

```python
import torch

x = torch.tensor([
    [1.0, 2.0, 3.0],
    [4.0, 5.0, 6.0],
])

mask = torch.tensor([
    [False, True,  False],
    [True,  False, False],
])

result = x.masked_fill(mask, value=-1.0)

print(result)
```

Output:

```text
tensor([[ 1., -1.,  3.],
        [-1.,  5.,  6.]])
```

Only positions where `mask` is `True` are replaced. Positions where it is `False` keep their original values.

### Example: Combine `triu` and `masked_fill` for attention scores

Attention mechanisms often use a very negative value to prevent a softmax from assigning meaningful probability to forbidden positions.

```python
scores = torch.tensor([
    [1.0, 2.0, 3.0, 4.0],
    [1.0, 2.0, 3.0, 4.0],
    [1.0, 2.0, 3.0, 4.0],
    [1.0, 2.0, 3.0, 4.0],
])

seq_len = scores.size(-1)
future_mask = torch.triu(
    torch.ones(seq_len, seq_len, dtype=torch.bool),
    diagonal=1,
)

masked_scores = scores.masked_fill(future_mask, float("-inf"))
print(masked_scores)
```

Output:

```text
tensor([[1., -inf, -inf, -inf],
        [1., 2., -inf, -inf],
        [1., 2., 3., -inf],
        [1., 2., 3., 4.]])
```

This pattern is commonly used before applying softmax to attention scores. The mask has shape `(seq_len, seq_len)` and can broadcast across leading batch and head dimensions in a tensor shaped `(batch_size, num_heads, seq_len, seq_len)`.

### Common pitfalls

- **`True` means “replace this element.”** This is easy to confuse with some selection operations where `True` means “keep this element.”
- **The mask must be Boolean and broadcastable to the input shape.** A mask does not need to have exactly the same shape if normal broadcasting rules can expand it.
- **`masked_fill` is out-of-place.** Use `masked_fill_` only when you specifically want to modify the original tensor in-place.
- **`-inf` is a masking value, not a general replacement value.** It is useful for logits before softmax, but check that each softmax row has at least one unmasked position to avoid undefined results such as `NaN`.

---

## Quick comparison

| Operation | Main purpose | Key behavior |
|---|---|---|
| `torch.gather` | Select values using indices along one dimension | Output shape follows `index`. |
| `torch.triu` | Keep an upper-triangular region | Values outside the region become zero (or `False` for Boolean input). |
| `Tensor.masked_fill` | Replace values selected by a Boolean mask | Positions with `True` are replaced by `value`. |

## Official documentation

- [`torch.gather`](https://docs.pytorch.org/docs/stable/generated/torch.gather.html)
- [`torch.triu`](https://docs.pytorch.org/docs/stable/generated/torch.triu.html)
- [`Tensor.masked_fill`](https://docs.pytorch.org/docs/stable/generated/torch.Tensor.masked_fill.html)
