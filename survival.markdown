---
layout: post
title: A Quick Introduction to Survival Analysis
author: Mark Goldstein
permalink: survival.html
---

* * * 

This post is a brief intro to the biostats and ML for health problem called Survival Analysis.

## What is Survival Analysis?
* * *

Survival Analysis is the modeling of time-to-event distributions and is widely used in biostats / ML for healthcare to predict the time until clinical events such
as hospital discharge, changes in level of care, or progression of disease. 

As a first approximation, one can think of survival modeling as like regression, except where we only predict positive numbers (time remaining until the event) instead any real number. However, what makes survival distinct is that outcomes are often not observed for all datapoints (in this context usually patients).

Consider this scenario: you run a 5 year heart disease study starting in 1990. You recruit 5 participants without heart disease for this study
and measure their features at time 0 (i.e. 1990). Two participants develop the disease at years 3 and 4. We will call such patients *uncensored*.
Two more participants leave the study in year 4 before developing disease. This could be, for example, because they may have moved away 
and we lost contact with them and could not follow-up about their disease status. The last participant stays for the whole study but does not develop disease.
We will call the latter 3 participants *censored*.

This is depicted below, in a graphic from [Haider et. al, 2020](https://jmlr.org/papers/volume21/18-772/18-772.pdf) with the hospital beds depicting uncensored patients with observed disease and stick figures over green lights depicting censored ones

![Survival](assets/img/haider.png)

For the 3 censored patients, we don't know if or when exactly they will develop disease. We only know that their time-to-disease is greater than their censoring time
(either when they left while still healthy or when the study ended).

**What makes survival analysis distinct from regression is the need to deal with censored data**. As is briefly discussed below, simply discarding
the censored datapoints can lead to biased estimates of the time-to-event distribution. Instead, assumptions about the censoring mechanism must be made clear
and then used to derive objectives/loss functions/estimation methods that accounts for censoring appropriately. Luckily, as discussed below, this is not so
difficult to do in many standard cases.

