---
layout: post
title: Inverse-Weighted Survival Games
author: Mark Goldstein
permalink: games.html
---

* * * 

This post is about Survival Analysis (the modeling of time-to-event distributions), 
and about our recent NeurIPS 2021 work [Inverse-Weighted Survival Games](https://arxiv.org/abs/2111.08175)

![games](assets/img/games.png)

(*For those not familiar with survival modeling, you will be in just a few moments by reading my* 
<a href="./survival.html">Survival Analysis Intro</a>!)

The work presents a new method, *IPCW Games*, for estimating survival
distributions that is an alternative to
maximum likelihood estimatation (MLE). The motivation
for IPCW Games is based on these three points:

- survival models are often evaluated on crtieria like 
Brier Score (BS), Bernoulli Log Likelihood (BLL), and Calibration

- MLE for survival analysis is proper/consistent and asymptotically efficient, but on finite sample
it does not always yield good models with respect to the evaluations

- Optimizing directly for the above evaluations can improve performance on small datasets

In this work we explore the challenges involved in the third point (in short, dealing with
re-weighting estimators, see below) and the benefits of doing so:  

- shares certain good properties with MLE (that it still yields the true failure distribution)

- achieves better performance than MLE does empirically on small datasets

Most interestingly, unlike the typical MLE approach, our method makes use of the censoring distribution
(the distribution that causes missingness of times-to-event, see <a href="./survival.html">survival intro</a>). The estimation
method we present takes the form of a *game* between a *failure model player* and *censoring model player*.

Zooming out for a moment, this work also provides a positive example to a question our group has been asking recently:
**when can modeling the censoring/missingness mechanism in missing-data problems help in the primary estimation problem?**


## Brief recap of survival setup
* * *

For each patient $$X$$, 
we are interesting in modeling the time-to-event (called the *failure time*) $$T|X$$ with CDF $$F$$.
There is also a censoring time $$C|X$$ with CDF $$G$$ for each patient.
Under right-censoring, we only observe $$X, U=\min(T,C)$$ and $$\Delta=1[T \leq C]$$ instead of $$T,C$$.
We assume i.i.d. data and random censoring $$T \perp C | X$$. Finally, let $$\overline{F}=1-F$$
and $$\overline{G}=1-G$$ and let $$\neg \Delta=1-\Delta$$.

Under these assumptions, we now construct the typical failure model likelihood. Suppose we have failure model 
$$F_\theta$$ with density $$f_\theta$$. Then, under the assumptions, the likelihood can be written as

$$L(\theta) = \prod_i f_\theta(U_i|X_i)^{\Delta_i}\overline{F}_\theta(U_i|X_i)^{\neg \Delta_i}$$

The intuition for optimizing this expression is that, for patients with observed failure time $$U$$ ($$\Delta=1$$ means $$U_i=T_i$$),
we increase the model density at that time. Otherwise, when $$\Delta=0$$ we only know that $$T > U$$ meaing we should
maximize $$P_\theta(T > U|X) = 1 - P_\theta(T \leq U | X) = 1 - F_\theta(U|X) = \overline{F}_\theta(U|X)$$.


## Evaluating Survival Models
* * *

Beyond likelihood, survival models are evaluated under several variants of metrics like Concordance and Brier Score (BS). 
Concordance measures the fraction of datapoints whose modeled risks are ordered correctly with respect to their true failure times.
Brier Score picks a particular time $$t$$ and considers the classification problem of whether or not a datapoint fails by $$t$$. It does so
by measuring the average squared error between the event status and modeled probability:

$$BS(t;\theta) = \mathbb{E} \Big[ \Big( F_{\theta}(t|X) - 1[T \leq t] \Big) ^2 \Big]$$ 

where $$F_{\theta}$$ is a model for the failure CDF $$F$$. We focus on BS and discuss why we do not focus on concordance at the end of this post.

Censoring makes it challenging to compute $$BS(t;\theta)$$.
For patients censored before time t, we do not observe $$1[T \leq t]$$. However, if the true censoring distribution were to be known,
we could use re-weighting (see section TODO in the paper) to estimate the BS consistently:

$$\begin{align*}
BS(t;\theta) &=  \mathbb{E} \Big[ 
									\frac{\overline{F_\theta}(t|X)^2 \Delta 1[U \leq t]}{\overline{G}(U^{-}|X)}								
							+
									\frac{F_\theta(t|X)^2 1[U > t]}{\overline{G}(t|X)}
							\Big] 
\end{align*}
$$
 
This expectation equals the previous definition of $$BS(t;\theta)$$ but only samples from observed data $$X,U,\Delta$$.

![grad plot](assets/img/grad_plot.png)


![min plot](assets/img/min_plot.png)


![conc vs nll](assets/img/conc_vs_nll.png)


## IPCW BS GAME
* * *

todo todo todo 

## Experiments

* * * 

todo todo todo 








