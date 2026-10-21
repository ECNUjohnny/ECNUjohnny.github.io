+++
authors = "Johnny"
title = "trifles in pytorch"
date = "2026-10-21"
tags = [
   "pytorch", "python"
]
+++

1. function in pytorch ends with "_" means this function is a "in_place" function --- it will modify the tensor instead of return a copy of that tensor. 

2. view() and slice ([index1:index2]) will return the shallow copy of a tensor. clone() will return the deep copy of a tensor.

3. use torch.fill(tensor, number) to fulfill a tensor by a same number. 

4. if we want to calculate the mean of multi-dimensions, we can use a list as a parameter in torch.mean(), e.g. torch.mean(tensor, dim=[2, 3]) will change the shape [B, C, H, W] into [B, C] 

5. In the early version of pytorch, tensor is pure data and does not contain any primitives of backward propagation. We should use Variable (a class) to wrap a tensor to operate backward propagation on this tensor.

6. more may be supplemented in the future. 