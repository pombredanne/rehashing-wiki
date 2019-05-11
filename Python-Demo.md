# Introduction

This page describes how to use the python demo [(demo/rehashing.py)](https://github.com/kexinrong/rehashing/blob/master/demo/rehashing.py) to
- Investigate how many samples on average are required to answer random queries from the dataset to a desired relative error.
- Tune the parameters of eLSH
- Visualize the local structure of queries from the dataset to gain intuition about the distance scales that contribute to the query density
- Prepare the config file to use the efficient C++ implementation

# Assumptions

We assume that:
- the dataset in question is stored in a CSV file with each row corresponding to a vector (point).
- the kernel k(x,y)  is a decreasing function of the Euclidean distance ||x-y|| and the bandwidth \sigma is known. 
- the desired relative accuracy \epsilon in (0,1) is known
- a threshold \tau \in (0,1)  that defines a lower bound on the densities for which we want to approximate with relative error at most \epsilon.

## Attention
- Bandwidth selection for Kernel density estimation or kernel regression is still a topic of research and there are different approaches. We do not discuss here how to choose the bandwidth.
- If the dataset is large (e.g. > 500K rows) we recommend running the demo on a smaller CSV file, by sampling a certain number N = 18 / (\epsilon ^ 2 * \tau) of random rows (points).
- Recommended values for kernel density estimation is \epsilon = 0.25, \tau = 0.1 / \sqrt{n}. 

# Step 1: Problem Specification

The first step is to load the dataset and specify the kernel evaluation problem by instantiating:
- weights to be used w_i for i in [n]
- the kernel function k(x,y)
- the bandwidth \sigma
- the threshold \tau
- the target relative error \epsilon

In this demo we are going to apply our method to the [covertype](https://archive.ics.uci.edu/ml/datasets/Covertype) UCI dataset.
```python
# example dataset
points = np.loadtxt(open("datasets/covtype.data", "rb"), delimiter=",", skiprows=0)
points = points.transpose()
(d,n) = points.shape
# specify kernel evaluation paramaters
weights = np.ones(n) / n # in general weights can be non-negative and sum to 1
sigma = np.sqrt(np.mean(np.mean(points**2))) / 5 # example bandwidth
# exponential kernel
kernel_fun = lambda x,y: np.exp(-np.linalg.norm(x-y) / sigma)
# lower bound of densities of interest suggested value is 0.1 / sqrt{n}
tau = 0.0001
# target multiplicative accuracy
eps = 0.2
```

# Step 2: Initialize the data-structure and Sketch the data
```python
    # Initialize data structure
    covtype = rehashing(points, weights, kernel_fun)
    # Create sketch
    covtype.create_sketch(method='random', accuracy=eps, threshold=tau,\
                  num_of_means=1)
    # Set parameters for hashing through eLSH
    hash_probs = []
```

# Step 3: define the hashing schemes under consideration

The hashing scheme with label 0, corresponds to the setting of parameters that theory suggests. Schemes with label 1 and 2, correspond to using hashing schemes where the *power* (number of hash functions concatenated to produce a single sample) and hence the evaluation time is reduced by a factor of 5 and 10 respectively. 

```python
    R = np.log(1/eps/tau) # effective diameter for exponential kernel

    # For illustration we will apply our method for 2 hashing schemes
    # but one can extend this to an arbitrary number of hashing schemes

    # Hashing scheme #0
    kappa_0 = int(np.ceil(np.sqrt(2*np.pi)*R*np.log(1/tau))) # ``power"
    w0 = np.sqrt(2 / np.pi) * 2 * kappa_0 # set hash bucket width
    p0 = partial(eLSH_prob, w=w0*sigma, k=kappa_0) # collision probability
    hash_probs.append(p0) # add to list

    # Hashing scheme #1
    kappa_1 = kappa_0 / 5 # ``power"
    w1 = np.sqrt(2 / np.pi) * 2 * kappa_1 # set hash bucket width
    p1 = partial(eLSH_prob, w=w1*sigma, k=kappa_1)  # collision probability
    hash_probs.append(p1) # add to list
    
    # Hashing scheme #2
    kappa_2 = kappa_0 / 10 # ``power"
    w2 = np.sqrt(2 / np.pi) * 2 * kappa_1 # set hash bucket width
    p2 = partial(eLSH_prob, w=w2*sigma, k=kappa_2)  # collision probability
    hash_probs.append(p2) # add to list
```
# Step 4: Run the diagnostic procedure to estimate sample complexity and visualize dataset
```python
    # Run diagnostic and visualization
    covtype.diagnostic(hash_probs, acc=eps, threshold=tau, \
                   num_queries=100, visualization_flag=True)
```
In this setting, we observe that the hashing scheme with label 1 has lower estimated relative variance and will be faster to evaluate. All hashing schemes have lower estimated relative variance (sample complexity) than uniform random sampling.

![Diagnostic](https://github.com/kexinrong/rehashing/blob/master/plots/diagnostic.png "Diagnostic")

We can also use the diagnostic procedure to produce a visualization of the local query structure of the dataset. For more information see the [Visualizations](https://github.com/kexinrong/rehashing/wiki/Visualizations) page.

![Visualization](https://github.com/kexinrong/rehashing/blob/master/plots/covertype.png "Visualization")


# Step 5: use the best parameters and create the C++ config file 

After running our diagnostic procedure we have identified a good setting of parameters for the hashing scheme that we can use to create a config file.
