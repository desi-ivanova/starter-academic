---
title: "Transformer-based entropy models for neural video compression:"
subtitle: Review of some recent papers

# Summary for listings and search engines
summary: "In this post we will review 'VCT: A Video Compresion Transformer' by Fabian Menzer et al. published at NeurIPS 2022 and briefly discuss similarities and differences with the even more recently published 'MIMT: Masked Image Modeling Transformer for Video Compression' by Xiang et al., ICLR 2023. We provide code references to facilitate understanding."

# Link this post with a project
projects: []

math: true

# Date published
date: "2023-04-08T00:00:00Z"

# Date updated
lastmod: "2023-20-20T00:00:00Z"

# Is this an unpublished draft?
draft: true

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption: # 'Image credit: [**Unsplash**](https://unsplash.com/photos/CpkOjOcXdUY)'
  focal_point: ""
  placement: 2
  preview_only: 1

authors:
- admin

tags:
- Academic
- Machine learning
- Deep learning
- Neural compression

categories:
- ML
---


Repeat the paprs, point to goggle repo, include links

## Brief notes on neural video comression

At a high level, the neural (or learnt) video compression pipeline, based on transform coding [CITE] consists of 3 components:
- Analysis transform $f_a$, which transforms video frames to latent representations
- Entropy model $P$, which is used to *losslessly* store the quantized representations to disk
- Synthesis transform $f_s$, which maps quantized latents back to a video.

#### Notation
- $x_i, i=1,\dots, F$ denotes a the $i$-th frame of a video that consists of a total of $F$ frames
- $y_i = f_a(x_i, c)$ are latent representations; $\hat{y}_i = \lfloor y_i \rceil$ is the quantized latent reprsentation.
  -  Note: the analysis transform $f_a$ may or may not depend on other context variables $c$.
- $\hat{x}_i = f_s(\hat{y}_i, c)$ is the reconstruction of $x_i$
  - Note: the synthesis transform $f_s$ may or may not depend on other context variables $c$.
- Q is the true distribution of the quantized latents $\hat{y}_i \sim Q$ (i.e. what we store to disk), and and $P$ is our model to approximate $Q$.


### The role of an entropy model


## Brief note on transformers

?

## VCT: Main components (with code annotations)

### Image analysis and synthesis transforms
Each frame in a video $x_i, i = 1,\dots, F$ is independently mapped into (resp. from) a quantized latent
representation $y_i$ using standard image analysis (resp. synthesis) transforms.
In its updated version, VCT uses [ELIC transforms](https://arxiv.org/abs/2203.10886).
Concretely, an input frame $x_i$ of shape `[3, I_H, I_W]` is mapped to $y_i$ of shape
`[C, I_H/16, I_W/16]`, where by default $C=192$ and we set `I_H/16=H`, `I_W/16=W`.

- The ELIC transforms are available in `neural/transforms.py`, which also contains
other transforms, e.g. from [Balle et al., 2018](https://arxiv.org/abs/1802.01436).

### Transformer-based entropy model
At the heart of the VCT framework is a powerful transformer-based entropy model.
It consists of both an encoder and a decoder and is thus similar in spirit to the original transformer architectures used in machine translation ([Vaswani et al., 2017](https://arxiv.org/abs/1706.03762)).
The compression of a video frame (or, more precisely, its latent representation) is therefore viewed as a translation task: the previous two latent representations $(y_{i-2}, y_{i-1})$  are translated to the current one $y_i$.

Concretely, the entropy model $P$ predicts the probability mass function (PMF) of the current latent, conditional on  the last two, $P(y_i \mid y_{i-2}, y_{i-1})$. This PMF is then used by a standard entropy coder to *losslessly* compress and store $y_i$.
The better $P$ predicts this distribution, the fewer bits will be required to store $y_i$.

-  The entropy model is implemented in `neural/entropy_model.py`.

### Loss
VCT optimizes a standard rate-distortion (RD) loss with lambda $\lambda >0$ controlling the trade-off:
$$
\mathbb{E}_{x\sim p_x, u\sim p_u} [ -\log p(\tilde{y} + u) + \lambda \text{MSE}(x, \hat{x}) ]
$$
where $p_x$ is the data distribution, $p_u = U[-0.5, 0.5]$, $\tilde{y} = \text{AnalysisTransform}(x)$ is the unquantized latent, and $\hat{x}=\text{SynthesisTransform}(\text{straight-through-round}(\tilde{y}))$.

Since training batches consist of 3 frames, to account for train-inference mismatch, the first frame is weighed lower: with weight=1, whilst the other two frames get a weight of 10.

- We use the same loss, which is defined in the lightning module `model_lightning.py`.

### Tokenization

To obtain 'tokens' for the transformer: the latents are patched into blocks whose size depends on whether the latent has already been transmitted (and therefore acts as an input to the transformer *encoder*) or is the one whose PMF is being predicted (and is an input to the transformer *decoder*).
Suppse we want to transmit $y_i$ using $y_{i-1}, y_{i-2}$:

- the **c**urrent $y_i$ is shape `[C, H, W]` and is patched into *non-overlapping* blocks of size $w_c \times w_c$, where $w_c=4$. Thus we have $N_{c} = H*W /w_c^2=H*W/16$ blocks in total, padding if required to make a full block.
Each block/patch contains $w_c^2=16$ elements (this is the max sequence length of the transformer decoder).
Shape of the patched current latent therefore is `[C, N_c, w_c, w_c]` (which equals `[192, 16, 4, 4]` during training).

- each of the **p**revious latents is also shape  `[C, H, W]`. They are patched into *overlapping* blocks of size  $w_p \times w_p$, $w_p=8$ by default, with stride equal to $w_c$.
This would give us a total of $N_{p} = (H - w_p + w_c) * (W - w_p + w_c) / w_c^2$ blocks.
To ensure that the current latent and previous latents are patched into the same number of blocks, the previous latents are padded symmetrically by adding $(w_p - w_c) / 2$ on each side.
Thus, there are two requirements on $w_p$ and $w_c$: 1) $w_p>w_c$ and 2) $w_p - w_c$ must be divisible by 2.
This gives us $N_{p} = ((H + w_p - w_c) - w_p + w_c) * ((W + w_p - w_c) - w_p + w_c) / w_c^2 = N_c$.
Each block/patch contains $w_p^2=64$ elements (this is the max sequence length of the transformer encoder).
Shape of the patched previous latent therefore is `[C, N_p, w_p, w_p]` (which by default equals `[192, 16, 8, 8]` during training).

#### Independence assumption
The $N_c$ patches are assumed independent. So the transformers will operate independently on the patches, i.e. predicting means and scales for $w_c^2=16$ tokens of dimension $C = 192$ in the current latent patch using $2*w_p^2 = 128$ tokens of the encoded past 2 tokens.

### Transformer encoder:
The encoder takes as an input the flattened blocks of tokens of the past two latents, $y_{i-1}, y_{i-2}$, which are processed first independently by a transformer layer (with learnt positional encoding), then concatenated and processed jointly by another transformer layer (with learnt "temporal" positional encoding).
The resulting representation $z_i := f(y_{i-2, i-1})$, which has shape `[N_p, 2*w_p^2, 4*C]` (which by default equals `[16, 128, 768]` during training), is passed to the transformer decoder and is used to predict the PMF of $y_i$.


### Transformer decoder:

The patched current latent $y_i$ is shifted with a learnt 0 element and embedded into `d_model=4*C` representation space and thus has a shape `[N_c, w_c^2, d_model]`.

At train time:
Training is done in the standard teacher-forcing paradigm using a diagonal mask.
The decoder takes the shifted patched latent and $z_{i}$ as inputs and applies self-attention (latent) and cross-attention (on $z$) to obtain an output `dec_output`, which is passed through `mean` and `scale` linear head layers to obtain the predictions of the means and scales.
The image synthesis transform reuses these outputs of the decoder as features.


At inference time: The decoder takes patched $z_{i}=\lbrace z\_{i}^{(j)}\rbrace \_{j=1}^{N_c}$ as input, and autoregressively computes $p(y^{(j)}\_{i, n+1} \mid z\_{i}^{(j)}, y^{(j)}\_{i, \leq n})$ independently and in parallel for each patch $j=1, \dots, N_c$.
In other words, decoding is done in a standard autoregressive 'left-to-right' way: starting with a learnt initial token $n=0$, the next token $n$ is predicted using $y_{i, \leq n}$ (self-attention) and the output from the encoder $z$ (cross-attention).

------
<!--
#### Relaxed training objective and equivalence to VAEs

Ball&#233 et al. (2017) replace the quantization step **during training** with additive uniform noise on the unit interval centred at $z$, i.e. $\hat{z} = Q(z)$ is substituted by $\tilde{z}$, sampled from a distribution $\tilde{z}|x \sim \mathcal{U}\big(\mathcal{E}\_\theta(x)-1/2, \mathcal{E}\_\theta(x)+1/2\big) =: q(\tilde{z}|x)$. The relaxed training objective can then be written as
$$
\begin{aligned}
L(\theta, \psi, \zeta) = \mathbb{E}\_{p(x)q(\tilde{z}|x)}\big[-\log  p\_\zeta(\tilde{z}) + \lambda \lVert x - \mathcal{D}\_\psi(\tilde{z})\rVert^2\_2 \big].
\end{aligned}
$$

Notice that
$$
\log \mathcal{N} \left(x; \mathcal{D}\_\psi(\tilde{z}), \frac{1}{2\lambda} \right) = - \lambda \lVert x - \mathcal{D}\_\psi(\tilde{z})\rVert^2\_2 + \text{const}, $$
so minimising the loss $L$ above is exactly equivalent to minimising
$$
\begin{aligned}
L(\theta, \psi, \zeta) = \mathbb{E}\_{p(x)q(\tilde{z}|x)} \left[-\log  p\_\zeta(\tilde{z}) - \log \mathcal{N}\left(x; \mathcal{D}\_\psi(\tilde{z}), \frac{1}{2\lambda} \right) \right].
\end{aligned}
$$


Recall that VAEs (Kingma and Welling, 2014) are trained to minimize a KL divergence between a variational posterior $q(\tilde{z}|x)$ and the true posterior $p(\tilde{z}|x)$:
$$
\begin{aligned}
\mathbb{E}\_{p(x)}\Big[ & KL \big(q(\tilde{z}|x) \lVert p(\tilde{z}|x) \big) \Big] = \mathbb{E}\_{p(x)}\mathbb{E}\_{q(\tilde{z}| x)}\left[ \log \frac{q(\tilde{z}; x)}{p(\tilde{z}|x)} \right] \\\\
&= \mathbb{E}\_{p(x)}\mathbb{E}\_{q(\tilde{z}| x)}\Big[ \log q(\tilde{z}| x) - \log p(x|\tilde{z}) - \log p\_\zeta(\tilde{z}) +\log p(x)  \Big] \\\\
&= \mathbb{E}\_{p(x)} \underbrace{\Big[ - H(q(\tilde{z}| x))\Big]}_{=0} + \underbrace{\mathbb{E}\_{p(x)}\mathbb{E}\_{q(\tilde{z}| x)}\Big[ - \log p\_\zeta(\tilde{z})  - \log p(x|\tilde{z})  \Big]}\_{\equiv\text{relaxed rate-distortion objective}}  + \text{const}
\end{aligned}
$$
If we choose a Gaussian distribution for the conditional likelihood of the observation, i.e. set $p(x|\tilde{z}) = \mathcal{N}\left(x;  \mathcal{D}\_\psi(\tilde{z}), 1 / 2\lambda\right)$, then we recover the relaxed rate-distortion objective.

To summarise, we have established that optimising rate-distortion trade-off (with fixed $\lambda$), using MSE as a distortion metric, is equivalent to training a VAE with Gaussian likelihood. In general, different choices of distortion metrics correspond to different distributions, though not all metrics give rise to a normalised density function. The rate corresponds to the negative log-prior, while distortion corresponds to the negative log-likelihood.
The expected KL then becomes
\begin{aligned}
\mathbb{E}\_{p(x)}\Big[ & KL \big(q(\tilde{z}|x) \lVert p(\tilde{z}|x) \big) \Big] =
\end{aligned}
$$ -->
<!-- establishes the equivalence between optimizing rate-distortion trade-off and training a VAE. -->
<!--
#### Why not use a simple autoencoder?

If you are new to the field of neural compression, you might be wondering (at least I was) why do we bother training a VAE. Indeed, recall that we introduced the uniform noise to deal with the non-differentiability of the quantization step. What if we simply skip quantization? A simple alternative compression pipeline would then be *"encode an image $x$ into a low-dimensional latent representation $z$ and store that losslessly; at decompression time use a decoder to reconstruct $x$ from $z$"*, which would correspond to training a standard (i.e. not variational) autoencoder.

The reason why this procedure will not work is that $z$ being low-dimensional is not sufficient for efficient compression; what we require is $z$ to be **low entropy**. We can have a low-dimensional latent representation $z$, which is high entropy, and vice-versa---$z$ can be the same dimension as $x$ but much lower entropy, making it easy to compress losslessly. To ensure that the latent $z$ is low-entropy we need to be able to calculate that entropy, $H(z)=-\mathbb{E}\_{p(z)}[\log p(z)]$, and to that end, we require the distribution $p(z)$. In other words, for efficient compression we require a full probabilistic model.

**Side note:** The entropy model $p_\zeta(z)$ corresponds to the prior in a VAE. In some applications, this prior is a fixed (simple) distribution, e.g. standard Gaussian, and its main role is to act as a regulariser. In neural compression settings the prior is learned and, as discussed above, is crucial for efficient compression. Indeed a lot of research has been focused on improving the entropy model---for example, Ball&#233 et al. (2018) introduce a hyperprior. For a recent review on learned image compression see Hu et al. (2021).


## How good are learned image compression models?

Learned image compression methods are competitive or outperforming traditional ones, such as JPEG and BPG, on the basis of standard performance metrics such as peak signal-to-noise ratio (PSNR) and multi-scale structural similarity index measure (MS-SSIM). The figure below is taken from  Hu et al. (2021) and shows rate-distortion curves for different approaches.

{{< figure library="true" src="neural_compression_performance.png" title="Rate-distortion curves achieved by traditional methods (JPEG and BPG) neural-based learned approaches (all the rest) on the Kodak dataset. **Source**: Hu et al. (2021)." numbered="true"  align="left">}}


## Challenges

The field of learnt image compression has undoubtedly made huge progress over the past few years.
It is an active area of research, and many enhancements are being developed, e.g. better probabilistic models, better training objectives, per instance fine-tuning etc.
However, two key challenges need to be addressed before these methods find their way into practice:
- Computational cost: encoding and decoding can be too slow for practical applications.
- Storage cost: encoder, decoder and entropy model typically have a lot of parameters, which could make them prohibitively expensive to store on end devices such as mobile phones.
<!--

## References

Ball&#233 J., V. Laparra, and E. P. Simoncelli, "End-to-end Optimized Image Compression", in International Conference on Learning Representations (ICLR), 2017.

Ball&#233 J., D. Minnen, S. Singh, S. J. Hwang, and N. Johnston, "Variational image compression with a scale hyperprior", in International Conference on Learning Representations (ICLR), 2018.

Hu Y.,  W. Yang, Z. Ma, and J. Liu  "Learning End-to-End Lossy Image Compression: A Benchmark", IEEE transactions on pattern analysis and machine intelligence, 2021.

Kingma D. P., and M. Welling, "Auto-encoding variational Bayes", arXiv preprint arXiv:1312.6114, 2013.

## Further reading
This blog is in no way an extensive introduction to the field of neural compression. A good starting point is the review paper of Hu et al. (2021); in addition to the papers mentioned, some further resources include:

- Papers:
  - F. Mentzer et al. (2019) "Conditional Probability Models for Deep Image Compression"
  - J. Townsend et al. (2019) "Practical Lossless Compression with Latent Variables using Bits Back Coding"
  - Y. Yang et al. (2021) "Improving Inference for Neural Image Compression"

- Other:
  - K. Ullrich's PhD thesis ["A coding perspective on deep latent variable models"](https://pure.uva.nl/ws/files/50173216/Thesis.pdf).
  - C. Steinruecken's PhD thesis ["Lossless data compression"](https://q4.github.io/thesis.pdf).
  - J. Townsend's tutorial on rANS: ["A tutorial on the range variant of asymmetric numeral systems"](https://arxiv.org/pdf/2001.09186.pdf).
  - R. Bamler's course ["Data compression with deep probabilistic models"](https://robamler.github.io/teaching/compress21/).

**Update**: ["An Introduction to Neural Data Compression"](https://arxiv.org/abs/2202.06533) (2022) by Y. Yang, S. Mandt and L. Theis is the best comprehensive introduction to the topic.


<!-- Mentzer F., E. Agustsson, M. Tschannen, R. Timofte, and L. Van Gool, "Conditional probability models for deep image compression", In IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2018. -->
