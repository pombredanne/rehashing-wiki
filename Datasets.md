All datasets used in our experiments could be found on [LIBSVM](https://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/regression.html) and the [UCI](https://archive.ics.uci.edu/ml/datasets.php) machine learning repository. For detailed descriptions and bandwidth used, please refer to the table below: 

![](https://github.com/kexinrong/rehashing/blob/master/plots/dataset.png)

We include two of the datasets [covtype](https://archive.ics.uci.edu/ml/datasets/covertype) and [shuttle](https://archive.ics.uci.edu/ml/datasets/Statlog+(Shuttle)) as examples in the Github [repository](https://github.com/kexinrong/rehashing/tree/master/resources/data).

By default, we **z-normalize** each dimension in the dataset as a preprocessing step. Without normalization, HBE might suffer from performance degradation, since the algorithm determines the hashing scheme based on the effective "diameter" of the dataset. 
