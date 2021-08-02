---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Using coursera-dl to download copies of Coursera courses"
subtitle: ""
summary: ""
authors: []
tags: ["Python", "Web", "learning", "MOOC"]
categories: []
date: 2020-11-07T23:09:54Z
lastmod: 2020-11-07T23:09:54Z
featured: false
draft: true

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
I think [Coursera](https://www.coursera.org/) is great for learning new research techniques. There's some amazing courses and specialisations (groups of related courses from a single provider) on their for statistics, programming, machine learning, and medical informatics, and because it's charged per month for access (with persistent access for free after completion of a course) it's really not unreasonably priced if you've got a chunk of time to sit down and work on a project.

However, I'm still a bit of a cloud-skeptic, and like to have hard copies of things, especially things I've paid for, on my laptop for working on when I'm subject to dodgy NHS wifi and on my trusty backup NAS due to my paranoia that they could just disappear. Thankfully I'm not the only person with this concern regarding Coursea, and a much more talented programmer than I put together [coursera-dl](https://github.com/coursera-dl/coursera-dl). It's not been updated for 11 months, and Coursera keeps updating their security preferences to avoid people using it, but this is how I got it functioning:

Firstly, install using Python3. You may need to update your Python distribution on some older OSs, or if you've never had Pyton I heartily recommend [the Anaconda distribution](https://www.anaconda.com/products/individual) for all your Pythonic, data scientific needs.

```
pip install coursera-dl
```

Next you need to update a few lines in the coursera-dl code
