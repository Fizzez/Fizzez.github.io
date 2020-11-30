---
title: Differentiable Operations in PyTorch
date: 2020-11-23 18:30:41
tags:
- Python
- PyTorch
- ML
- English
---

# Description

I was wondering how PyTorch deals with those mathematically non-differentiable loss function for these days. So I have a brief summary here to share my findings.

## TL;DR:

Basically, all the operations provided by PyTorch are **'differentiable'**. As for mathematically non-differentiable operations such as relu, argmax, mask_select and tensor slice, the elements at which gradients are not able to be calculated are set to gradient 0.

# Investigation

## Mathematically non-differentiable situation

For mathematically non-differentiable operations such as relu, argmax, mask_select and tensor slice, the elements at which gradients are not able to be calculated are set to gradient 0.

Take absolute function for example:
$$
abs(x)=\left\{
\begin{aligned}
x, x > 0 \\
0, x = 0 \\
-x, x < 0
\end{aligned}
\right.
$$
Absolute function is not differentiable at $x=0$, mathematically, but PyTorch set the gradient at this point to be 0. Here is a test:

```python
import torch
for i in range(11):
  x = torch.tensor([i-5], dtype=float, requires_grad=True)
  y = torch.abs(x)
  y.backward()
  print(x.grad)
```

The output will be:

```plain
tensor([-1.], dtype=torch.float64)
tensor([-1.], dtype=torch.float64)
tensor([-1.], dtype=torch.float64)
tensor([-1.], dtype=torch.float64)
tensor([-1.], dtype=torch.float64)
tensor([0.], dtype=torch.float64)
tensor([1.], dtype=torch.float64)
tensor([1.], dtype=torch.float64)
tensor([1.], dtype=torch.float64)
tensor([1.], dtype=torch.float64)
tensor([1.], dtype=torch.float64)
```

Like [what Mertens said in this answer](https://discuss.pytorch.org/t/non-differentiable-loss-function-in-cnn/67200/8), 

>This function isn’t analytically differentiable. However, at every point except 0, it is. In practice, for the purpose of gradient descent, it works well enough to treat the function as if it were differentiable. You’ll rarely be computing the gradient at precisely 0, and even if you do, it’s sufficient to handle things via a special case.

As for how to handle the special case, [here](https://pytorch.org/tutorials/beginner/examples_autograd/two_layer_net_custom_function.html) is a good official example. The case could be specially treated in your `backward` function.