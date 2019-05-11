# Introduction

![KDE definition](https://github.com/kexinrong/rehashing/blob/master/plots/kde_definition.png)

The problem of Kernel Evaluation is to produce a multiplicative approximation of the Kernel Density Estimate (KDE) at a query point q when the query density μ is larger than some threshold τ. We show how we can exploit this fact to compress the set of points and weights into a "sketch" such that we can still accurately estimate the density for all densities larger than τ. 

![Sketch definition](https://github.com/kexinrong/rehashing/blob/master/plots/sketch_definition.png)

The advantage of using a sketch is that if the original space or pre-processing time usage of a kernel evaluation algorithm is T(n), using a sketch (or combination of a few sketches) results in a complexity ~ T(m). For m << n, this reduction can be significant. Below, we give an example of the reduction achieved by using Hashing-Based-Sketch on real-world datasets.



![Overhead Reduction](https://github.com/kexinrong/rehashing/blob/master/plots/overhead_reduction.png "Overhead reduction using Sketching")

In what follows, we describe the main approaches used to "sketch" the KDE and our modifications that constrain the overall pre-processing time of all sketching methods to be ~ 2 * n (where n is the number of points) in the dataset.

# Approaches

The main approaches on "sketching" KDE are:
- Uniform Random Sampling
- [Kernel Herding](https://alex.smola.org/papers/2010/CheWelSmo10.pdf)
- Hashing-Based-Sketch
- [Sparse-Kernel-Approximation](https://arxiv.org/abs/1503.00323)

![sketching approaches](https://github.com/kexinrong/rehashing/blob/master/plots/sketching_table.png)

## Uniform Random Sampling

A "sketch" of size s is constructed by repeating s times:
- sampling point i with probability u_i and 
- adding the point in the sketch with weight 1 / s

## Approximate Kernel Herding

The kernel herding algorithm first estimates the density of the dataset via random sampling; the sketch is then produced by iteratively selecting points with maximum residual density to add to the sketch. The algorithm has a complexity of O(nm + ns), where m stands for the sample size used to produce the initial density estimates. 

![Approximate Kernel Herding](https://github.com/kexinrong/rehashing/blob/master/plots/AKH.png "Approximate Kernel Herding")

### Modification

To keep Herding under the same 2n pre-processing budget, we downsample the dataset to size n_h = n / s, and use m = s samples to estimate the initial density. This means that the larger the sketch size is, the less accurate the initial density estimate is. As a result, we observe degrading performance at larger sketch sizes s > \sqrt(n). 

## Hashing-Based-Sketch

Given a hash function that partitions the data in hash buckets H_1,..., H_B, the scheme we propose samples a random point by first sampling a hash bucket H_i with probability   ~ ( \sum_{j in H_i} u_j ) ^ γ and then sampling a point j from the bucket with probability ~ u_j. The weights are chosen such that the sample is an unbiased estimate of the density. This scheme interpolates between uniform over buckets and uniform over the whole data set as we vary γ in [0,1].  

We pick γ* so that the variance is controlled and with the additional property that any non-trivial bucket  u_(H_i) = \sum_{j in H_i} u_j \geq τ will have a representative in the sketch with some probability. This last property is useful in estimating low-density points (e.g. for outlier detection).  For any hash table H and a non-negative vector u that sums to 1,  let  B  denote the number of buckets and  u_max =: max{ u_(H_i)): i in [B]} the maximum weight of any hash bucket of the hash table H. The precise definition of our Hashing-Based-Sketch is given  below.

![Hashing-Based-Sketch](https://github.com/kexinrong/rehashing/blob/master/plots/HBS.png "Hashing-Based-Sketch")

The Hashing-Based-Sketch algorithm comes with the following theoretical guarantees.

![HBS Analysis](https://github.com/kexinrong/rehashing/blob/master/plots/HBS_theorem.png "HBS Analysis")

### Modification

In practice to create the sketch with HBS, we used 5 hash tables, each hashing a subset of  2 * n / 5 points in the dataset. In practice, we found that varying this small constant on the number of hash tables does not have a noticeable impact on the performance. 

## Sparse Kernel Approximation

The sparse kernel approximation (SKA) algorithm  produces the sketch by greedily finding s points in the dataset that minimizes the maximum distance (k-center). The associated weights are given by solving an equation involving the kernel matrix of the selected points. SKA has a complexity of O(ns+s^3), dominated by the matrix inversion procedure used in solving the kernel matrix equation. 

![Sparse Kernel Approximation](https://github.com/kexinrong/rehashing/blob/master/plots/SKA.png "Sparse Kernel Approximation")

### Modification

To ensure that SKA is able to match the sketch size of alternative methods under the same compute budget of 2n, we augment SKA with random samples when necessary:   
- If the target sketch size is smaller than  n ^ 1/3 (s < n ^ 1/3), we use SKA to produce a sketch of size s from a subsample of n / s data points. 
- For  s > n ^ 1/3, we use SKA to produce a sketch of size  s_c =  n ^ 1/3 from a subsample of  n_c =  n ^ 2/3 data points. We match the differences in sketch size by taking an additional  s - s_c random samples from the remaining n - n_c data points. The final estimate is a weighted average between the SKA sketch and the uniform sketch: 1 / s_c for SKA and (1 - 1 / s_c) for uniform samples. The above weights are determined by the number of data points involved in both sketches.
 

The modification can be thought of as using SKA as a form of *regularization on random samples*. Since SKA iteratively selects points that are farthest away from the current set (k-center), the resulting sketch is helpful in predicting the "sparser" regions of the space. These sparser regions, in turn, are the ones that survive in the n ^ 2/3 random sub-sample of the dataset (sub-sampling with probability n ^ -1/3), therefore SKA naturally will include points from "sparse" clusters of size ~ n ^ 1/3 in the original set.




# Experiments and Comparison

Here we compare the above approaches in sketching the KDE for real-world datasets. We pick parameters so that they all have pre-processing time ~ 2 * n. The figure below 
reports the relative error on random queries (left) and low-density queries (right). Uniform, SKA and HBS achieve similar mean error on random queries, while the latter two have improved performance on low-density queries, with HBS performing slightly better than SKA. By design, HBS has similar performance with random sampling on average but performs better on relatively ``sparse" regions due to the theoretical guarantee that buckets with weight at least $\tau$ are sampled with high probability. Combining SKA with random samples initially results in performance degradation but eventually acts as a form of regularization improving upon random.  Kernel Herding is competitive only for a small number of points in the sketch. 
 
![Sketching Experiments](https://github.com/kexinrong/rehashing/blob/master/plots/sketching.png "Sketching Experiments")