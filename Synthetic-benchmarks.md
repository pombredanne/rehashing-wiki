# Introduction

There are two major approaches for fast kernel evaluation:
- *Space partitioning*, as exemplified by the [Fast Multipole Method](https://math.nyu.edu/faculty/greengar/shortcourse_fmm.pdf) and [Dual Tree Algorithms](https://arxiv.org/pdf/1304.4327.pdf)
- *Monte Carlo estimation*, as exemplified by uniform random sampling.

HBE combines the two approaches as it uses randomized (Monte Carlo) space partitions (Locality Sensitive Hashing) to construct unbiased estimates of the density and gets provably accurate estimates by averaging many such samples.

In order to construct benchmarks that are difficult for all methods and do not favor a particular approach we require the instances to have diverse:
- *distance structure:* meaning that multiple distance scales should contribute to the query density
- *correlation structure:* meaning that there should be multiple orthogonal directions (high-dimensionality) around the query that contribute to the query density
- *entropic structure:* meaning that both "sparse" (few) and "dense" cluster of points should have a non-trivial contribution.

Each of the above desiderata aims at creating instances where no single geometric aspect of the problem can be used to create fast algorithms, and a holistic approach is required.

# A family of benchmarks

Since the problem of kernel density is query dependent and the kernel typically depends only on the distance, we shall always assume that the query point is at the origin q=0 in R^d.

We further assume that the kernel is an invertible function of the distance K(r)\in [0,1] and let K_inv(\mu) in  [0,\infty) be the inverse function. For example, the exponential kernel is given by K(r) = e ^-r and the inverse function is given by K_inv(\mu) = log(1 / \mu). 

## Tunable family of instances 
Given parameters **(\mu, D, n, s, d)**, the dataset is created with points lying in **D** random directions in R^**d** and **s** distance scales (equally spaced between 0 and R = K_inv(**\mu**)  such that the *contribution from each direction and scale* to the kernel density at the origin is *equal* to **\mu**. To achieve this the number of points n_j placed at the j-th distance scale r_j is given by 

n_j = floor( **n** *\mu / K(r_j) )

The reasoning behind this design choice is to make sure that we have diversity in the distance scales that matter in the problem, so as not to favor a particular class of methods.  Also, placing the points in the same direction makes the instance more difficult for HBE as the variance increases. We give an example visualization of such data sets below for \mu = 0.01, D = 3, s = 4, d = 2, sigma = 0.05, where each direction is plotted with a different color. 

![Tunable instance](https://github.com/kexinrong/rehashing/blob/master/plots/correlated2.png)

We have the following remarks:
- If D << n  this class of instances becomes highly structured with a small number of tightly knit "clusters". One would expect in this case, space-partitioning methods to perform well.  
- On the other hand, if D >> n the instances become spread out (especially in high dimensions). This type of instance is ideal for sampling based methods when s = 1, and difficult for space-partitioning methods.

The exact procedure in pseudo-code is given below.

![Tunable instance](https://github.com/kexinrong/rehashing/blob/master/plots/synthetic_algorithm.png)

Based on the above remarks we propose the following sub-class of instances.
 
## Worst-case instances

In order to create an instance that is *hard for all methods* we take a union of the two extremes D >> n and D >> n. We call such instances ``worst-case"   as there does not seem to be a single type of structure that one can exploit, and these type of instances realize the worst-case variance bounds for both HBE and RS. In particular, if we want to generate an instance with N points, we first set D a small constant and n ~ N and take the union of such a dataset with another using D ~ N ^ (1-o(1)) and n ~ N ^ o(1). An example of such a dataset is given in the next figure.

![Generic instance](https://github.com/kexinrong/rehashing/blob/master/plots/generic2.png)

## D-structured instances

"Worst-case" instances are aimed to be difficult for any kernel evaluation method. In order to create instances that have more varied structure, we use our basic method to create a single parameter family of instance by fixing (N, \mu, \sigma, s, d) and setting n = N/D. We call this family of instances as *D-structured*. As one increases D, two things happen:
- The number of directions (clusters) increases.
- n = N/D decreases and hence certain distance scales disappear. By construction,  if  n\mu < K(r_j), equivalently D_j > N * \mu / K(r_j) then the j-th distance scale will have no points assigned to it. 

Hence, for this family when D << N / D, i.e.  D << \sqrt(N), the instances are highly structured and we expect space-partitioning methods to perform well. On the other extreme as D increases and different distance scales start to die out the performance of random sampling keeps improving until there is only one (the outer) distance scale, where random sampling will be extremely efficient. On the other hand, HBE's will have roughly similar performance on the two extremes as both correspond to worst-case datasets for scale-free estimators and will show a slight improvement in between 1 << D << n. This picture is confirmed by our experiments below.

![D-structured](https://github.com/kexinrong/rehashing/blob/master/plots/clusters2.png)

## Python code

The routine to generate such instances and the example visualizations can be found in [demo/kde_instances.py](https://github.com/stanford-futuredata/hbe/blob/clean/demo/kde_instance.py).

# Experiments
We test kernel evaluation algorithms (RS, HBE, ASKIT, FIGTree) on synthetic instances. We consider two type of instances:
- *"worst-case":* we take the union of two datasets generated for D = 10, n = 50K and D=5K, n=100 while keeping s = 4, \mu = 0.001 fixed and varying d in [10,500].
- *D-structured:* we set N = 500K, s = 4, \mu = 0.001, d = 100 and vary D while keeping n * D = N.


![synthetic instances](https://github.com/kexinrong/rehashing/blob/master/plots/synthetic.png)

We evaluate the four algorithms for kernel density estimation on worst-case instances with d  and on D-structured instances with D in [1,100K]. 
- For "worst-case" instances, while all methods experience increased query time with increased dimension, HBE consistently outperforms other methods for dimensions ranging from 10 to 500 (left). 
- For the D-structured instances (right), FigTree and RS dominate the two extremes where the dataset is highly "clustered"  D >> n or "scattered" on a single scale D >> n,  while HBE achieves the best performance for datasets in between. ASKIT's runtime is relatively unaffected by the number of clusters but is outperformed by other methods. 