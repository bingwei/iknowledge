# 数学基础

## Master 定理

Master定理是用来分析**分治算法**的时间复杂度 ([Wikipedia](https://en.wikipedia.org/wiki/Master_theorem_(analysis_of_algorithms))。对于 $$ T(n) = a T(\frac{n}{b}) + O(n^d) $$:

![Master theorem](master.png)

常数a, b, c分别对应: (1) divide, (2) conquer, (3) combine.

Master定理的证明不是很重要，主要是要有一个recursion tree的思想：

![The tree of master theorem](master-tree.png)

通过几个例子来理解master定理，主要体会一下是怎么卡住那个函数的边界的。

1. 整数相乘算法 ([Wikipedia: Karatsuba algorithm](https://en.wikipedia.org/wiki/Karatsuba_algorithm)) [DPV 2.2]
1. 矩阵相乘算法 ([Wikipedia: Strassen algorithm](https://en.wikipedia.org/wiki/Strassen_algorithm)) [DPV 2.5]
1. 多项式相乘算法 （略复杂）([Wikipedia: Fast Fourier transform](https://en.wikipedia.org/wiki/Fast_Fourier_transform)) [DPV 2.6]

参考章节：[DPV 2.2], [CLRS 4]
