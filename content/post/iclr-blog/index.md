---
title: "Towards more rigorous evaluations of language models"
subtitle: ""

# Summary for listings and search engs
summary: "As language models become increasingly sophisticated and existing benchmarks approach saturation, the need for rigorous evaluation methods grows more pressing. Many evaluations lack the statistical rigour needed to draw meaningful conclusions, leading to a potential over-confidence in results that might not hold up under scrutiny or replication. This post advocates for bringing fundamental statistical principles to language model evaluation, demonstrating how basic statistical analysis can provide more reliable insights into model capabilities and limitations."
# Link this post with a project
projects: []

# Date published
date: "2024-10-22T00:00:00Z"

# Date updated
lastmod: "2024-10-22T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption: 
  focal_point: ""
  placement: 2
  preview_only: 1

authors:
- admin

tags:
- Machine learning
- Language models
- LLM
- Statistics

categories:
- ML
---



<!-- A well-designed and well-executed empirical study should: (1) clarify assumptions and present alternative explanations; 
(2) offer measures of (un)certainty; and (3) make reasonable attempts to test models on "out-of-sample" data or report 
level of train-test overlap. 
Careful consideration of training-test overlap, with reasonable attempts to evaluate on out-of-sample data when possible-->

# Introduction

A staggering volume of research papers on large language models (LLMs) is published daily. 
On the day of writing this (5th Nov 2024), 239 papers containing "LLM" or "Language Model" in their titles were 
added to the Computer Science section of arXiv alone.
This flood of publications makes it difficult to separate genuine insights from noise. 
This challenge is particularly pronounced in areas that lack precise definitions (e.g. "reasoning"). 
Given the empirical nature of the LLM research field, much is open to interpretation, making rigorous analysis essential to reduce bias and improve reliability.


In this post, we argue that LLM researchers, especially those areas where core concepts lack established definitions, must more actively adopt statistical methods to assess claims rigorously.
Various techniques from classical statistics provide quantitative measures that can help validate and strengthen research findings.
Leveraging these techniques will help move the field beyond anecdotal observations and philosophical arguments toward a more scientific understanding of model behavior.



### Elements of Rigorous Empirical Evaluation:

We identify three key elements that are necessary for rigorous empirical evaluation:

1. Clear articulation of assumptions and consideration of alternative explanations.
2. Quantification of uncertainty in results through appropriate statistical measures.
3. Careful consideration of train-test overlap, with reasonable attempts to evaluate on "out-of-sample" datasets when possible.

> If one does not give thought to what the data would be like under the assumption that one’s theory is false, one is likely reinforcing confirmation bias rather than establishing the validity of the theory. *"Reproducibility, p-values, and type III errors: response to Mayo", Philip B. Stark (2022)*

The first step in rigorous evaluation is to clarify assumptions and explore alternative explanations.
Experiments can often be designed in ways that unintentionally favour the hypothesis being tested.
By being upfront about assumptions, questioning their validity, and investigating alternative explanations, researchers will improve the reliability and robustness of their conclusions. 

> In some sense it [the p-value] offers a first line of defense against being fooled by randomness, separating signal from noise [...]. *"It’s Not the P-Values’ Fault", Yoav Benjamini (2016).*

A second essential step is quantifying uncertainty, using tools like error bars and confidence intervals, which help us gauge the reliability of performance metrics by providing a range of plausible values.
Additionally, although often criticized, hypothesis tests and p-values can serve as an initial, coarse filter to distinguish signal from noise, laying the groundwork for deeper exploration into the practical significance of the reported results.


Finally, evaluating models on entirely unseen ("out-of-sample") data is essential for accurately assessing their true capabilities. Unfortunately, determining the extent of train-test overlap is both challenging and frequently overlooked. Many language models lack transparency in this area, with benchmark datasets often *leaked* (or *contaminated*) during training, resulting in biased evaluations and inflated performance metrics. 
As Zhang et al. (2024) highlight, reporting train-test overlap statistics is crucial for ensuring the validity of model evaluations.


To illustrate these principles, we use [Mirzadeh et al. (2024)](https://arxiv.org/pdf/2410.05229) as a case study---a recent paper that received substantial attention from the LLM research community (e.g. see [this](https://machinelearning.apple.com/research/gsm-symbolic), [this](https://www.reddit.com/r/singularity/comments/1g1zphu/apple_ai_researchers_question_openais_claims/), or [this](https://x.com/MFarajtabar/status/1844456880971858028)).
The paper examines whether LLMs perform "formal reasoning" or rely on "sophisticated pattern matching". 
We review their methods, identify gaps in their analysis, and offer a more rigorous statistical assessment of their claims.


# Summary of [Mirzadeh et al. (2024)](https://arxiv.org/pdf/2410.05229)

Overall, the paper makes two technical contributions: (1) a new benchmark, called GSM-Symbolic, for evaluating mathematical reasoning of LMs, and (2) an empirical evaluation of 25 LMs on this new benchmark to assess their reasoning capabilities.

## 1. What is the new benchmark and how is it constructed?

The authors propose GSM-Symbolic, a variant of the well-established GSM8K benchmark for grade school math word problems (Cobbe et al., 2021). Since GSM8K has likely leaked into many LMs' training sets due to its popularity, GSM-Symbolic aims to match its distribution whilst eliminating (or reducing) train-test overlap.

The authors also construct four variants of GSM-Symbolic by modifying question difficulty:
- GSM-M1: An easier version that removes one clause from each question.
- GSM-P1 and GSM-P2: Progressively harder versions that add one or two extra clauses respectively.
- GSM-NoOp: A version that adds "seemingly relevant but ultimately irrelevant information" that should have no operational significance (No-Op).

To generate GSM-Symbolic and the variants, the authors create "templates" from GSM8K questions by identifying modifiable variables whilst preserving the underlying logic. 
Each variable has a specified domain and constraints to ensure valid questions and answers. Here's an example template:

{{< figure library="true" src="template_gsm.png" title="Figure from Mirzadeh et al. (2024) https://arxiv.org/pdf/2410.05229." numbered="false">}}

For their analysis, the authors select 100 questions from GSM8K and create such a template for each of them.
Whilst the paper does not specify their selection method, we suppose that these questions were sampled randomly from GSM8K's test set.
By sampling values for the variables, 50 new questions are generated from each template. 
This means that GSM-Symbolic and each of its 4 variants (GSM-M1, GSM-P1, GSM-P2, and GSM-NoOp) contain **50 datasets of 100 samples each**.

Throughout this post, when we refer to "GSM8K", we specifically mean those 100 original GSM8K questions that were used to create the templates, not the full GSM8K dataset.

## 2. What are the key findings and conclusions from the empirical evaluation?

The evaluated model families include Gemma, Phi, Mistral and Llama (open weights), and GPT and o1 (proprietary, OpenAI). Of the open weights models, one can be considered "medium" size (Gemma2 27B params), and all the rest can be considered "small" (9B params or less).
The metrics reported are average accuracy across the 50 versions of each dataset and the standard deviation of these accuracies.

The key findings are:

- Performance variability: LMs exhibit some variability in performance across different instnationals of the same question.

- Performance decline: Compared to GSM8K, performance on GSM-Symbolic drops, suggesting potential data contamination.

- Sensitivity to question complexity: As complexity of the questions increases, “the performance [of models] decreases and the variance increases”, which is said to suggest that “models are not performing formal reasoning”, and that the increase in variance is “in line with the pattern-matching hypothesis”.

- Impact of irrelevant information: Introducing a clause of no operational significance (No-Op), leads to large performance degradation across models, suggesting “deeper issues in their reasoning processes”.

The paper concludes that LMs “are not performing formal reasoning”.

# Critical analysis and re-evaluation

## 1. Performance variability: expected or surprising?

> As shown, all models exhibit a **non-negligible variance** across different sets. […] It is **interesting that this variation even exists** […]. (page XX)

The authors emphasise the "non-negligible variance" in model performance across different GSM-Symbolic datasets, framing it as surprising. But is it?

### When is variability *not expected*?

Variability would indeed be unexpected if each resampled question was effectively the same as the original. 
The implicit assumption here is that if an LM solves (or fails to solve) a given question once, it should always solve (or fail to solve) it when presented with the same problem but with different numbers. 
In other words, this assumes that LMs never make arithmetic mistakes---a very strong assumption, that is not mentioned or examined in the paper. 

**Is this assumption valid?** 
For humans, it clearly is not. 
Even when solving the same problem with different numbers, humans can make arithmetic mistakes.
The same applies to LMs. [todo: quote some papers that look into this??]

We can demonstrate this empirically with a simple experiment. 
The table below shows how two LMs perform on basic addition problems with varying digit lengths (e.g. "What is 147 + 562?"). Consistent with prior literature [??], as the numbers get larger, accuracy declines, showing that simple arithmetic isn't perfectly reliable.

Model | 1 digit | 2 digits | 3 digits | 4 digits
--- | --- | --- | --- | ---
Phi-3.5-mini-instruct | 100% | 90.2% | 90.8% | 84.0%
Llama-3-8B-Instruct | 100% | 100% | 100% | 95.3%
*Table XX: CoT prompting, zero-shot.*

More on this in Section XX. [TODO]

**Is performing arithmetic part of reasoning?** 
Solving a word math problem consists of two steps: (1) translating the text to a sequence of operations, and (2) performing the operations correctly.
Whilst the first step clearly requires reasoning ability, we argue that the second is more mechanical in nature. 
The authors could have isolated "pure reasoning" capabilities by providing models with calculator access, which would have reduced (though likely not fully eliminated) the confounding effect of arithmetic errors in their evaluation.

### When is variability *expected*?

If we (rightfully) reject the assumption that LMs are infallible in arithmetic, performance variability across datasets becomes entirely expected.
Even if a model applies consistent reasoning, it may still make arithmetic errors, leading to natural performance variation.

**How much variation is expected?**  
By now, it should be clear that the answer to this question depends on the assumptions we make about the data.
One reasonable assumption we could make is that each model $m=1, \dots, 25$, answers each question $n=1,\dots,100$ correctly with some probability $p_{m,n}$.
Unfortuantely, since the paper does not provide question-level performance data, for the purposes of this analysis, we must further assume that this probability is constant across questions, that is $p_{m,n}=p_m$ for all $n$. 

Thus, an LM answering questions is modelled as an independent and identically distributed Bernoulli trial with a model-specific success probability $p_m$.
Under this assumption, the total number of correct answers on a dataset of size $N=100$ is

$$\text{Binomial}(N, p_m).$$

The variance of this distribution is **fully determined by the success probability $p_m$** and equals 

$$N \cdot p_m \cdot (1-p_m).$$

This variance is maximised when $p_m=1/2$ and goes to $0$ as $p_m$ approaches $0$ or $1$.

To quantify what constitutes "normal" variation under our assumption, we can construct confidence intervals (CI) for the point estimates of $p_m$ (these are reported in the second column of Table 1 in the Appendix of the paper). 
There are different ways to construct CI for the [Binomial proportion](https://en.wikipedia.org/wiki/Binomial_proportion_confidence_interval). Figure XX below shows [Wilson score](https://en.wikipedia.org/wiki/Binomial_proportion_confidence_interval#Wilson_score_interval) intervals. (See Appendix for more results.)

{{< figure library="true" src="wilson_0.95ci.png" title="95% Wilson score intervals for the point estimates of $p_m$." numbered="false">}}

As expected, models with success probabilities closer to $1/2$ (e.g. Gemma-7b, Phi-2, Mistral-7b-v0.1) exhibit wider confidence intervals, reflecting higher variability. Conversely, models with success probabilities closer to 0 or 1 (Gemma2b, GPT-4o, o1-preview) have substantially narrower intervals.

**Assuming** that GSM8K and GSM-Symbolic come from the same distributions (more on that in Section XX), let’s look at Figure 2 of the paper showing.

**Note:** The x-axis scales are different for different models.

{{< figure library="true" src="fig2_gsm.png" title="Figure 2 from Mirzadeh et al. (2024) https://arxiv.org/pdf/2410.05229." numbered="false">}}

For the models shown in this figure, the GSM8K accuracy ranges from 74% for the weakest model (Llama3-8B-instruct) to 95% for the strongest model (GPT-4o). 
The weakest model, Llama3-8B-instruct, shows a much wider variation in accuracy in the range $\sim (69\%, 81\%)$. 
The strongest model, GPT-4o, has a considerably narrower spread, between $\sim (91\%, 98\%)$. 
Critically, for both of these models, **the variation in GSM-Symbolic performance falls well within the CIs we calculated above!** 
These are $(64.6\%, 81.6\%)$ for Llama3-8B-instruct and $(88.8\%, 97.8\%)$ for GPT-4o. 
This suggests that the variation in performance for these two models is quite expected.

The table below compares the 95% Wilson score intervals to the approximate accuracy ranges reported in the paper:
| Model                          | 95% Wilson score CI | Reported ranges (approximate)  |
|--------------------------------|---------------------|-----------------------|
| Gemma2-2b-it                   | (37%, 56%)          | (34%, 48%)            |
| Gemma2-9b-it                   | (79%, 92%)          | (71%, 85%)            |
| Phi-2                          | (43%, 62%)          | (35%, 50%)            |
| Phi-3-mini-128k-instruct       | (77%, 91%)          | (75%, 90%)            |
| Phi-3-medium-128k-instruct     | (81%, 94%)          | (75%, 84%)            |
| Mistral-7b-instruct-v0.1       | (33%, 52%)          | (23%, 38%)            |
| Mathstral-7b-v0.1              | (71%, 87%)          | (71%, 81%)            |
| Llama3-8b-instruct             | (65%, 82%)          | (69%, 81%)            |
| GPT-4o                         | (89%, 98%)          | (91%, 98%)            |
| o1-mini                        | (86%, 97%)          | (90%, 97%)            |
| o1-preview                     | (90%, 98%)          | (88%, 96%)            |
*Table XX: 95% Wilson score intervals for the point estimates of $p_m$ and reported ranges, approximated from Figure 1 in Mirzadeh et al. (2024), as well as Figures 10 and 12 from the appendix.*

We note that our confidence intervals are generally wider than the reported ranges, i.e. under the iid Bernoulli assumption, the variation is actually **larger** than what is reported.
This discrepancy is likely due to the correlations between questions, which can inflate the variability captured by the confidence intervals. 
As initially suggested, a more precise estimation would involve modelling the probability of success on a question level, $p_{m,n}$, rather than assuming each question is equally likely to be answered correctly. 
The analysis can be repeated once (if) the detailed question-level data becomes available.

**Verdict:** The observed variability in GSM-Symbolic performance is not inherently surprising, and is in fact expected.


## 2. Performance decline on GSM-Symbolic

The paper claims that LMs perform worse on GSM-Symbolic compared to GSM8K. Let's examine the evidence presented in Section 4.1, which we quote directly:

> Another noteworthy observation is that the performance (represented by the dashed line in Fig. 2) on the original questions from the 100 examples of GSM8K used as templates is **often more than one standard deviation away from the center** of the GSM-Symbolic performance distribution, frequently on the right side of the distribution (this holds for 21 out of 25 models). **One explanation** for this could be data contamination […]

There are two issues with the above quote. 
First, the authors suggest data contamination as one possible explanation for the performance decline, but do not explore other plausible explanations.
Second, they rely on a hand-wavy "one standard deviation" criterion to suggest that the decline in performance is significant, without proper statistical analysis. 
We address both of these next. 

### Alternative explanations

Mention somehwere: not mutually exclusive -- both can be true and should be evaluated.

In addition to data contamination, another plausible explanation for the alleged performance discrepancy is a **distribution mismatch** between GSM8K and GSM-Symbolic.
Indeed, there is some evidence suggesting that GSM-Symbolic questions might be inherently more difficult.
Looking at the example template (Figure 1 above), we can see that the proposed sampling ranges for variables **exclude** the original GSM8K values.
For instance: the variable "total" is sampled from $[100, 500]$ whilst in the original question we have total$=62$; similarly, the variable "ans" is sampled from $[85, 200]$ whilst in the original question we have ans$=14$.
In other words, it is impossible to generate the original GSM8K question from the proposed template.

A more appropriate sampling domain for both "total" and "ans" might be $[50, 99]$.
This seemingly small change in number ranges can significantly impact the performance of language models due to tokenisation.
Consider how Phi-3.5-mini-instruct and Llama3-8b handle numbers differently: Phi-3.5-mini-instruct uses 2 tokens for numbers $[10, 99]$ and 3 tokens for numbers $[100, 500]$, whilst Llama3-8b uses just 1 token for all numbers in $[10, 500]$.

To understand the impact, let's model the probability of making mistakes during arithmetic operations.
Assume there's a base error probability $\epsilon$ per token operation, which increases linearly with the total number of tokens involved.
For $N$ total tokens in an operation, the error probability becomes $(N-1)\epsilon$ (so that adding two numbers each taking 1 token we have $N=2$ and the error probability is $\epsilon$).

[run the analysis on the data, will need the raw data for this!]
The question in Figure 1 involves three arithmetic operations (two additions and one subtraction).
In the worst case, where all numbers are at least 100, Llama3-8b would be correct with probability $(1-\epsilon)^3$ since it uses 1 token per number. 
Phi-3.5-mini-instruct, on the other hand, would be correct with probability $(1-5\epsilon)^3$ as it uses 3 tokens per number.
Taking $\epsilon=0.01$ as an example gives us a correct solution probability of $97.03\%$ for Llama3-8b, and only $85.74\%$ for Phi-3.5-mini-instruct.
If all numbers are in the range $[10, 99]$, the probability of correct solution remains $97.03\%$ for Llama3-8b and goes up to $91.27\%$ for Phi-3.5-mini-instruct.

If the number ranges in GSM-Symbolic are systematically chosen to be larger than those in GSM8K (and don't even include the original question values), then we cannot claim that both datasets come from the same distribution.
Tokenisation is one mechansim that explains why this matters; larger number ranges in GSM-Symbolic may inherently disadvantage certain (and eventually all) models, potentially explaining some of the observed performance differences between models and datasets.

> We have also observed the performance of LLMs deteriorating as question complexity increases.

The same analysis is applicable to more complex questions; e.g. adding one extra clause, even if it takes only one operation --> similalry for 2 

do a table!

--> Say reasoning: translating the text to a sequence of operations; what the paper tests is whether models can do that *and* perofrm the operations correctly. The rest of this post will not deal with reasoning; 

### Statistical significance?

For many models in Figure 2, the dashed line is in the right tail of the distribution. Additionally, Figure 3 of the paper reports substantial performance decrease for many other models. Can we claim that there is a statistically significant difference in model performance between GSM-Symbolic and GSM8K?

{{< figure library="true" src="fig3_gsm.png" title="Figure from Mirzadeh et al. (2024) https://arxiv.org/pdf/2410.05229." numbered="false">}}


Even assuming GSM8K and GSM-Symbolic come from the same distribution, the results presented lack statistical rigour:

Additionally, the claims about performance differences on GSM-Symbolic compared to GSM8K are not supported by formal statistical testing. Without a certain level of statistical rigour, it is impossible to distinguish between genuine phenomena and random variation in the data. 

The rest of this section aims to provide the statistical rigour needed to properly analyze these results.


### Hypothesis testing

To assess whether there are statistically significant differences in model performance between GSM8K and GSM-Symbolic, we can use hypothesis testing.
For each model $m$, we want to test whether its success probability on GSM8K, $p_{m,8k}$ equals its success probability on GSM-Symbolic, $p_{m,symb}$. This equality forms our null hypothesis. Our alternative hypothesis can take two forms:

- Two-sided: The success probabilities are different
$$
H_0: p_{m, 8k} = p_{m, symb} \quad\quad\quad H^\text{two-sided}_A: p_{m, 8k} \neq p_{m, symb}.
$$
- One-sided: The success probability on GSM8K is greater than on GSM-Symbolic
$$
H_0: p_{m, 8k} = p_{m, symb} \quad\quad\quad H^\text{one-sided}_A: p_{m, 8k} < p_{m, symb}.
$$

Want to asssess this for each model independently, we use a two-sample test for the Binomial proportion. We use the normal approximation (i.e. a two-sample t-test) for those models where $N \cdot p_m \geq 10$ and $N \cdot (1-p_m) \geq 10$ ( the rule of thumb is that the approximation is good); and exact test otherwise (this is only for the OpenAI models)


Common choices of significance level for statistical test are 5% and 1%.

The results of these hypothesis tests at the 5% significance level can be read from the following chart (the results at the 1% level are included in the Appendix):


We see that there are 4 models for which we are able to reject the null: Gemma-7b, Mistral-7b-instruct-v0.1, Phi-2 and Llama3-8b. Note that the performance of Llama3-8b on GSM-Symbolic appears to be statistically better than on GSM8K (with some minor technicalities on one- vs two-sided tests that I won’t get into here). 

This analysis is by no means perfect (e.g. we haven’t discussed things like multiple testing,, which I might do in another post), but it does provide a degree of rigour, which I find increasingly necessary in LLM research. Especially when talking about concepts like “reasoning” that don’t have well-established definitions.

**To summarise: analysing models independently, 21 out of 25 models show statistically equivalent performance on GSM8K and GSM-Symbolic.**


### Paired hypothesis test

A few people correctly pointed out the trend that that many models perform worse on GSM-Symbolic than on GSM8K. To assess the statistical significance of this systematic trend, we can conduct what is known as a paired difference test. The Wilcoxon signed-rank test would be an appropriate one to apply in our case with two important caveats.  

**Caveat 1**: Non-independent data. It will be incorrect to perform the test on all 25 models as these are not independent. There are several types of dependence to consider. Most obviously, the base models and their instruct-tuned version are clearly related (e.g. Gemma2-9b and Gemma2-9b-it). I’d also argue that different sizes within the same model family cannot be considered independent (e.g. mini-small-medium for Phi or 2b-9b-27b for Gemma); minor version updates (e.g. Mistral v0.1 vs 0.3, Phi 3 vs 3.5) will also likely be correlated. So although we have a sample of 25 models, the “effective” sample size is much, much smaller. 

Here’s my attempt to come up with independent set of models. In each model family, I’ll take the latest, largest instruct-tuned version. I’ll repeat this by also taking the smallest version. This gives us two sets of 7 models (differences between them are in italics):

- Largest subset of models: Gemma2-27b-it, Phi-3.5-mini-instruct, Mistral-7b-instruct-v0.3, Mathstral-7b-v0.1, Llama3-8b-instruct, GPT-4o, o1-preview.

- Smallest subset of models: Gemma2-2b-it, Phi-3.5-mini-instruct, Mistral-7b-instruct-v0.3, Mathstral-7b-v0.1, Llama3-8b-instruct, GPT-4o-mini, o1-mini.

**Caveat 2**: The results of GSM-Symbolic are averages across 50 datasets, whilst GSM8K is based on a single sample. I think we’d want to compare accuracies on GSM8K vs accuracies on each of the 50 datasets (at the time of writing these are not publicly available). I’m still trying to figure out what is the correct way to handle the unbalanced number of samples in this case is (more on this to come). 

With those two caveats, here is the hypothesis and results:

$$
H_0: p_{8k} = p_{symb} \quad\quad H_A^\text{two-sided}: p_{8k} \neq p_{symb} \text{ ~or~ } H_A^\text{one-sided}: p_{8k} < p_{symb},
$$
where  $p_{8k}=[p_{1,8k}, \dots, p_{7,8k}]$ and $p_{symb}= [p_{1,symb}, \dots, p_{7,symb}]$.

- Largest subset of models: 

  - p-value of the two-sided test is 0.046875. This means that we can reject the null at the 5%, but not at the 1%, that there are performance differences between GSM-Symbolic and GSM8K. 

  - p-value of the one-sided test is 0.0234375.

- Smallest subset of models: 

  - p-value of the two-sided test is 0.078125, i.e. we cannot reject the null at the 5% or the 1% level. 

  - p-value of the one-sided test is 0.0390625.

**To summarise: analysing the results of models jointly, there appears to be some evidence of differences in performance.**


## 3. Performance decreases and increasing variance with question complexity

The paper highlights this on multiple occasions, most notably in Section 4.3. Some examples include:

> [page 3] We show that performance degradation and variance increase as the number of clauses increases, indicating that LLMs’ reasoning capabilities struggle […]

> [page 9] the increase in variance suggests that searching and pattern-matching become significantly harder for models as the difficulty increases.

> [page 9, Figure 6 reproduced below] the distribution of performance shifts to the left (i.e., accuracy decreases), and the variance increases.

{{< figure library="true" src="fig6_gsm.png" title="Figure from Mirzadeh et al. (2024) https://arxiv.org/pdf/2410.05229." numbered="false">}}

Here we have 4 different datasets: the baseline GSM-Symbolic, an easier version of it (M1) and two harder versions of it (P1 and P2). It is not unreasonable to expect that a model might perform differently on different datasets, for example better on the easier ones and worse on harder ones. We can make a similar assumption as before, namely that the answers a model m gives to questions of difficulty $d=-1, 0, 1, 2$ are i.i.d. Bernoulli trials with success probability $p_{m,d}$. The distribution of the total number of correct answers is then $\text{Bin}(N=100, p_{m,d})$, which has variance $N \cdot p_{m,d} \cdot (1 - p_{m,d})$.

If we happen to have decreasing probabilities of success, i.e. $p_{m,-1} > p_{m,0} > p_{m,1} > p_{m,2} > 0.5$ as is the case for some of the models in Figure 6, then the corresponding variances must increase. Whether or not this decrease in probability of success has any relationship to the “reasoning abilities” of a model is beyond the scope of this blog :)

**To summarise: the emphasis on “non-negligible variance” and “increase in variance” throughout the paper appears to be an over-interpretation of normal statistical variation.**

## Conclusion

There’s huge value in developing new benchmarks and I think the proposed GSM-Symbolic is quite neat and useful! The accompanying analysis, in my opinion, can be substantially improved with the help of basic statistics. Without those we risk over-interpreting results and drawing misleading conclusions.

Findings:
- Depending on assumptions, variability is expected and quantifiable; and the observed increase in variance with question complexity is expected.
- The observed performance degradation on GSM-Symbolic is not statistically significant.
- Mismatch between GSM8K and GSM-Symbolic distributions may explain some of the observed performance differences between models and datasets (in addition to contamination and "lack of reasoning").




References 

[1] Mirzadeh, I., Alizadeh, K., Shahrokhi, H., Tuzel, O., Bengio, S., & Farajtabar, M. (2024). GSM-Symbolic: Understanding the Limitations of Mathematical Reasoning in Large Language Models. arXiv preprint arXiv:2410.05229.

[2] Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., ... & Schulman, J. (2021). Training verifiers to solve math word problems. arXiv preprint arXiv:2110.14168.

[3] Benjamini, Y. (2016). It’s Not the P-Values’ Fault. 

[4] Stark, P. B. (2022). Reproducibility, p-values, and type III errors: response to Mayo (2022). Conservation Biology, 36, e13986. https://doi.org/10.1111/cobi.13986

[5] Zhang, A. K., Klyman, K., Mai, Y., Levine, Y., Zhang, Y., Bommasani, R., & Liang, P. (2024). Language model developers should report train-test overlap. arXiv preprint arXiv:2410.08385.

[6] Wasserstein, R. L. (2019). Moving to a world beyond “p< 0.05”.

Valmeekam, K., Olmo, A., Sreedharan, S., & Kambhampati, S. (2022, November). Large language models still can't plan (a benchmark for LLMs on planning and reasoning about change). In NeurIPS 2022 Foundation Models for Decision Making Workshop.

### BIBTEX

@misc{wasserstein2019moving,
  title={Moving to a world beyond “p< 0.05”},
  author={Wasserstein, RL},
  year={2019},
  publisher={Taylor \& Francis}
}

@inproceedings{valmeekam2022large,
  title={Large language models still can't plan (a benchmark for LLMs on planning and reasoning about change)},
  author={Valmeekam, Karthik and Olmo, Alberto and Sreedharan, Sarath and Kambhampati, Subbarao},
  booktitle={NeurIPS 2022 Foundation Models for Decision Making Workshop},
  year={2022}
}

### Acknowledgement 

I’d like to Momchil Konstantinov and Alex Coca, Ilija Ilievski and Adam Goliński for their feedback on this post.

### Appendix


Paragraph about p-values should go somehwere. Things to say: 

- p-values get misinterpreted; Some "Don'ts" : don't base your conclusions solely on whether an association or effect was found to be “statistically significant”; Don’t believe that an association or effect exists just because it was statistically significant. Don’t believe that an association or effect is absent just because it was not statistically significant. Don’t conclude anything about scientific or practical importance based on statistical significance (or lack thereof).


#### Clopper-Pearson CIs
For robustness purposes, here are the [Clopper-Pearson](https://en.wikipedia.org/wiki/Binomial_proportion_confidence_interval#Clopper–Pearson_interval) confidence intervals as well:

<iframe title="95% confidence intervals for Binomial proportion" aria-label="Dot Plot" id="datawrapper-chart-LkJtn" src="https://datawrapper.dwcdn.net/LkJtn/9/" scrolling="no" frameborder="0" style="width: 0; min-width: 100% !important; border: none;" height="607" data-external="1"></iframe><script type="text/javascript">!function(){"use strict";window.addEventListener("message",(function(a){if(void 0!==a.data["datawrapper-height"]){var e=document.querySelectorAll("iframe");for(var t in a.data["datawrapper-height"])for(var r=0;r<e.length;r++)if(e[r].contentWindow===a.source){var i=a.data["datawrapper-height"][t]+"px";e[r].style.height=i}}}))}();</script>

The Clopper-Pearson CIs are slightly wider than those obtained using the Wilson score. Using Clopper-Pearson would not have changed any of the results presented in this post. 

#### Two-sample binomial proportion test
Strictly speaking, pm,symb is not a fixed number but an estimate computed using another sample. Therefore truly correct test to perform here is a two-sample binomial proportion test:
$$
H_0: p_{m, 8k} - p_{m, symb} =0 \quad\quad\quad H_A: p_{m, 8k} - p_{m, symb} \neq 0
$$

Under the null $p_{m, 8k} = p_{m, symb}$ and so we estimate both using a pooled estimate:
$$
p_{pool} = \frac{(100 p_{8k} + 5000p_{symb})}{100+5000} \quad \text{SE}(p_{pool}) = \sqrt{p_{pool}*(1-p_{pool}) (1/100+1/5000)}.
$$
The test statistic (pm,8k - pm,symb) / SE(ppool)  is then approximately normal and is used co compute p-values, which I’ve done in [this spreadsheet](https://docs.google.com/spreadsheets/d/1Ul6ZgFXf_II5EFUCgnJ9hSIQYwHxogxYBmwDn_bA4sA/edit?usp=sharing). The results in this case are exactly the same as before:  we are able to reject the null for Gemma-7b, Mistral-7b-instruct-v0.1 and Phi-2 (performing worse), and Llama3-8b (performing better). 

#### 99% Confidence intervals (hypothesis test at the 1% confidence level)

<iframe title="GSM-Symbolic and GSM8K along with 99% CI" aria-label="Dot Plot" id="datawrapper-chart-4DrS6" src="https://datawrapper.dwcdn.net/4DrS6/1/" scrolling="no" frameborder="0" style="width: 0; min-width: 100% !important; border: none;" height="624" data-external="1"></iframe><script type="text/javascript">!function(){"use strict";window.addEventListener("message",(function(a){if(void 0!==a.data["datawrapper-height"]){var e=document.querySelectorAll("iframe");for(var t in a.data["datawrapper-height"])for(var r=0;r<e.length;r++)if(e[r].contentWindow===a.source){var i=a.data["datawrapper-height"][t]+"px";e[r].style.height=i}}}))}();</script>



## Suggested workflow for evaluating language models

1. **Specify the objective and any relevant assumptions:** Define what is being investigated and explicitly outline any relevant assumptions that could impact the evaluation.

2. **Design and run experiments:** Collect performance metrics of interest (e.g., accuracy) along with relevant metadata (e.g., runtime) that could constitute potential confounding factors.

3. **Formulate hypotheses and select an appropriate statistical test:** Null hypothesis significance testing (NHST) is the most common framework for assessing whether the results observed in 2. are substantially (or *statistically significantly*) different from a baseline. 
The null hypothesis ($H_0$) typically posits that there is no difference, whilst the alternative hypothesis ($H_1$) suggests a *meaningful difference* exists. 
The choice of which statistical test to use is determined by characteristics of the data and experimental design. For instance, an independent t-test might be appropriate for comparing performance metrics that can be assumed to follow a normal distribution. For paired data that violate normality assumptions, a Wilcoxon signed-rank test might will be more appropriate. 

4. **Compute relevant statistics:** Calculate confidence intervals and p-values for the observed performance metrics.

5. **Interpret the results:** Confidence intervals provide a range of plausible values for the observed metric, helping establish the reliability and potential variability in model performance. The p-value indicates how likely the observed data would be if the null hypothesis were true; a low p-value suggests evidence against it. Consider these statistical indicators within the broader context of model evaluation. (See section ...).

6. **Accept Uncertainty:** A small p-value serves as an initial, coarse filter for distinguishing signal from noise, but statistical methods cannot transform randomness into certainty. Rather than seeking binary declarations of effects, embrace variation, and treat all statistical conclusions as provisional.
Regardless of the NHST outcomes, critically examine the results, initial assumptions, and alternative explanations. 

**Remark:** Although NHST is the most commonly used framework, it is not the only approach for statistical analysis. 
Bayesian analysis offers an alternative with some practical similarities, such as the use of highest density intervals (HDI). 
However, the two approaches differ deeply in their epistemological foundations: NHST relies on a frequentist interpretation, viewing the results (e.g. accuracy) as repeated sampling from a fixed parameter. In contrast, Bayesian analysis treats parameters themselves as random variables and uses probability to express a degree of belief, that gets updated with new evidence.