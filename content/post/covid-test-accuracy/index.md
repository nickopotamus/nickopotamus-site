---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "How accurate is my COVID-19 test?"
subtitle: "Sensitivity, specificity, and precision explained using COVID-19 tests. And of course, the answer involves Bayesian statistics"
summary: "And of course the answer involves Bayesian statistics"
authors: []
tags: []
categories: []
date: 2021-01-23T11:13:37Z
lastmod: 2021-01-23T11:13:37Z
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
Late yesterday afternoon, while trying to drag my [night-shift addled brain](https://link.springer.com/article/10.1007/s00134-015-4115-4) through a meeting with some collaborators, I got a call from my kids' nursery - they both had temperatures, and one was coughing, so it was time to [take them for a test](https://www.nhs.uk/conditions/coronavirus-covid-19/testing-and-tracing/).

> Presumably this is the more accurate test than the nasal pregnancy one?

Thankfully the results came back negative, and now I've got the same snotty febrile illness as them and my twice-weekly [lateral flow test](https://www.birmingham.gov.uk/info/50231/coronavirus_covid-19/2308/covid-19_lateral_flow_device_lfd_testing_information) is also negative, but based on that text on the [family WhatsApp group](https://www.vice.com/en/article/3a8v99/how-to-navigate-family-whatsapp-group-disagreements) it got me thinking about what the "accuracy" of all these tests actually means.

## Accuracy vs precision

There's a few steps to take before we can really answer that questions. Firstly, what do we actually mean by accuracy? [Wikipedia](https://en.wikipedia.org/wiki/Accuracy_and_precision) isn't much help:

> Accuracy has two definitions:
>1. More commonly, it is a description of systematic errors, a measure of statistical bias; low accuracy causes a difference between a result and a "true" value. ISO calls this trueness.
>2. Alternatively, ISO defines accuracy as describing a combination of both types of observational error above (random and systematic), so high accuracy requires both high precision and high trueness.

Essentially an **accurate** test is one that's *close to "truth"* (at least on a frequentest basis, so it averages out as truth), and a **precise** test is one which *repeatedly returns the same result*. The classical analogy for this is archery - an archer can be accurate (on average hit the target) but not precise (shoot arrows all over the shop), or they can be precise (hit the same spot over and over again) yet not accurate (never hitting the target), or any combination of these.

![](/post/covid-test-accuracy/targets.png)

Clearly the ideal archer is both accurate and precise. But for them the tatget is fairly obvious - when talking about medical testing then we have to define what the "target" for the test, or more accurately the "truth" that we're trying to uncover, is.


## What do we mean by "accuracy" anyway?

For any test, there's going to be two groups of people taking the test (the "truth") - some with the disease, and some without - and two outcomes from the test - positive, or negative. 

The *perfect* test would label 100% of those with the disease as positive, and 100% of those without the test as negative. But in reality, tests aren't binary like that. Tests look for something $X$ - a chemical, or bit of virus, or whatever - that has a *distribution* or spread of values in both healthy (blue) and diseased (red) people.

![](/post/covid-test-accuracy/cutoffs.png)

Really high values are clearly true positives, and really low clearly true negatives, but due to both random distribution within people and systematic testing errors, there's always a number of false positives (FP, people who don't have the disease, but are labeled positive) and false negatives (FN, people labeled negative, but who do have the disease) in the middle ground where the distribution of thing $X$ in the healthy and diseased overlaps. We can sum these numbers up in a *confusion matrix* comparing what we predict from the test against the "truth".

![](/post/covid-test-accuracy/confusion.png)

Making the test "accurate" then involves moving the cutoff value of $X$ for defining what is a positive and what is a negative result:

* We can move the cutoff to the left, towards lower values of $X$, which would capture more of the true positives (reducing false negatives) but also increase the rate of false positives.
* Wwe could shift it right, towards higher values of $X$, and so increase the rate at which we correctly diagnose people as healthy (reducing false positives), but also miss more of the diseased (increasing false negatives).

So "accuracy" depends on what we're trying to do - are we trying to detect as many people as possible with the disease, or as many people as possible who don't have the disease? And necessarily making the test more accurate in one domain *decreases* accuracy in the other

We can summarise these two concepts mathematically into **sensitivity** (also called recall in the data science world, or the true positive rate) and **specificity** (or true negative rate): 

$$ Sensitivity = \frac{TP}{TP+FN} $$ 
$$ Specificity = \frac{TN}{TN+FP} $$

A **sensitive** very sensitive test would pick up lots of true positives, minimising false negatives. However this also means it would pick up lots of false positives. Knowing that you're negative on a sensitive test would therefore be reassuring (it's pretty definite that you don't have the disease), but testing positive has is less meaningful. A sensitive test is therefore a great **rule-out test** for when it's very important that you find all the people who might possible have the disease.

The opposite to this is a very **specific** test which would pick up lots of true negatives, minimising false positive, but then would also pick up lots of false negatives. This makes it a **rule-in test**, so a positive means it's definitely the disease in question, but a negative could still mean that you have the disease. 

So a very specific test clears the most healthy people as it can, but doesn't necessarily pick up all the disease, whereas a sensitive test finds as much disease as it can, but doesn't necessarily reassure the healthy. How do we decide which to preference? Well a lot depends on the above, do we want need a rule-out or a rule-in test, but you'll commonly see [receiver operating characteristic](https://www.ahajournals.org/doi/full/10.1161/CIRCULATIONAHA.105.594929) (or ROC) curves in the literature which plot 1-specificity (which equals the false positive rate) on the x-axis against sensitivity on the y-axis:

![](/post/covid-test-accuracy/roc.png)

These curves are generated by altering that cut-off for $X$ in the test as discussed above, and plotting the specificity and sensitivity for each value of the cut-off. Clearly the ideal test would have 100% sensivity, and 100% specificity (i.e. 1-specificity = 0), as marked by the dark blue dot, but in reality they plot curves. The closer the curve to the dark blue dot, the better the performance of the test (the test plotting blue line is better than the one plotting the orange curve), and the cut-off value for the best test is usually chosen as the value matching the point on that curve which maximises sensitivity while minimising (1-specificity), although as discussed above you can trade off one against the other if you have a particular goal of the test.

There's one final bit of maths we need, and that is the **precision** (or positive predictive value) of the test:

$$ Precision = \frac{TP}{TP + FP} $$

The more precise the test, the more often a "positive" result is actually identifying a diseased rather than a healthy person.


## Accuracy of available COVID tests

There's a number of ways to look for COVID-19 (or more precisely the presence of SAR-CoV-2). The two biggies in common use are:
* [Lateral flow tests](https://publichealthmatters.blog.gov.uk/2020/12/08/lateral-flow-testing-new-rapid-tests-to-detect-covid-19/). These work and look similar to a pregnancy test, but use nose goo rather than urine. They were [trialed as rapid screening for the public](https://www.bbc.co.uk/news/explainers-54872039) (see later for how well that "worked"), and are currently used as [screening before visiting to nursing homes](https://www.gov.uk/government/publications/coronavirus-covid-19-lateral-flow-testing-of-visitors-in-care-homes) and [to try and keep health workers from spreading COVID around hospitals](https://www.england.nhs.uk/coronavirus/publication/asymptomatic-staff-testing-for-covid-19-for-primary-care-staff/)
* The "gold standard" [reverse transcriptase polymerase chain-reaction](https://en.wikipedia.org/wiki/Reverse_transcription_polymerase_chain_reaction) (RT-PCR) test, which looks for the presence of viral DNA. This is the technology used on the nasal swabs that get sent away by the "NHS"/"PHE"/[private-sector-money-making-scheme](https://www.ft.com/content/5dc5184e-0f6c-47db-a730-d173539c2d36) test-and-trace centres, and used in hospitals to confirm the diagnosis (both through nasal swabs and deep samples taken from the lungs).

So how do the different tests stack up?

| Test | Sensitivity % | Specificity % |
|-|-|-|
| [RT-PCR nasal swab](https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/895843/S0519_Impact_of_false_positives_and_negatives.pdf) | ">95"" | ">95" |
| [Innova lateral flow test](https://www.ox.ac.uk/news/2020-11-11-oxford-university-and-phe-confirm-lateral-flow-tests-show-high-specificity-and-are) | 76.8 | 99.68 |
| _Other diagnostic tests_ |
| [SAMBA-II "rapid" test](https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/941767/technical-validation-report-SAMBA-II.pdf) (used in some Emergency Departments for rapid screening) | 92.98-100 | 87.23-100 |
| [CovidNudge realtime PCT](https://www.bmj.com/content/370/bmj.m3682) | 94 | 100 |
| [OptiGene RT-LAMP test](https://www.gov.uk/government/publications/rapid-evaluation-of-optigene-rt-lamp-assay-direct-and-rna-formats/rapid-evaluation-of-optigene-rt-lamp-assay-direct-and-rna-formats) | 79-95 | 99-100 |
| [CT scan of the chest](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7350782/) | 89.8-93.7 | 21.0-29.5 |
| _Antibody tests_ (for previous infection) |
| [Roche antibody test](https://www.bmj.com/content/369/bmj.m2066) | 83.87-100 | 100 |
| [Abbott antibody test](https://www.bmj.com/content/369/bmj.m2066) | 87.5-93.9 | 99.63 |
| [Rapid fingerprick antibody test](https://www.bmj.com/content/371/bmj.m4262) ([AbC-19](https://www.abc19.com/)) | 84.7-92.5 | 97.9 |

All the tests are **very specific** (with the exception of chest CT), meaning that a positive test is almost definitely COVID-19. But this also means that the chance of a false negative, missing a less obvious COVID, is pretty sporting too. The relatively **low sensitivity** for the lateral flow test is particularly worrying, making it a poor choice as a rule-out test - a negative doesn't really mean it's a negative, so should people really be going to work with or visit the vulnerable based on its result?

So one could argue that the message in my family WhatsApp group was right, the nasal PCR swab is more "accurate" in ruling out one of my kids having COVID-19, even through the real world sensitivity [seems a lot lower](https://www.medrxiv.org/content/10.1101/2020.04.16.20066787v2) probably due to people not using the test properly (a real swab feels like you're biopsying the brainstem, and how many members of the public will do that to themselves or their children?) or at [the wrong point in the disease time-course](https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/895843/S0519_Impact_of_false_positives_and_negatives.pdf) to pick up the virus from the nose or mouth. 

## Real world accuracy

All of these numbers are based on the person being tested actually having COVID-19, while only around 1:85 of the population does ([or at least did at Christmas](https://www.ons.gov.uk/peoplepopulationandcommunity/healthandsocialcare/conditionsanddiseases/bulletins/coronaviruscovid19infectionsurveypilot/24december2020)). What does this mean for the accuracy of these tests? And did you really think we'd get away without mentioning [Bayesian statistics](/post/bayesian-statistics-for-doctors/)?

Let's start by returning to our confusion matrix, and introduce the **postivie predictive value** (PPV) and **negative predictive value** (NPV), or the chances of a test positive or negative being meaningful:

|  | Disease+ | Disease- | |
|-|-|-|-|
| **Test+** | True+ (TP) | False+ (FP) | PPV = TP/(TP+FP) |
| **Test-** | False- (FN) | True- (TN) | NPV = TN/(FN+TN) |
| | Sens = TP/(TP+FN) | Spec = TN/(TN+FP) | |

We can also derive a few more values from this table now:

| Condition | Value | Meaning |
|-|-|-|
| Sensitivity | TP/(TP+FN) | True positive rate |
| Miss rate | FN/(TP+FN) | False negative rate, 1-sensitivity |
| Specificity | TN/(TN+FP) | True negative rate |
| Fall-out | FP/(TN+FP) | False positive rate, 1-specificity |
| Positive likelihood ratio (LR+) | Sensitivity/fall-out | Change in odds of disease+ if test+ |
| Negative likelihood ratio (LR-) | Specificity/miss rate | Change in odds of disease- if test- |
| Diagnostic odds ratio | LR+/LR- | Change in odds of disease- if test- |

**Diagnostic odds ratio** seems like something we've seen before, no? A likelihood ratio?

$$ P(posterior) = LR * P(prior)$$

Lets assume we're using community PCR swabs. Sensitivity is 90% (so for 90% of people with COVID, the test will be positive) and specificity is 95% (so for 95% of peoplem without COVID, the test will be negative). What's the probability that a positive test means that you have COVID?

It all depends on the probability of having COVID in the first place - $P(prior)$. For the sake of argument let's say thats 1%, so of 10,000 people to be tested today, 100 of them have COVID. 90% sensitivity means that 90 of these test positive, and 10 test negative (false negatives).  But worse, 9,900 of them _don't_ have COVID, yet with a 95% specificity 5% of these, of **495**, will recieve a false positive diagnosis. 

Accuracy of this test is 90+9,405 (TP + TN) / 10,000 = 95% (seems pretty good), yet only 15.% of those who tested positive actually have COVID!

$$p(COVID|+) = \frac{p(+|COVID)p(COVID)}{p(+|COVID)p(COVID)+p(+|!COVID)p(!COVID)} $$

Or put more simply:

$$p(COVID|+) = \frac{Sensitivity * COVID in popn}{Sensitivity * population risk + (1 - Specificity)(1 - population risk)}$$

Why did the lateral flow test [fail so badly when used to screen the public in Liverpool](https://www.bmj.com/content/371/bmj.m4848)?

https://www.thelancet.com/journals/lanepe/article/PIIS2666-7762(20)30002-8/fulltext
https://www.medrxiv.org/content/10.1101/2020.11.24.20236950v1
https://www.bmj.com/content/370/bmj.m3682
https://www.bmj.com/content/bmj/369/bmj.m1808.full.pdf
https://medical.mit.edu/covid-19-updates/2020/06/how-accurate-diagnostic-tests-covid-19
https://blogs.bmj.com/bmj/2021/01/12/covid-19-government-must-urgently-rethink-lateral-flow-test-roll-out/

