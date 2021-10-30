---
title: "Deep Adaptive Design: Bayesian experimental design in real time"

event: Oxford Machine Learning Summer School (OxML)
event_url: https://icml.cc

location: University of Oxford, UK
#address:
#  street:
#  city:
#  region:
#  postcode:
#  country:

summary: "Presented our Deep Adaptive Design (DAD) method at the Un-Conference track of the OxML summer school."
abstract: "In this presentation we will talk about (implicit) Deep Adaptive Design (DAD) - a new policy-based method for performing Bayesian experimental design in real-time. The traditional approach to adaptive experimentation is a two step procedure consisting of posterior inference followed by optimisation of the expected information gain (EIG) objective. Both of these steps usually require heavy computations during the experiment, making the traditional approach unsuitable for many real-world applications, where decisions must typically be made quickly. The DAD approach addresses this restriction by learning an amortised design network that takes past design-outcome pairs as input and outputs the design for the next stage of the experiment using a single forward pass. We illustrate the applicability of our method on a number of experiments and show that it provides fast and effective mechanism for performing adaptive experiments with a wide class of models."

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2021-08-13T10:00:00Z"
date_end: "2021-08-13T10:45:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2021-06-23T00:00:00Z"

authors: []
tags: []

# Is this a featured talk? (true/false)
featured: false

image:
  caption: #'Image credit: [**Mnih et al (2013)**](https://arxiv.org/pdf/1312.5602.pdf)'
  focal_point: Right

links:
#- icon: twitter
#  name: Follow
#  icon_pack: fab
#  url: https://twitter.com/georgecushen
url_code: ""
url_video: "" #"uploads/202103_deepprob_DQN.pdf"
url_slides: "https://docs.google.com/presentation/d/e/2PACX-1vSD_lktYeL-rNc3whhlBaDyjMimlCw6VHVJHo-8Sfm0PPy0A-vYJh5hcPxgY6nRTFun40ctBDFEoTjI/pub?start=false&loop=false&delayms=3000"
url_pdf: "http://proceedings.mlr.press/v139/foster21a.html"

# Markdown Slides (optional).
#   Associate this talk with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects:
- []

date_format: Jan 2006
time_format: ""
---
