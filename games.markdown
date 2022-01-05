---
layout: post
title: Inverse-Weighted Survival Games
author: Mark Goldstein
permalink: games.html
---

* * * 


This post is about Survival Analysis (the modeling of time-to-event distributions), 
and about our recent NeurIPS 2021 work [Inverse-Weighted Survival Games](https://arxiv.org/abs/2111.08175)

(*For those not familiar with survival modeling, you will be in just a few moments by reading my* 
<a href="./survival.html">Survival Analysis Intro</a>!)

This work presents a new method of estimating survival distributions that makes use of the relationship
between the failure (outcome) distribution and censoring (missingness) distributions in survival problems.
While the typical maximum-likelihood based approach usually drops
estimation of the censoring distribution, here we purposely do it and show how it can help!

The method centers around a *game* between two players, the models of the two distributions.
We characterize when these games have stationary points at the true data distributions 
show empirically that they produce survival models that do well (on several evaluations) in the small-data regime relative to standard maximum likelihood estimation, 
even under moderate censoring.

TODO should mention that likelihood doesnt perform well under criteria, but hard to optimize criteria, so inverse weighting, 
so our game featuring two distributions, gives true failure distribution. 

Zooming out for a moment, this work also provides a positive example to a question our group has been asking recently:
**when can modeling the censoring/missingness mechanism in missing-data problems help in the primary estimation problem?**

## Brief recap of survival setup
* * *

For each patient $$X$$, 
we are interesting in modeling the time-to-event (or *failure time*) $$T|X$$ with CDF $$F$$.
There is also some censoring time $$C|X$$ with CDF $$G$$ for each patient.
Under right-censoring, we only observe $$X, U=\min(T,C)$$ and $$\Delta=1[T \leq C]$$ instead of $$T,C$$.
We assume i.i.d. data and independent censoring $$T \perp C | X$$. Finally, let $$\overline{F}=1-F$$
and $$\overline{G}=1-G$$ and let $$\neg \Delta=1-\Delta$$.

Under these assumptions, we now construct the typical failure model likelihood. Suppose we have failure model 
$$F_\theta$$ with density $$f_\theta$$. Then, under the assumptions, the likelihood can be written as

$$L(\theta) = \prod_i f_\theta(U_i|X_i)^{\Delta_i}\overline{F}_\theta(U_i|X_i)^{\neg \Delta_i}$$

The intuition for optimizing this expression is that, for patients with observed failure time $$U$$ ($$\Delta=1$$ means $$U_i=T_i$$),
we increase the model density at that time. Otherwise, when $$\Delta=0$$ we only know that $$T > U$$ meaing we should
maximize $$P_\theta(T > U|X) = 1 - P_\theta(T \leq U | X) = 1 - F_\theta(U|X) = \overline{F}_\theta(U|X)$$.


## Evaluating Survival Models
* * *

concordance, bs, ipcw, ipcw bs 

attempt at optimizing sum of ipcw bs's

![grad plot](assets/img/grad_plot.png)


![min plot](assets/img/min_plot.png)

## IPCW BS GAME
* * *

todo todo todo 

## Experiments

* * * 

todo todo todo 








