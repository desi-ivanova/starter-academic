---
title: "On Some (Fixable) Limitations of 'Understanding the Limitations of Mathematical Reasoning in LLMs'"
subtitle: "Time to make statistics great again?"

# Summary for listings and search engs
summary: "There is huge value in developing new benchmarks and I think the one that proposed in the paper 'GSM-Symbolic: Understanding the Limitations of Mathematical Reasoning in Large Language Models' by Apple is quite neat and useful! The accompanying analysis, in my opinion, can be substantially improved with the help of basic statistics. Without those we risk over-interpreting results and drawing misleading conclusions. I never thought I would be the one advocating for the use of hypothesis tests and p-values, but here we are... When it comes to language models evals, it is time to make statistics great again!"
# Link this post with a project
projects: []

# Date published
date: "2024-10-21T00:00:00Z"

# Date updated
lastmod: "2024-10-21T00:00:00Z"

# Is this an unpublished draft?
draft: false

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
- Machine learning
- Language models
- LLM
- Statistics

categories:
- ML
---

**Also on [Substack](https://substack.com/home/post/p-150508215).**

Apple’s recent paper "GSM-Symbolic: Understanding the Limitations of Mathematical Reasoning in Large Language Models" [1], has sparked another round of debates on social media about whether (large) language models (LMs) do real reasoning or not.

After presenting this paper at a reading group last week, I thought I’d share my thoughts and findings from some additional analysis in a short post. Unlike most of the posts on social media, I’ll try to stay as far away as possible from the philosophical claims or hypotheses around what constitutes “true reasoning” vs “sophisticated pattern matching”. Instead, my aim is to provide a bit of statistical rigour to help us evaluate the claims and conclusions made in the paper.


**TLDR:** The emphasis on “non-negligible variance” or “increase in variance” throughout the paper appears to be an over-interpretation of normal statistical variation. Further, for 21 out of 25 models, there isn’t enough evidence to reject the null hypothesis that performance on GSM8K is equal to that on GSM-Symbolic. Of the 4 models where the null can be rejected, 3 perform worse on GSM-Symbolic, and 1 performs better.

**Note:** My analysis is somewhat constrained by the data and detail presented in the paper. The authors promised to [release the benchmark soon](https://x.com/MFarajtabar/status/1844617489679860088), so more rigorous analysis can follow.

Preliminaries (paper summary)

What does the paper do?

Propose a new benchmark: GSM-Symbolic, which generates variants of GSM8K [2] — a well-established benchmark, containing grade school math word problems. GSM-Symbolic is designed to have the same distribution as the original GSM8K dataset. However, GSM-Symbolic is much less likely to have been leaked into model training sets (data contamination). Further, the paper introduces a way to control the difficulty of the questions, giving rise to 3 more datasets (one simpler and two more complex than GSM-Symbolic). A final benchmark, GSM-NoOp, is obtained by introducing  “seemingly relevant but ultimately irrelevant” information into GSM-Symbolic. 

Evaluate 25 language models on the new benchmarks. Model families include: Gemma, Phi, Mistral and Llama (open weights), and GPT and o1 (proprietary, OpenAI).

How is the GSM-Symbolic benchmark constructed?

The idea is to create a template for the questions in the original GSM8K dataset by identifying variables whose values can be modified, whilst preserving the structure and logic of the question. For each of these variables a domain of possible values is specified, along with a set of necessary conditions to ensure correctness of the new question and answer. Here’s an example of such a template:

{{< figure library="true" src="gsm_template.png" title="Figure from Mirzadeh et al. (2024) https://arxiv.org/pdf/2410.05229." numbered="false">}}





**Cite as:**
```
@misc{
    ivanova2024gsm, 
    title = "On Some (Fixable) Limitations of 'Understanding the Limitations of Mathematical Reasoning in LLMs'", 
    author = "Ivanova, Desi R", 
    journal = "desirivanova.com", 
    year = "2024",
    url = "https://desirivanova.com/post/gsm-symbolic/"
}
```

[1] Mirzadeh, I., Alizadeh, K., Shahrokhi, H., Tuzel, O., Bengio, S., & Farajtabar, M. (2024). GSM-Symbolic: Understanding the Limitations of Mathematical Reasoning in Large Language Models. arXiv preprint arXiv:2410.05229.