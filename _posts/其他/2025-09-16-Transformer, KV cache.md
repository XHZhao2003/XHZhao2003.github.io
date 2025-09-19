---
layout: post
title: "Transformer, KV cache"
category: "其他"
order: "5"
math: true
---


Transformer 是一种神经网络架构，应用于众多 LLM ；而 KV cache 是加速 Transformer 模型推理的基础技术。本文借助一个 LLM 生成文本的实例，来解释基于 Transformer 的模型和 KV cache 的工作方式。

我们可以向 DeepSeek、Qwen 等模型输入如下提示词：“用一句话来解释 Transformer”

得到的一个回答是：“Transformer 是一种基于自注意力机制的深度学习模型，能够并行处理序列数据并捕捉长距离依赖关系，彻底改变了自然语言处理等领域。”

**分词**

我们的输入 $I = \text{用一句话来解释 Transformer} $ 需要被数字化。

首先，LLM 会应用它的词表（vocabulary）来识别 $I$ 中出现的 token（词元）。词表中通常具有上万个 token，保证覆盖通常语言的全部词汇。例如，这句话中包含的 token 可能是 

```
['用','一'，‘句’，‘话’，‘来’，‘解释’，‘Trans’，‘for’，‘mer’]
```

这是一个虚构的例子，具体的分词情况与之类似。汉字的单字、词语，英文的单词、亚单词单元都可能是 token。词表中包含哪些 token，取决于 LLM 的训练语料与采用的分词算法。词表中的每个 token 都具有唯一的 id，我们假定上述 token 序列的 id 序列是

$$
IDs = [100, 200, 300, 400, 500, 1000, 10000, 10100, 10200]
$$

**词嵌入**

token id 本身只是一个索引，真正实现将输入数字化的过程是词嵌入。LLM 具有一个词嵌入矩阵 $\text{word embedding}$，每一个索引 id 都对应这个矩阵中的一行，即一个长为 $d_{\text{word embedding}}$ 的向量。词嵌入矩阵是随机初始化，并在 LLM 的预训练阶段学习得到的。$d_{\text{word embedding}}$ 的典型值包括 $768, 1024, 1280$ 等。这样，token id 就转换成嵌入矩阵
$$
embedding = [\text{word embedding}[ID] \text{ for ID in IDs}]
$$
**位置嵌入**

位置嵌入是为了建模 token 在序列中的顺序而采用的一种嵌入向量。通常，它的长度与词嵌入相同，并与词嵌入相加，得到融合位置信息的嵌入，作为 Transformer 模块的输入。

最简单的位置嵌入是可学习的，也就是随机初始化并在预训练中学习，所以它在形式上就是另一种词嵌入。

Transformer 论文中使用了正弦/余弦位置编码（Sinusoidal）。其计算公式为
$$
\text{position embedding}(pos, 2i) = \sin(\frac{pos}{10000^{\frac{2i}{d_{\text{word embedding}}}}}) \\
\text{position embedding}(pos, 2i+1) = \cos(\frac{pos}{10000^{\frac{2i}{d_{\text{word embedding}}}}})
$$
其中 $\text{position embedding}(pos, i)$ 表示位置 pos 的位置嵌入的第 $i$ 维度的值。

这种位置编码是支持外推的，这是相对于可学习的位置编码而言的。可学习位置编码是为每一个 pos 学习一个嵌入，自然会具有一个最大 pos，也就是能够处理的最大序列长度。超过这一长度，就没有可用的位置编码了。而正弦/余弦位置编码不具有这一限制，对于任何 pos 值代入公式即可计算位置嵌入。

这种位置编码还支持相对位置变换。也就是，对任意固定偏移量 $k$，$ \text{position embedding}(pos + k)$ 可表示为 $\text{position embedding}(pos)$ 的线性函数 —— 这有助于模型学习相对位置关系。

词嵌入与位置嵌入相加，就可以作为 Transformer 网络的输入了：
$$
E = \text{embedding} + \text{position embedding}
$$
$E \in \mathbb{R}^{L\times d_{\text{word embedding}}}$，$L$  即为输入序列的长度，上述例子中 $L=9$

**Encoder: Self Attention**

词嵌入 $E$ 首先会经过 self-attention 层，这一层具有三类参数 $W_Q, W_K, W_V$，其形状通常是相同的，即 $W_{\{Q, K, V\}} \in \mathbb{R}^{d_{\text{model}}\times d_{\text{model}}}$。$d_{\text{model}}$ 与 $d_{\text{word embedding}}$ 通常是相同的。否则，就需要在这一步之前加一个投影矩阵，进行维度转换。这三类参数会与 $E$ 相乘，得到每个 token 的 $Q, K, V$ 向量形成的矩阵：
$$
\{Q, K, V\} = W_{\{Q, K, V\}}\times E
$$
$Q, K, V \in \mathbb{R}^{L\times d_{\text{model}}}$。$W_{\{Q, K, V\}}$ 也是随机初始化，然后在预训练阶段学习的参数。

设计者为 $Q, K, V$ 赋予了含义。为了理解整个序列，每一个 token 都需要关注序列中的某些其他 token。回顾我们的输入 $I = \text{用一句话来解释 Transformer}$ ，'解释' 这个 token 显然就需要多关注一下 'Transformer'相关的词元，以及 '一句话'相关的词元。

为了建模这种现象，每个 token 都具有一个 query 向量、key 向量和 value 向量。为了计算一个 token 应该关注什么 token，就用它的 query 去 “查询” 其他所有 token 的 key，也就是计算二者的内积。内积的结果代表了注意力，结果大的表示应该多关注，小的就表示应该少关注。归一化后结果，就是这个 token 对于其他所有 token 的 “注意力”。然后，这个 token 的嵌入，就会被替换为
