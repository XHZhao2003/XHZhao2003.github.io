---
layout: post
title: "Transformer, KV cache"
category: "其他"
order: "5"
math: true
---


Transformer 是一种神经网络架构，应用于众多 LLM。

最初提出 Transformer 架构的论文 “Attention is all you need” 使用了 encoder - decoder 架构，但现在主流的 LLM 基本都采取了 decoder only 的架构，并且广泛使用 KV Cache 来进行推理加速。本文借助一个 LLM 生成文本的实例，来解释 encoder - decoder 以及 decoder only 的计算方式，以及 KV cache 的工作方式。

我们可以向 DeepSeek、Qwen 等模型输入如下提示词：“用一句话来解释 Transformer”

得到的一个回答是：“Transformer 是一种基于自注意力机制的深度学习模型，能够并行处理序列数据并捕捉长距离依赖关系，彻底改变了自然语言处理等领域。”



### Encoder Decoder 架构：分词

我们的输入 $I = \text{用一句话来解释 Transformer} $ 需要被数字化。

首先，LLM 会应用它的词表（vocabulary）来识别 $I$ 中出现的 token（词元）。词表中通常具有上万个 token，保证覆盖通常语言的全部词汇。例如，这句话中包含的 token 可能是 

```
['用','一'，‘句’，‘话’，‘来’，‘解释’，‘Trans’，‘for’，‘mer’]
```

这是一个虚构的例子，具体的分词情况与之类似。汉字的单字、词语，英文的单词、亚单词单元都可能是 token。词表中包含哪些 token，取决于 LLM 的训练语料与采用的分词算法。词表中的每个 token 都具有唯一的 id，我们假定上述 token 序列的 id 序列是

$$
IDs = [100, 200, 300, 400, 500, 1000, 10000, 10100, 10200]
$$

### 词嵌入

token id 本身只是一个索引，真正实现将输入数字化的过程是词嵌入。LLM 具有一个词嵌入矩阵 $W_E$，每一个索引 id 都对应这个矩阵中的一行，即一个长为 $d_E$ 的向量。词嵌入矩阵是随机初始化，并在 LLM 的预训练阶段学习得到的。$d_E$ 的典型值包括 $768, 1024, 1280$ 等。这样，token id 就转换成嵌入矩阵
$$
E = [W_E[id] \text{  for  } id \text{  in  } IDs]
$$


### 位置嵌入

位置嵌入是为了建模 token 在序列中的顺序而采用的一种嵌入向量。通常，它的长度与词嵌入相同，并与词嵌入相加，得到融合位置信息的嵌入，作为 Transformer 模块的输入。

最简单的位置嵌入是可学习的，也就是随机初始化并在预训练中学习，所以它在形式上就是另一种词嵌入。

Transformer 论文中使用了正弦/余弦位置编码（Sinusoidal）。其计算公式为

$$
PE(pos, 2i) = \sin(pos / 10000^{2i / d_E})
$$

$$
PE(pos, 2i+1) = \cos(pos / 10000^{2i / d_E})
$$

其中 $PE(pos, i)$ 表示位置 pos 的位置嵌入的第 $i$ 维度的值。

这种位置编码支持外推，这是相对于可学习的位置编码而言的。可学习位置编码是指，为每一个 pos 学习一个嵌入，那么自然会有一个最大 pos，也就是能够处理的最大序列长度。超过这一长度，就没有可用的位置编码了。而正弦/余弦位置编码不具有这一限制，对于任何 pos 值代入公式即可计算位置嵌入。

这种位置编码还支持相对位置变换。也就是，对任意固定偏移量 $k$，$ PE(pos + k)$ 可表示为 $PE(pos)$ 的线性函数。这有助于模型学习相对位置关系。

词嵌入与位置嵌入相加，就可以作为 Transformer 网络的输入了：

$$
E + PE \to E
$$

$E \in \mathbb{R}^{L \times d_E}$，$L$  即为输入序列的长度，上述例子中 $L=9$

### Encoder: Self Attention

词嵌入 $E$ 首先会经过 self-attention 层，这一层具有三类参数 $W_Q, W_K, W_V$，其形状通常是相同的，即 $W_{\{Q, K, V\}} \in \mathbb{R}^{d \times d}$。$d$ 表示内部的嵌入维度，与 $d_E$ 通常是相同的。否则，就需要在这一步之前加一个投影矩阵，进行维度转换。这三类参数会与 $E$ 相乘，得到每个 token 的 $Q, K, V$ 向量形成的矩阵：

$$
\{Q, K, V\} = E \times W_{\{Q, K, V\}}
$$

$Q, K, V \in \mathbb{R}^{L \times d}$。$W_{\{Q, K, V\}}$ 也是随机初始化，然后在预训练阶段学习的参数。

设计者为 $Q, K, V$ 赋予了含义：为了理解整个序列，每一个 token 都需要关注序列中的某些其他 token。回顾我们的输入：

$$
I = \text{用一句话来解释 Transformer}
$$ 

'解释' 这个 token 显然需要多关注一下 'Transformer' 的词元。为了建模这种现象，每个 token 都具有一个 query 向量、key 向量和 value 向量，也就是 $q, k, v$。为了计算一个 token 应该关注什么 token，就用它的 $q$ 向量 去 “查询” 其他所有 token 的 $k^\prime$ 向量，也就是计算二者的内积 $q^{T}k^\prime$。内积的结果代表了注意力，注意力值越大表示越应该关注这个 token。

用这种方式，我们需要计算每个 token 对每个 token 的注意力。例如，'解释' 这个 token 对其他所有 token 的注意力，经过 softmax 归一化之后，可能是 [0.01, 0.01, 0.01, 0.01, 0.01, 0.1, 0.3, 0.3, 0.25]。然后，我们会用这一组权重，对每个 token 的 $v$ 向量（Value）进行加权求和，作为 '解释' 这个 token 的语义表示。

这整个过程就是：

$$
\text{softmax}(\frac{Q^T K}{\sqrt{d}})V \to E^\prime
$$

其中，$Q^TK$ 的结果在进行 Softmax 归一化之前除以了 $\sqrt{d}$，其目的是稳定归一化的效果。其理论依据是，假设 $Q, K$ 中的每个元素都独立服从标准正态分布。则任意一组 $q_i^Tk_j$ 的期望为 0，方差为 $d$。将其值除以 $\sqrt{d}$ 就使结果中的每个值仍然服从标准正态分布。这有助于避免 Softmax 的输入中出现极端大的值，影响归一化结果。

上述过程就是自注意力机制（Self Attention），以 token 序列的嵌入 $E$ 作为输入，输出一个同样维度的矩阵 $E^\prime$，区别在于后者进一步捕捉了序列中 token 之间的语义关系。

### Encoder: Multihead Attention

多头自注意力机制基于上述过程进行改进。假设 $d=512$，那么有 $W_{Q, K, V} \in \mathbb{R}^{512 \times 512}$，$Q, K, V \in \mathbb{R}^{512 \times 9}$。我们希望在不提升参数量的情况下，增加模型的表示能力。我们将 $W_{Q, K, V}$ 的尺寸缩小 8 倍，也就是缩小为 $\mathbb{R}^{512 \times 64}$，这样得到的 $Q, K, V$ 尺寸就是 $\mathbb{R}^{64 \times 9}$。但同时，我们会设置 $h=8$ 个这样的副本。计算的方式变成：

$$
Q^i, K^i, V^i = E \times W_{Q, K, V}^i \quad i \in [1, 8]
$$

$$
\text{Concat}(
    \text{softmax}(\frac{{Q^i}^T K^i}{\sqrt{d/8}})V^i
) \times W_O
\to E^\prime
$$

这个过程也就是计算 8 个结构相同，但尺寸较小的副本，然后将结果拼接回去，得到尺寸与原来相同的结果。注意到拼接回去之后还与一个矩阵 $W_O \in \mathbb{R}^{d \times d}$ 相乘。它的意义是，我们可以更灵活的设置注意力头内部的维度，以及注意力头的个数。如果拼接得到的维度与模型维度不同，就使用一个投影矩阵 $W_O$ 来把形状调整回去。在这个例子中我们不需要它来改变形状。

实际中广泛使用的是多头自注意力机制，而不是单头自注意力机制。

### Encoder: Add & Norm

然后我们会使用深度学习中常用的基础部件来处理得到的嵌入 $E^\prime$。

首先是进行残差连接，也就是：

$$
E + E^\prime \to E
$$

它的主要作用是避免深度学习中的梯度消失问题。在非常深的网络中，要将最终的 Loss 回传到早期参与计算的参数是很困难的，因为整个过程有非常多的乘法，而这些乘数累积起来很可能使回传梯度趋近于 0。使用这样的残差连接，可以梯度回传路径大大缩短。

在语义上，使用这样的残差连接就意味着，多头自注意力机制得到的 $E^\prime$ 并不位于我们期望的特征空间中，而是代表了 $E$ 与我们期望的特征空间中的嵌入的差值。

然后，我们使用层归一化（Layer Norm）。层归一化的目的是将每个 token 对应的嵌入中各个维度的嵌入值，调整至均值为 0，方差为 1。对于某个 token 的嵌入 $e \in \mathbb{R}^{d}$，层归一化的过程为：

$$
\frac{(e - \mu)}{\sqrt{\sigma^2 + \epsilon}} \to e
$$

$$
\mu = \text{mean}(e), \quad \sigma^2 = \text{var}(e)
$$

其中 $\epsilon$ 为极小值，为了防止除 0 错误以及除法值溢出。

Layer Norm 让嵌入向量始终保持良好的分布，有利于模型的训练。不得不提的是，在原始的 Transformer 设计中，Layer Norm 被安排在注意力机制模块之后（Post Norm），但后续的改进模型都把 Layer Norm 放在了注意力机制模块之前（Pre Norm），实践证明后者的效果更好。


### Encoder: FFN + Add & Norm

经历上述操作之后，token 嵌入 $E$ 被送入全连接层，包含两层线性层与一层 ReLU 激活层，引入非线性关系。这其中通常包含更大的隐藏层维度，例如在嵌入维度 $d=512$ 的模型中，隐藏层维度可能是 $d_h = 2048$。也就是，512 维嵌入被投影到 2048 维特征空间，经历非线性变换后再投影回 512 维线性空间。

然后，嵌入会再经历一遍 Add & Layer Norm 的标准操作。

目前位置，词嵌入 $E$ 就经历一个 Encoder block 所进行的完整的操作。(多头自注意力、Add & Norm、FFN、Add & Norm)在 Encode 阶段（或者称 Prefill 阶段），会经历多个 Encoder block，来得到最终的嵌入表达，也就是

$$
\text{Encoder}^n(E) \to E
$$

### Decoder
