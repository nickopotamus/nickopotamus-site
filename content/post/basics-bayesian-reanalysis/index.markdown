---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Basics Bayesian Reanalysis"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2021-08-27T10:27:59+01:00
lastmod: 2021-08-27T10:27:59+01:00
featured: false
draft: false
output: html_document

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

[BaSICS](https://jamanetwork.com/journals/jama/fullarticle/2783039) was kind of a big deal earlier this month. You may have [read about it on Twitter](https://twitter.com/search?q=%23basics%20fluid&src=typed_query&f=live) when al the excitement was being live-streamed…

![I’m kinda a big deal](/post/basics-bayesian-reanalysis/big_deal.gif)

[The choice of which fluid t use and when](https://www.youtube.com/watch?v=mUFouvB9Fa4) is an argument that intensivists love to have, and [multiple studies](https://doi.org/10.1177%2F1060028019866420) seemed to suggest that using balanced crystalloids (fluids which more closely mimic the composition of blood plasma) have better outcomes in critically unwell adults than the “old faithful” of 0.9% sodium chloride ([“abnormal” saline](https://www.nature.com/articles/s41581-018-0008-4)), which has an abnormally high chloride load ([and hence is acidotic](https://emcrit.org/wp-content/uploads/2016/05/27140683_-Stewart-Acid-Base_-A-Simplified-Bedside-Approach.pdf)) and lacks any potassium, calcium, or other electrolytes.

Then along came BaSICS. Aiming to randomise 11,000 critically ill patients to brine vs a balanced solution (in this case, [Plasma-Lyte 148](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5109922/), beloved of neurointensivists), it reported recently [via a hyped lifestream on the amazing Critical Care Reviews](https://criticalcarereviews.com/livestream/basics) that there was no difference between mortality rates, or rates of acute kidney injury requiring renal replacement therapy, between the two (p = 0.47). Cue the “balanced cyrstalloids are pointless” [editorials](https://rebelem.com/the-basics-trial-balanced-solution-vs-0-9-saline-solution-in-critically-ill-patients/)…

Hmmm. An arbitrary p-value. Sounds like an excuse for yet more [Bayesian statistics](/post/bayesian-statistics-for-doctors/) chat.

![We don’t do that here](/post/basics-bayesian-reanalysis/do_that.gif)

A couple of months ago Arthur Albuquerque, a medical students and Bayesian stats machine, published a [fantastic Bayesian renalaysis](https://www.medrxiv.org/content/10.1101/2021.06.15.21258966v1) of the [RECOVERY trial’s toilizumab results](https://www.recoverytrial.net/results/tocilizumab-results), so we’ll borrow his code and do the same thing for BaSICS. For simplicity, we’ll simply look at mortality in all patients, but AKI and/or subgroup analysis would be performed the same way.

## Get some data

We’ll start with importing packages we need:

``` r
library(tidyverse)  # Data wrangling and plotting
library(here)       # To find file paths within R project
library(janitor)    # To change quickly change names of columns
library(metafor)    # To create priors from meta-analysis
library(ggdist)     # To plot distributions
library(patchwork)  # To arrange plots
library(gt)         # To create nice tables
library(kableExtra) # To create nice tables
```

We’ll borrow the data from [Hammond et al’s excellent pre-BaSICS meta-analysis](https://doi.org/10.1177%2F1060028019866420), focusing just on the randomised controlled trials and just the mortality outcomes, and add BaSICS to it.

``` r
d = read_csv("extracted-data.csv")
d = clean_names(d) # to change the names of columns

d %>% 
  filter(outcome == "mortality") %>% 
  kable(format = "markdown") %>% 
  kable_styling()
```

| trial         | outcome   | events\_balanced | total\_balanced | events\_control | total\_control |
|:--------------|:----------|-----------------:|----------------:|----------------:|---------------:|
| Annane 2013   | mortality |               22 |              72 |             275 |           1035 |
| Semler 2017   | mortality |               72 |             520 |              68 |            454 |
| Semler 2018   | mortality |              928 |            7942 |             975 |           7860 |
| Verma 2016    | mortality |                5 |              33 |               2 |             34 |
| Young 2014    | mortality |                3 |              32 |               4 |             33 |
| Young 2015    | mortality |               87 |            1152 |              95 |           1110 |
| Zampieri 2021 | mortality |             1381 |            5230 |            1439 |           5290 |

## Calculate posterior distributions

First step in Bayesianing this up is working out the posterior distribution - what does BaSICS tell us, and all the other trials that Hammond analyses, tell us in isolation.

``` r
# Log(odds ratio) for BASICS
logOR_basics_mortality =
  rma(
    # Outcome
    measure = "OR",
    
    # Balanced crystalloid group
    ai = events_balanced,
    n1i = total_balanced,
    
    # Control group
    ci = events_control,
    n2i = total_control,
    
    # Mechanics
    data = d %>% filter(outcome == "mortality",
                        trial == "Zampieri 2021"),
    method = "REML",
    slab = paste(trial)
  )

# Log(odds ratio) for all other trials
logOR_prior_mortality =
  rma(
    # Outcome
    measure = "OR",
    
    # Balanced crystalloid group
    ai = events_balanced,
    n1i = total_balanced,
    
    # Control group
    ci = events_control,
    n2i = total_control,
    
    # Mechanics
    data = d %>% filter(outcome == "mortality",
                         trial != "Zampieri 2021"),
    method = "REML",
    slab = paste(trial)
  )
```

We also need to build the prior distributions - what we knew (or thought) before BaSICS was on the scene. As in the Albuquerque, we’ll use a number of priors to determine the strength of the outcomes after BaSICS - the “evidence based prior” (i.e. what we think after Hammond’s summary of the evidence), an “optimistic” prior (what the authors were hoping for, which was a 10% reduction in the odds ratio[^1]), a “pessimistic” prior (so the opposite, symmetrically opposite the optimisitc), and a skeptical (no difference) and non-informative (no information at all) prior. To do this we’ll re-use [Albuquerque’s scripts, available from Github](https://github.com/arthur-albuquerque/tocilizumab_reanalysis/tree/master/final_analyses/script) but modified for our ORs:

[^1]: Technically the hazard ratio, but it’s close to 1 so we can assume HR \~ OR.

``` r
# Normal conjugate posterior probabilities
source("conjugate_normal_posterior.R")
# Generate posterior probabilities with an evidence-based prior
source("normal_approximation_logOR.R")
# Generate posterior probabilities with multiple priors
source("logOR_normal_approximation_multiple_priors.R")
# Compile all posteriors into a tibble
source("tibble_all_posteriors_logOR.R") 
```

Now we can build our prior distributions:

``` r
# Build evidence-based priors
posterior_mortality = normal_approximation_logOR(
  logOR_basics_mortality, # Data object with BASICS data
  logOR_prior_mortality,  # Output from the rma() in the previous code chunk
  posterior_mortality
)

# Build other priors
posteriors_mortality = logOR_normal_approximation_multiple_priors(
    posterior_mortality, # Data object with evidence-based posterior from normal_approximation()
    "mortality",         # Outcome (within quotes)
    posteriors_mortality # Name for the final output with data+prior+posterior
  )

# Calculate OR for posterior distribution 
samples_logOR_posteriors_mortality = 
  tibble_all_posteriors_logOR(posteriors_mortality)

samples_posteriors_mortality = 
  samples_logOR_posteriors_mortality %>% 
  pivot_longer(
    cols = c('Non-informative':'Pessimistic'),
    names_to = "Underlying_Prior", values_to = "log(OR)"
  ) %>% 
  mutate(OR = exp(`log(OR)`))
```

``` r
# View them
posteriors_mortality %>% kable(format = "markdown") %>%  kable_styling()
```

| type            |   data.mean |   data.sd | prior.mean |    prior.sd |  post.mean |   post.sd |
|:----------------|------------:|----------:|-----------:|------------:|-----------:|----------:|
| non-informative | -0.04062107 | 0.0440286 |  0.0000000 | 100.0000000 | -0.0406211 | 0.0440286 |
| evidence-based  | -0.04062107 | 0.0440286 | -0.0653904 |   0.0444135 | -0.0528979 | 0.0312681 |
| skeptical       | -0.04062107 | 0.0440286 |  0.0000000 |   0.0444135 | -0.0204873 | 0.0312681 |
| optimistic      | -0.04062107 | 0.0440286 | -0.1053605 |   0.0444135 | -0.0727091 | 0.0312681 |
| pessimistic     | -0.04062107 | 0.0440286 |  0.1053605 |   0.0444135 |  0.0317344 | 0.0312681 |

## Visualise some results

So what does all this mean? Remember that a Bayesian analysis is taking your prior beliefs (in this case, either the evidnece based belief from the Hammond paper, or the pre-specified other priors), and applying the likelihood from the new results (BaSICS) to update your posterior beliefs. Let’s visualise this.

Firstly, we’ll extract the priors…

``` r
# Extract priors
priors_mortality =
  d %>%
  filter(
    trial != "Zampieri 2021",
    outcome == "mortality"
  )

basics_mortality =
  d %>%
  filter(
    trial == "Zampieri 2021",
    outcome == "mortality"
  )

# Set seed for reproducibility
set.seed = 69 #dudes
```

… and combine these into a dataframe of distributions…

``` r
### Create distributions dataframe ----
distributions =
  posteriors_mortality %>%
  filter(type == "evidence-based") %>%
  summarise(
    BASICS = list(rnorm(10e4,
                        mean = data.mean,
                        sd = data.sd
    )),
    Prior = list(rnorm(10e4,
                        mean = prior.mean,
                        sd = prior.sd
    ))
  ) %>%
  unnest(BASICS:Prior)

distributions = bind_cols(
  distributions,
  # Bind corresponding posterior distribution data
  samples_posteriors_mortality %>% 
    filter(`Underlying_Prior` == "Evidence-based") %>% 
    select(`log(OR)`) %>% 
    rename("Posterior" = `log(OR)`)
)
```

… which we can then plot (ggplot faffing skipped for brevity):

<img src="{{< blogdown/postref >}}index_files/figure-html/plot_one-1.png" width="480" style="display: block; margin: auto;" />

Finally, we can compare the effect of different priors (again, ggplot code skipped):

<img src="{{< blogdown/postref >}}index_files/figure-html/plot_priors_plot-1.png" width="480" style="display: block; margin: auto;" />

So how do you interpret this? The posterior distribution (probabilty of the real value) is the bell curve, and there’s one for each prior. The 95% [credible interval](https://en.wikipedia.org/wiki/Credible_interval) (kind of like the 95% confidence interval, but in this case actually giving you the probability of the real value, rather than the value given the null hypothesis) is shown as the dark line.

Holding the results in isolation (non-informative and skeptical priors) there’s no real benefit to be seen, although the probability leans towards balanced solutions being beneficial. However, using the evidence-based prior (based on what we knew before BaSICS), BaSICS actually narrows down the credible interval, providing *more* evidence that balanced crystalloids confer a survival benefit, and the same finding is seen given an optimistic prior (what the investigators powered the study around) - based on this, BaSICS again supports the use of balanced crystalloids.

In light of this then, all BaSICS has done is helped narrow down what we already knew - balanced crystalloids are probably better for the critically unwell. This then feeds into the critisicms of the paper (high number of elective - and hence “well critically unwell” patients, low numbers of septic patients, etc to suggest that actually this isn’t the fluids debate paper to end all fluids debates paper that people on Twitter think it is.

## One last thing…

More worryingly, BaSICS also seemed to find a signal for harm using balanced solutions in patients with traumatic brain injuries, so my next step will be looking into this in a bit more detail. Get in touch via the usual socials if you want to help out with this…
