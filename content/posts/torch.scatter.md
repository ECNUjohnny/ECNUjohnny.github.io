+++
authors = "Johnny"
title = "torch.scatter"
date = "2026-10-22"
tags = [
   "pytorch", "python"
]
+++


# 1. Function Definition and Arguments

`torch.scatter` (and its in-place version `scatter_`) is used to embed or write all values from a source tensor (src) into a target tensor (input / self) at the positions specified by the index tensor along a given dimension (dim). It is commonly used for generating One-hot encodings, building sparse matrices, or aggregating features in sequence models and Graph Neural Networks (GNNs).

```python

# In-place version (modifies the tensor itself)
tensor.scatter_(dim, index, src, reduce=None)

# Out-of-place version (returns a new tensor)
torch.scatter(input, dim, index, src, *, reduce=None)

```

- dim: The axis along which to index and fill.
- index: A LongTensor indicating the indices of elements to scatter. Its shape must match src (or be broadcastable).
- src: The source tensor containing the values to be scattered (can also be a scalar value).
- reduce (Optional): The reduction operation to apply, such as 'add' (summing up values) or 'multiply'. PyTorch also provides a dedicated scatter_add_ method.

# 2. Basic Code Examples

```python

# Example 1: 2D Tensor Row/Column Filling
 


import torch

# 1. Create a target tensor filled with zeros
input_tensor = torch.zeros(3, 4)
print("Initial input_tensor:\n", input_tensor)

# 2. Prepare the index and src tensors
# The index tensor specifies the row indices for each column element when dim=0
index = torch.tensor([[0, 1, 2, 0], 
                      [2, 0, 1, 2]])
src = torch.tensor([[1, 2, 3, 4], 
                    [5, 6, 7, 8]])

# 3. Perform scatter along dim=0 (Row-wise alignment)
output = input_tensor.scatter_(dim=0, index=index, src=src)
print("Result after scatter_:\n", output)


# Example 2: Classic Application – Generating One-Hot Encodings

# Assume batch_size = 3 and the number of classes = 5
batch_size, num_classes = 3, 5
labels = torch.tensor([1, 4, 3])  # Target class indices

# Create an all-zero matrix
one_hot = torch.zeros(batch_size, num_classes)

# Use scatter_ to fill 1s
# dim=1 means we fill values along the column axis based on the positions in labels
one_hot.scatter_(dim=1, index=labels.unsqueeze(1), value=1)

print("One-hot Result:\n", one_hot)

```

# 3. Common Derivatives and Related Functions

- scatter_add_: If there are duplicate indices in the index tensor, a standard scatter_ will overwrite the previous value. scatter_add_ ensures that values at the same index are accumulated instead.
- scatter_reduce_: A more flexible aggregation tool introduced in newer PyTorch versions (supporting operations like "sum", "prod", "mean", "amax", "amin").
- Relationship with gather: While gather collects/extracts values from specified positions of a tensor, scatter distributes/writes values into specified positions. They are effectively inverse operations during backpropagation.
