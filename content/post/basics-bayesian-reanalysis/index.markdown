---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "A Bayesian reanalysis of the BaSICS trial"
subtitle: ""
summary: "Short version: saline is still evil. Probably."
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

[BaSICS](https://jamanetwork.com/journals/jama/fullarticle/2783039) was kind of a big deal earlier this month. You may have [read about it on Twitter](https://twitter.com/search?q=%23basics%20fluid&src=typed_query&f=live) when all the excitement was being live-streamed…

![I’m kinda a big deal](/post/basics-bayesian-reanalysis/big_deal.gif)

[The choice of which fluid to use and when](https://www.youtube.com/watch?v=mUFouvB9Fa4) is an argument that intensivists love, and [combining multiple studies](https://doi.org/10.1177%2F1060028019866420) seemed to suggest that using balanced crystalloids (fluids which more closely mimic the composition of blood plasma) have better outcomes in critically unwell adults than the “old faithful” of 0.9% sodium chloride ([“abnormal” saline](https://www.nature.com/articles/s41581-018-0008-4)), which has a high chloride load ([and hence is acidotic](https://emcrit.org/wp-content/uploads/2016/05/27140683_-Stewart-Acid-Base_-A-Simplified-Bedside-Approach.pdf)) and lacks any potassium, calcium, or other electrolytes.

Then along came BaSICS. Aiming to randomise 11,000 critically ill patients to medical brine vs a balanced solution (in this case, [Plasma-Lyte 148](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5109922/), beloved of neurointensivists), it reported recently [via alifestream on the amazing Critical Care Reviews website](https://criticalcarereviews.com/livestream/basics) that there was no difference between mortality rates, or rates of acute kidney injury requiring renal replacement therapy, between the two (p = 0.47). Cue the “balanced crystalloids are pointless” [editorials](https://rebelem.com/the-basics-trial-balanced-solution-vs-0-9-saline-solution-in-critically-ill-patients/)…

Hmmm. A yes/no answer supported by [an arbitrary p-value](https://royalsocietypublishing.org/doi/10.1098/rsbl.2019.0174)? Sounds like an excuse for yet more [Bayesian statistics](/post/bayesian-statistics-for-doctors/) chat.

![We don’t do that here](/post/basics-bayesian-reanalysis/do_that.gif)

A couple of months ago [Arthur Albuquerque](https://twitter.com/arthur_alb1), a medical student and Bayesian stats wizard, published a [fantastic Bayesian renalaysis](https://www.medrxiv.org/content/10.1101/2021.06.15.21258966v1) of the [RECOVERY trial’s toilizumab results](https://www.recoverytrial.net/results/tocilizumab-results), so we’ll borrow his code and do the same thing for BaSICS. For simplicity, we’ll simply look at mortality in all patients, but AKI and/or subgroup analysis would be performed the same way.

## Get some data

We’ll start with importing packages we need:

``` r
library(tidyverse)  # Data wrangling and plotting
library(janitor)    # To change quickly change names of columns
library(metafor)    # To create priors from meta-analysis
library(ggdist)     # To plot distributions
library(patchwork)  # To arrange plots
library(kableExtra) # To create nice tables
```

We’ll borrow the data from [Hammond et al’s excellent pre-BaSICS meta-analysis](https://doi.org/10.1177%2F1060028019866420), focusing just on the randomised controlled trials and just the mortality outcomes, and add the outcomes from BaSICS to it.

``` r
d = read_csv("extracted-data.csv")
d = clean_names(d) 
```

| trial         | events\_balanced | total\_balanced | events\_control | total\_control |
|:--------------|-----------------:|----------------:|----------------:|---------------:|
| Annane 2013   |               22 |              72 |             275 |           1035 |
| Semler 2017   |               72 |             520 |              68 |            454 |
| Semler 2018   |              928 |            7942 |             975 |           7860 |
| Verma 2016    |                5 |              33 |               2 |             34 |
| Young 2014    |                3 |              32 |               4 |             33 |
| Young 2015    |               87 |            1152 |              95 |           1110 |
| Zampieri 2021 |             1381 |            5230 |            1439 |           5290 |

## Calculate distributions

First step in Bayesian-ing this up is working out the posterior distributions (what do BaSICS tell us, and all the other trials that Hammond analyses, tell us in isolation) and the prior distributions (what did we think before BaSICS).

Lets start with working out the odds ratio for both BaSICS, and everything else combined:

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

So now we build a range of prior distributions - what we knew (or thought) before BaSICS was on the scene. As in the Albuquerque paper, we’ll use a number of priors to determine the strength of the outcomes after BaSICS. These include:

-   The “evidence based prior” (i.e. what we think after Hammond’s summary of the evidence)
-   An “optimistic” prior (what the authors were hoping for, which was a 10% reduction in the odds ratio[^1])
-   A “pessimistic” prior (so the opposite of the optimistic, that balanced crystalloids cause harm)
-   A skeptical (no difference, but same variance as the evidence-based prior) and non-informative (no information at all) prior.

To do this we’ll re-use Albuquerque’s scripts, [available from Github](https://github.com/arthur-albuquerque/tocilizumab_reanalysis/tree/master/final_analyses/script), modified for our ORs:

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

Now we can build our prior distributions, and calculate the posterior distributions from applying the results of BaSICS to each of these:

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

# Calculate OR for posterior distributions based on the priors 
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

| type            |   data.mean |   data.sd | prior.mean |    prior.sd |  post.mean |   post.sd |
|:----------------|------------:|----------:|-----------:|------------:|-----------:|----------:|
| non-informative | -0.04062107 | 0.0440286 |  0.0000000 | 100.0000000 | -0.0406211 | 0.0440286 |
| evidence-based  | -0.04062107 | 0.0440286 | -0.0653904 |   0.0444135 | -0.0528979 | 0.0312681 |
| skeptical       | -0.04062107 | 0.0440286 |  0.0000000 |   0.0444135 | -0.0204873 | 0.0312681 |
| optimistic      | -0.04062107 | 0.0440286 | -0.1053605 |   0.0444135 | -0.0727091 | 0.0312681 |
| pessimistic     | -0.04062107 | 0.0440286 |  0.1053605 |   0.0444135 |  0.0317344 | 0.0312681 |

## Visualise some results

So what does all this mean? Remember that a Bayesian analysis is taking your prior beliefs (in this case, either the evidence based belief from the Hammond paper, or the pre-specified other priors), and applying the likelihood from the new results (BaSICS) to update your posterior beliefs. Let’s visualise this.

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

… which we can then plot ([ggplot](https://ggplot2.tidyverse.org/) faffing skipped for brevity):

<img src="{{< blogdown/postref >}}index_files/figure-html/plot_one-1.png" width="960" style="display: block; margin: auto;" />

## What does it all mean?

So how do we interpret this?

The probability distribution (probability of the real value, given the data available) is shown as a bell curve. The 95% [credible interval](https://en.wikipedia.org/wiki/Credible_interval) (kind of like the 95% confidence interval, but in this case actually giving you the probability of the real value, rather than the range of values given the null hypothesis) is shown as the dark line.

The prior, pink, shows the range of potential odds ratios given the pre-existing evidence base. The likelihood of the true odds ratio from the BaSICS data is in green, and the posterior distribution (the prior, updated by the new data) is in red, seeming to show that BaSICS in fact confirmed our pre-existing suspicions that balanced crystalloids are better.

But what is we think the evidence base on which BaSICS was designed is flawed? Well, we can compare the effect of different priors (again, ggplot code skipped, but available on Github):

<img src="{{< blogdown/postref >}}index_files/figure-html/plot_priors_plot-1.png" width="960" style="display: block; margin: auto;" />

Holding the results of BaSICS in isolation (non-informative and skeptical priors) seems to show no conclusive benefit to using balanced crystalloids, although the probability does lean towards balanced solutions being beneficial - more data needed. However, using the evidence-based prior (based on what we knew before BaSICS), BaSICS actually narrows down the credible interval, providing *more* evidence that balanced crystalloids confer a survival benefit, and the same finding is seen given an optimistic prior (that the BaSCIS investigators powered their study around). So based on these, BaSICS again supports the use of balanced crystalloids.

In light of this then, what BaSICS has done is helped narrow down what we already knew - balanced crystalloids are (probably) better for the critically unwell. This then feeds into [the other criticisms of the paper](https://www.thebottomline.org.uk/summaries/icm/basics/) (high number of elective - and hence “well critically unwell” patients, low numbers of septic patients, etc) to suggest that actually that although a great paper, especially given the number of patients collected in the middle of a COVID-19 pandemic, this isn’t the fluids debate paper to end all fluids debates paper that Twitter seems to think it is.

## One last thing…

More worryingly, BaSICS also seemed to find a signal for harm using balanced solutions in patients with traumatic brain injuries, so my next step will be looking into this in a bit more detail. This will require a bit more digging into the literature, and the oversight of a proper statistician, so get in touch if you want to help out with this…
