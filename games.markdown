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
distributions, motivated by the following points:

- Survival models are often evaluated on criteria like 
Brier Score (BS), Bernoulli Log Likelihood (BLL), and Calibration

- Maximum Likelihood Estimation (MLE) for survival analysis is proper/consistent and asymptotically efficient, but on finite sample
it does not always yield good models with respect to the evaluations

- Optimizing directly for the above evaluations can improve performance on small datasets

In this work we explore the challenges involved in the third point (optimizing re-weighting estimators)
and the benefits targeting these evaluations directly:

- can share certain good properties with MLE (yielding the true failure distribution)

- can achieve better performance than MLE empirically on small datasets

Most interestingly, unlike the typical MLE approach, our method makes use of the censoring distribution
(the distribution that causes missingness of times-to-event, see <a href="./survival.html">survival intro</a>). The estimation
method we present takes the form of a *game* between a *failure model player* and *censoring model player*.

Zooming out for a moment, this work also provides a positive example to a question our group has been asking recently:
**when can modeling the censoring/missingness mechanism in missing-data problems help in the primary estimation problem?**


## Outline
* * *

- review survival analysis math setup 
- discuss the Brier Score as an evaluation and how to estimate it under censoring using inverse-weighting
- discuss why you cannot just optimize re-weighting estimators directly 
- present games based on Brier Score featuring failure and censoring models, which are a particular way to optimize re-weighted estimators
- briefly summarize some theoretical and empirical results comparing the games against MLE 


## Brief recap of survival setup
* * *

For each patient $$X$$, 
we are interesting in modeling the time-to-event (called the *failure time*) $$T|X$$ with CDF $$F$$.
There is also a censoring time $$C|X$$ with CDF $$G$$ for each patient.
Under right-censoring, we only observe $$X, U=\min(T,C)$$ and $$\Delta=1[T \leq C]$$ instead of $$T,C$$.
We assume i.i.d. data and random censoring $$T \perp C | X$$. Finally, let $$\overline{F}=1-F$$
and $$\overline{G}=1-G$$ and let $$\overline{\Delta}=1-\Delta$$.

Under these assumptions, we now construct the typical failure model likelihood. Suppose we have failure model 
$$F_\theta$$ with density $$f_\theta$$. Then, under the assumptions, the likelihood can be written as

$$L(\theta) = \prod_i f_\theta(U_i|X_i)^{\Delta_i}\overline{F}_\theta(U_i|X_i)^{\overline{\Delta_i}}$$

The intuition for optimizing this expression is that, for patients with observed failure time $$U$$ ($$\Delta=1$$ means $$U_i=T_i$$),
we increase the model density at that time. Otherwise, when $$\Delta=0$$ we only know that $$T > U$$ meaing we should
maximize $$P_\theta(T > U|X) = 1 - P_\theta(T \leq U | X) = 1 - F_\theta(U|X) = \overline{F}_\theta(U|X)$$.


## Evaluating Survival Models
* * *

Beyond likelihood, survival models are evaluated under variants of metrics like Concordance and Brier Score (BS). 
Concordance measures the fraction of datapoints whose modeled risks are ordered correctly with respect to their true failure times.
Brier Score picks a particular time $$t$$ and considers the classification problem of whether or not a datapoint fails by $$t$$. It does so
by measuring the average squared error between the event status and modeled probability:

$$BS(t;\theta) = \mathbb{E} \Big[ \Big( F_{\theta}(t|X) - 1[T \leq t] \Big) ^2 \Big]$$ 

where $$F_{\theta}$$ is a model for the failure CDF $$F$$. We focus on BS and discuss why we do not focus on concordance at the end of this post.
BS is usually motivated by the fact that it measures both discrimination (prediction) and calibration (that model probabilities match with empirical frequencies).

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
 
Crucially, while this expectation equals the previous definition of $$BS(t;\theta)$$, Monte-Carlo estimates of it only require samples from the *observed data* $$X,U,\Delta$$.

## So can we optimize the inverse-weighted BS? 
* * *

What stops us from using the re-weighted BS as an objective for our $$F_\theta$$ model? Unfortunately the censoring distribution $$G$$ required in the estimates
is the *true censoring distribution* rather than a model. But in practice, we do not know this distribution ahead of time. How can we get it? 

Modeling the censoring distribution is itself a *censored survival task*: observed failures censor the censoring times!  Moreover,
because $$G$$ shows up by computing probabilities, we care the the estimated model $$G_\theta$$ be calibrated.


**Quick recap**: we want to estimate $$F_\theta$$ with BS. The BS needs $$G$$ under censoring. We do not have $$G$$. So we can model
it with $$G_\theta$$ and then plug it into $$F_\theta$$'s objective. Which objective do we use for $$G_\theta$$?

Well, since we already know that BS optimizes calibration, can we get our $$G_\theta$$ estimate using BS as an objective? To do this we would need to use the re-weighted BS for $$G_\theta$$. This is like the above re-weighted BS
but with the roles of $$F$$ and $$G$$ swapped (and $$\Delta$$ flipped):


$$\begin{align*}
BS(t;\theta) &=  \mathbb{E} \Big[ 
									\frac{\overline{G_\theta}(t|X)^2 \overline{\Delta} 1[U \leq t]}{\overline{F}(U^{-}|X)}								
							+
									\frac{G_\theta(t|X)^2 1[U > t]}{\overline{F}(t|X)}
							\Big] 
\end{align*}
$$
 
Since we again do not know the true $$F$$, we replace it with $$F_\theta$$.

This leads to what we call the **inverse-weighting dilemma**: 
- $$F_\theta$$'s objective needs a $$G_\theta$$ model
- if the $$G_\theta$$ estimate is bad, the objective does not equal the BS
- to get a good $$G_\theta$$ we need to optimize $$G$$'s BS
- $$G$$'s BS needs $$F$$, but we only have an $$F_\theta$$ model of questionable quality

Is there anything we can do to ensure that the optimization leads to a good $$G_\theta$$, which in turn defines the BS for $$F_\theta$$,
which in turn we optimize to get a good estimate of $$F_\theta$$ that performs well under BS evaluation at test time?

## First try to optimizing inverse-weighted BS
* * *


A first try could be this: why not optimize the sum of $$F_\theta$$ and $$G_\theta$$'s Brier Scores, with respect to both models, where all inverse-weights
are replaced by models. That is, $$G_\theta$$ shows up in $$F_\theta$$'s BS and vice-versa. This is what we consider to be the "basic machine learning solution"
to this problem. Does it work?

We show the loss contours of this optimization for BS(t=1) for a problem over two timesteps (where the failure and censoring model each only have one parameter):

![min plot](assets/img/min_plot.png)
![legend](assets/img/key.png)

Unfortunately, the solution to this optimization is not at the true failure and censoring distribution. This occurs
because the models show up in denominators and can down-weight the contribution to the loss where the model is wrong.

**In the rest of this post**, we present our solution to this problem, Inverse-Weighted Survival Games, which define an optimization that follows gradients to the right solution,
which looks like this:

![grad plot](assets/img/grad_plot.png)


## Inverse-Weighted Survival Games
* * *

To keep things simple let us stick with the 2-timestep setting where each model
only has one parameter. 

Let's look again at the BS for $$F_\theta$$ and $$G_\theta$$:

$$\begin{align*}
BS(t;\theta) &=  \mathbb{E} \Big[ 
									\frac{\overline{F_\theta}(t|X)^2 \Delta 1[U \leq t]}{\overline{G}(U^{-}|X)}								
							+
									\frac{F_\theta(t|X)^2 1[U > t]}{\overline{G}(t|X)}
							\Big] \\
BS(t;\theta) &=  \mathbb{E} \Big[ 
									\frac{\overline{G_\theta}(t|X)^2 \overline{\Delta} 1[U \leq t]}{\overline{F}(U^{-}|X)}								
							+
									\frac{G_\theta(t|X)^2 1[U > t]}{\overline{F}(t|X)}
							\Big] 
\end{align*}
$$


## Experiments

* * * 


![conc vs nll](assets/img/conc_vs_nll.png)









