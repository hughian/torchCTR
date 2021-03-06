##
Torch-CTR

## feature info
**SparseFeature** 稀疏特征，通常是无序的类别特征，然后进行 One-Hot Encoding，存储成稀疏形式。使用 Embedding 嵌入到低纬稠密向量。

```
SparseFeature(name, dim, embedding_dim, hashing, dtype='int32') 
```

**DenseFeature** 稠密特征，连续值，或者有序的特征值

```
DenseFeature(name, dim, embedding_dim, dtype='float32')
```

**VarlenSparseFeature** 变长的稀疏特征，比如用户的历史行为向量。`max_len` 来描述这个特征可能出现的最长的长度。这个特征的嵌入是这个 list 中每一个元素 embedding_lookup 之后的聚合(sum, mean, max). 

```
VarlenSparseFeature(name, max_len, dim, embedding_dim, hashing, dtype='int32')
```

**关于 feture_metas** 需要提供:
- add sparse feature
- add dense feature
- add varlen feature
- \_\_len\_\_
- iterable(name, feat)


## Input
torch 没有单独的占位符，也不能像 Keras 那样提供一个 Input() 层(eager mode 下 Input() 是怎么实现的？)
所以输入直接输入处理成字典，keras 的 `model.fit()` 支持直接输入一个字典数据，
会按照 name 将字典的内容 feed 给 Input(). 

Torch 应该怎么做？

model 输入必须为 Tensor, 现在的解决方案，将输入分成三组，写一个 Dataset 子类 + DataLoader?
使用一个统一的 Tensor，embedding 的时候使用 .long() ？Tensordataset 效率更高？
先读一读 keras.fit 的代码，然后再考虑。



## Embedding
需要按名 Embedding, 所以需要使用 nn.ModuleDict
```python
import torch.nn as nn
features = ()
nn.ModuleDict({
    feature.name: nn.Embedding(feature.dim, embedding_dim=8) for feature in features
})
```

build model 的时候输入 Feature 信息，然后按照名字和类型 build Embedding.

* 对于 Sparse Feature, 直接使用 Embedding
* 对于 List Sparse Feature，使用 EmbeddingBag, mode='sum'/'mean', 也可以只使用 Embedding, 然后使用另外的方式来做 Pooling
* 对于 Dense Feature, 使用 Dense 变到同一维度, 或者直接 concat 到下一层


## MLP
这就是个简单的 DNN, 不同之处是使用 BN, Dropout 之类的，然后就是改用不同激活函数，以及层数。


## Sigmoid(Prediction)
对于二分类任务的话，当然是使用 Sigmoid, 对于多分类任务，要使用 softmax。

回归任务： MSE/MAE 这些评价指标.


# TODO
- [ ] embedding_dimme
- [ ] group_embedding
- [ ] L1, L2 Regularization 
- [ ] more models
