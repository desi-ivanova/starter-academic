---
title: ""

# Authors
# If you created a profile for a user (e.g. the default `admin` user), write the username (folder name) here
# and it will be replaced with their full name and linked to their profile.
authors:
- Jake Fawkes 
- Lucile Ter-Minassian
- admin
- Uri Shalit
- Chris Holmes

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
publication: ICML
publication_short:

abstract: Merging datasets across institutions is a lengthy and costly procedure, especially when it involves private information. Data hosts may therefore want to prospectively gauge which datasets are most beneficial to merge with, without revealing sensitive information. For causal estimation this is particularly challenging as the value of a merge will depend not only on the reduction in epistemic uncertainty but also the improvement in overlap. To address this challenge, we introduce the first cryptographically secure information-theoretic approach for quantifying the value of a merge in the context of heterogeneous treatment effect estimation. We do this by evaluating the Expected Information Gain (EIG) and utilising multi-party computation to ensure it can be securely computed without revealing any raw data. As we demonstrate, this can be used with differential privacy (DP) to ensure privacy requirements whilst preserving more accurate computation than naive DP alone. To the best of our knowledge, this work presents the first privacy-preserving method for dataset acquisition tailored to causal estimation. We demonstrate the effectiveness and reliability of our method on a range of simulated and realistic benchmarks.

# Summary. An optional shortened abstract.
summary:

tags: []

# Display this page in the Featured widget?
featured: false

# Custom links (uncomment lines below)
# links:
# - name: Custom Link
#   url: http://example.org

url_pdf: 'https://arxiv.org/pdf/2409.07215'
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
