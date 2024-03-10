---
title: "Leveraging Self-Consistency for Data-Efficient Amortized Bayesian Inference"

# Authors
# If you created a profile for a user (e.g. the default `admin` user), write the username (folder name) here
# and it will be replaced with their full name and linked to their profile.
authors:
- Marvin Schmitt
- admin
- Daniel Habermann
- Ullrich Köthe
- Paul-Christian Bürkner
- Stefan T. Radev

# Author notes (optional)
# author_notes:
# - "Equal contribution"
# - "Equal contribution"

date: "2023-02-11T00:00:00Z"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2023-02-01T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["1"]

# Publication name and optional abbreviated publication name.
publication: Pre-print
publication_short:

abstract: We propose a method to improve the efficiency and accuracy of amortized Bayesian inference by leveraging universal symmetries in the joint probabilistic model of parameters and data. In a nutshell, we invert Bayes' theorem and estimate the marginal likelihood based on approximate representations of the joint model. Upon perfect approximation, the marginal likelihood is constant across all parameter values by definition. However, errors in approximate inference lead to undesirable variance in the marginal likelihood estimates across different parameter values. We penalize violations of this symmetry with a \textit{self-consistency loss} which significantly improves the quality of approximate inference in low data regimes and can be used to augment the training of popular neural density estimators. We apply our method to a number of synthetic problems and realistic scientific models, discovering notable advantages in the context of both neural posterior and likelihood approximation.

# Summary. An optional shortened abstract.
summary:

tags: []

# Display this page in the Featured widget?
featured: true

# Custom links (uncomment lines below)
# links:
# - name: Custom Link
#   url: http://example.org

# url_pdf: 'https://arxiv.org/pdf/2302.14015.pdf'
#url_code: 'https://github.com/desi-ivanova/idad'
#url_dataset: ''
#url_poster: ''
#url_project: ''
#url_slides: ''
#url_source: 'https://nips.cc/Conferences/2021/ScheduleMultitrack?event=26928'
#url_video: ''

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
image:
  caption: "" #Image credit: [**Unsplash**](https://unsplash.com/photos/pLCdAaMFLTE)
  focal_point: Right
  # preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects:
- []

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
slides: ""
---
