---
title: "[Concept] Positional encoding"
date: 2025-11-15
tags:
 - Positional Encoding
 - Transformers
draft: false
---

Transformer models process inputs in parallel without inherent awareness of their sequence order.
Positional encoding provides cues that help capture the relationships and dependencies based on position.

The key idea is that each position in the sequence is assigned a unique vector representation, which is added to the token embeddings before feeding them into the transformer layers. This way, the model "knows" which token comes first, second, and so forth, enabling it to model the structure and context effectively. 

If positional encoding is omitted: No way to distinguish "dog bites man" from "man bites dog"


## Sinusoidal Positional Encoding
Sinusoidal positional encoding is the original technique introduced in "Attention Is All You Need" (2017) to inject position information into transformers. For every position in your sequence, a vector is generated using sine and cosine functions at different frequencies across the embedding dimensions.

For each position \(pos\) and each embedding dimension \(i\):

$$
PE_{(pos,2i)} = \sin\left(\frac{pos}{10000^{2i/d_{model}}}\right),\quad PE_{(pos,2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$

Where \(d_{model}\) is the embedding dimensionality, controlling the variation frequency in each part of the vector.

This postition are added to the input such that all the Q,K,V values metrices are changed:

\( q_m = W_q (x_m + p_m) \)

\( k_n = W_k (x_n + p_n) \)

\( v_n = W_v (x_n + p_n) \)

Notice that lower dimensions in the positional vectors change slowly over positions, encoding broad trends. Higher ones oscillate faster, encoding finer differences.
 ![sin_pos_embedding_curve](/posts/20251115_postinal_encoding/sin_pos_embedding_curve.png)


However, 
1. it struggles with generalizing to significantly longer sequences beyond training limits, causing positional ambiguity due to its cyclical nature. 
2. These encodings are not adaptive to data patterns, limiting effectiveness in tasks with irregular or complex positional dependencies. 
3. Additionally, fixed sinusoidal encodings cannot dynamically adjust to position shifts or varying sequence lengths during inference. 
These limitations have motivated development of learned and relative positional encoding methods for improved flexibility and performance.


```python
import torch
import torch.nn as nn

def sinusoidal_positional_encoding(max_len: int, d_model: int, device=None):
    position = torch.arange(max_len, dtype=torch.float32, device=device).unsqueeze(1)
    div_term = torch.exp(torch.arange(0, d_model, 2, dtype=torch.float32, device=device) *
             -(torch.log(torch.tensor(10000.0)) / d_model))
    pe = torch.zeros(max_len, d_model, device=device)
    pe[:, 0::2] = torch.sin(position * div_term)
    pe[:, 1::2] = torch.cos(position * div_term)
    return pe  # shape: (max_len, d_model)
  ```

## Learnable Positional Encodings
Learnable positional encodings are vectors whose values are initialized randomly and refined through training alongside other model parameters. This allows the model to tailor positional representations specifically to the task and data distribution, potentially learning more effective and nuanced positional signals than fixed encodings. As a result, learnable encodings can handle complex, irregular positional patterns better and may boost performance on tasks where absolute position matters strongly


However 
1. they require additional parameters and training data to generalize well and may overfit if the dataset is small or not diverse. 
2. Moreover, since the encoding is learned only within the training context, learnable encodings usually lack the broad extrapolation capability classic sinusoidal encodings provide.

```python
class LearnedPositionalEmbedding(nn.Module):
  def __init__(self, max_len: int, d_model: int):
    super().__init__()
    self.pos_embed = nn.Embedding(max_len, d_model)

  def forward(self, x):
    # x: (batch, seq_len, d_model) -- we only need seq_len
    seq_len = x.size(1)
    positions = torch.arange(seq_len, device=x.device).unsqueeze(0)
    return self.pos_embed(positions)  # (1, seq_len, d_model)
```

## Relative positional encoding
Encode relative distances between query and key positions rather than absolute positions. This often improves generalization for tasks where relative order matters more than absolute position (e.g., language modeling, local patterns). Examples include Shaw et al.'s relative biases and Transformer-XL's segment-aware mechanisms.

Relative positional information can be supplied to the model in the keys only or keys and values

Given input embeddings \(X \in \mathbb{R}^{L \times d}\) where \(L\) is sequence length and \(d\) is embedding size, define:

- Queries:  \(Q = X W^Q\)  
- Keys:  \(K = X W^K\)  
- Values:  \(V = X W^V\)  

Here  \(W^Q, W^K, W^V \in \mathbb{R}^{d \times d_z}\) are learnable linear projections, with  \(d_z\) the attention head dimension.

Relative positional encodings introduce learned tensors:

-  \(A^K \in \mathbb{R}^{L \times L \times d_z}\) added to keys  
-  \(A^V \in \mathbb{R}^{L \times L \times d_z}\) added to values  

The output of the attention layer is then 
$$
\text{Softmax}(\frac{Q (K + A^K)^\top}{\sqrt{d_z}}) (V + A^V)
$$


```python
class RelativePositionBias(nn.Module):
  def __init__(self, max_distance: int, num_heads: int):
    super().__init__()
    self.max_distance = max_distance
    self.num_heads = num_heads
    # distances from -max_distance..+max_distance mapped to indices
    self.bias_table = nn.Embedding(2 * max_distance + 1, num_heads)

  def forward(self, seq_len: int):
    # compute pairwise relative distances
    idxs = torch.arange(seq_len, device=self.bias_table.weight.device)
    rel = idxs[None, :] - idxs[:, None]  # (seq_len, seq_len)
    clipped = rel.clamp(-self.max_distance, self.max_distance) + self.max_distance
    # lookup and return shape (1, num_heads, seq_len, seq_len) to add to logits
    bias = self.bias_table(clipped)  # (seq_len, seq_len, num_heads)
    bias = bias.permute(2, 0, 1).unsqueeze(0)
    return bias
```

## Rotary Positional Embeddings (RoPE)

Rotary Positional Embeddings (RoPE) encode positional information in transformer models by rotating token embeddings in pairs of dimensions, with rotation angles proportional to the absolute token position.

Imagine each token’s embedding vector split into pairs of dimensions. RoPE applies a 2D rotation to each pair, where the rotation angle increases with the token’s position along the sequence. This means each token’s embedding is uniquely rotated according to its absolute position, embedding position information inherently into the vector. When attention computes dot products between rotated query and key embeddings, the result depends on the relative difference in rotation angles—effectively encoding relative positions while preserving absolute positional information in the embeddings.

RoPE applies to \(Q\) and \(K\) as follows for token position \(m \in [0, L-1]\) and dimension pair \((2i, 2i+1)$$:

$$
\hat{q}_m^{(2i)} = q_m^{(2i)} \cos(m \theta_i) - q_m^{(2i+1)} \sin(m \theta_i)
$$

$$
\hat{q}_m^{(2i+1)} = q_m^{(2i)} \sin(m \theta_i) + q_m^{(2i+1)} \cos(m \theta_i)
$$

$$
\hat{k}_m^{(2i)} = k_m^{(2i)} \cos(m \theta_i) - k_m^{(2i+1)} \sin(m \theta_i)
$$

$$
\hat{k}_m^{(2i+1)} = k_m^{(2i)} \sin(m \theta_i) + k_m^{(2i+1)} \cos(m \theta_i)
$$

with frequency term

$$
\theta_i = 10000^{-\frac{2i}{d_z}}.
$$

The transformer attention is then computed with these rotated embeddings:

$$
\text{Attention}(Q, K, V) = \text{Softmax}\left(\frac{\hat{Q} \hat{K}^\top}{\sqrt{d_z}}\right) V.
$$

Due to properties of rotation matrices:

$$
\hat{q}_m^\top \hat{k}_n = q_m^\top R_{n-m} k_n,
$$

where \(R_{n-m}\) is a rotation matrix corresponding to the relative position \(n - m$$.



![rope](/posts/20251115_postinal_encoding/rope.png)

```python
def apply_rope(x):
    batch_size, seq_len, n_heads, head_dim = x.shape
    assert head_dim % 2 == 0, "head_dim must be even for paired rotation"

    # Compute rotary frequencies
    dim = head_dim // 2
    inv_freq = 1.0 / (10000 ** (torch.arange(0, dim, 1).float() / dim))
    positions = torch.arange(seq_len, device=x.device).float()

    # Create position angles (seq_len x dim)
    angle_rates = torch.einsum('i,j->ij', positions, inv_freq) 

    # Compute sin and cos (seq_len, dim) ->  (seq_len, 1, dim)
    sin_angles = angle_rates.sin()[:, None, :]
    cos_angles = angle_rates.cos()[:, None, :]

    # Split x into two halves: even and odd
    x1, x2 = x[..., :dim], x[..., dim:]

    # Apply rotary embedding formula (rotation in 2D space)
    x_rotated_first = x1 * cos_angles - x2 * sin_angles
    x_rotated_second = x1 * sin_angles + x2 * cos_angles

    # Concatenate rotated halves back
    x_rotated = torch.cat([x_rotated_first, x_rotated_second], dim=-1)
    return x_rotated
```


## Comparing the Methods

**Practical considerations*.*

- **Add vs. concatenate:** Positional encodings are usually added to token embeddings, but concatenation or separate bias terms are also used in some architectures.
- **Max length / extrapolation:** Fixed learned embeddings have a hard max length; sinusoidal and some relative methods extrapolate better.
- **Normalization & scale:** When adding PE to token embeddings, treat scale carefully — some implementations scale embeddings or use layer norm to stabilize training.
- **Computational cost:** Relative methods sometimes add small computational overhead (extra bias lookup or specialized kernels) but often yield better performance.

1. Results / Where it helps

- Improves sequence modeling performance across NLP and multimodal tasks where order matters.
- Relative encodings and RoPE often yield better transfer and longer-context generalization than naive learned absolute embeddings for many language-modeling benchmarks.
- Positional information is essential for transformers; choice of encoding impacts generalization and max context handling.
- Use sinusoidal or RoPE when you need length extrapolation; use learned embeddings when training length is fixed and simplicity is preferred.
- Consider relative encodings for problems dominated by local relative structure.

# Resource

[Attention Is All You Need](https://arxiv.org/abs/1706.03762)

[How Rotary Position Embedding Supercharges Modern LLMs [RoPE]](https://www.youtube.com/watch?v=SMBkImDWOyQ)


[Inside Sinusoidal Position Embeddings: A Sense of Order](https://learnopencv.com/sinusoidal-position-embeddings/)


[Relative Positional Encoding](https://jaketae.github.io/study/relative-positional-encoding/)


[A gentle introduction to Rotary Position Embedding](https://krasserm.github.io/2022/12/13/rotary-position-embedding/)


- Vaswani et al., "Attention Is All You Need" (2017) — sinusoidal encodings.
- Shaw et al., "Self-Attention with Relative Position Representations" (2018).
- Dai et al., "Transformer-XL" (2019) — segment-level recurrence and relative biasing.
- Su et al., "RoFormer / RoPE" (2021) — rotary positional embeddings.