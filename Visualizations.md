# Introduction

Often besides estimating the kernel density of a query point, we would also like to get a sense of its place in the dataset. *Is the query density a result of the contributions of a lot of far away points or a few very close points or both?* In the first case, Random Sampling is expected to perform well, whereas in the second HBE and Nearest-neighbor type heuristics are expected to perform well. The third case is the hardest for all kernel evaluation methods.

A way to answer this question is for a given query to find two numbers \lambda_eps(q) <= \Lambda_eps(q) such that contributions of points x in the dataset with \lambda_eps(q) <= k(x,q) <= \Lambda_eps(q) are at least (1- \eps) fraction of the query density. Hence, a tuple of numbers (\lambda_eps(q), \Lambda_eps(q)) represent the local structure of the dataset around query q.

For a given set of points S, we can bound the relative variance (Variance divided by expectation squared) of the uniform random sampling estimator for the kernel density by the ratio max_{x,y in S}[k(q,y) / k(q, x)]. This ratio is often called the *condition number* of the set S. 

Given two numbers (\lambda_eps(q), \Lambda_eps(q)) and S_eps the set of points with weights between these two numbers, we can bound the condition number of S_eps (that contains most of the query density) by the ratio [\Lambda_eps / \lambda_eps]. We are going to use this fact to produce intuitive plots of the query density.

# [Log-Condition Plot](https://github.com/kexinrong/rehashing/blob/master/demo/rehashing.py#L501-L541)

Define R(q) = - log( \lambda_eps(q) ) and r(q) = - log( \Lambda_eps(q) ). We see that the relative variance of uniform random sampling is bounded by e^(R(q) - r(q)). Thus, if we plot a spherical annulus with radii r(q) <= R(q) around 0 (representing the query), the width of the annulus corresponds to the logarithm of the condition number. In the specific case of the Laplace (exponential) kernel, the radii we are plotting correspond to actual distances.


To produce a visualization of a dataset:
- we sample a number of random queries
- for each query we compute (\lambda_eps(q), \Lambda_eps(q))
- and plot a transparent spherical annulus around 0 with radii R(q), r(q).
- such that the density (color intensity) of each annulus at a certain "distance" represents the fraction of queries that have contribution from such "distance".

Below we give an example of such a visualization for the [covertype](https://archive.ics.uci.edu/ml/datasets/Covertype) UCI dataset.

![Covertype dataset Visualization](https://github.com/kexinrong/rehashing/blob/master/plots/covertype.png "Covertype dataset Visualization")

# Example Visualizations

Additional visualizations of datasets. In the top row, we provide visualizations of [synthetic benchmarks](https://github.com/kexinrong/rehashing/wiki/Synthetic-benchmarks) generated to be highly clustered (left, D=1, s>1) and highly
scattered (right, D=100K, s=1). RS performs better for all
datasets in the right column expect for SVHN.

![Query Visualizations](https://github.com/kexinrong/rehashing/blob/master/plots/viz_all.png "Query Visualizations")

We observe two distinctive types of visualizations.  Datasets like census exhibit dense inner circles, meaning that a small number of points close to the query contribute significantly towards the density. To estimate the density accurately, one must sample from these small clusters, which HBE does better than RS. In contrast, datasets like MSD exhibit more weight on the outer circles, meaning that a large number of "far" points is the main source of density. Random sampling has a good chance of seeing these "far" points, and therefore, tends to perform better on such datasets. The top two plots amplify these observations on synthetic datasets with highly clustered/scattered structures. RS performs better for all datasets in the right column except for SVHN. 
