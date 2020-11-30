---
title: The "Keng"s in PyTorch
date: 2020-10-19 19:43:32
tags:
- Python
- PyTorch
- ML
- English
---

#  What is the "Keng"

Well. "Keng" is actually the Pinyin of the Chinese character "坑", meaning somgthing may mess you up. Here the "Keng" stands for the pitfall better to be aware of when using PyTorch in several specific situations.

I feel like to write something about ML in this blog, but do not actually have any topic at hand. "Keng" series will be a good start to summarize my experience in ML. Hope these experiences could help you :-).

# Keng - 1

## Problem

Generally, while using  `torch.nn.parallel.DistributedDataParallel` , the code runs on multiple GPUs independently, meaning one process per GPU. The memory usage of each GPU should be close. 

But I once found that a part of memory of gpu 0 was occupied by processes running on gpu 1-3 and caused `CUDA out of memory` error.

## Cause

When loading a pretrained model with `torch.load()`, it put data to gpu 0.

## Solution

An easy solution is to remap the data to cpu with `torch.load('checkpoint.pth', map_location=torch.device('cpu'))`

