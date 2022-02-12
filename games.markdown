---
layout: post
title: Inverse-Weighted Survival Games
author: Mark Goldstein
permalink: games.html
---

* * * 

This post is about Survival Analysis (the modeling of time-to-event distributions), 
and about our recent NeurIPS 2021 work [Inverse-Weighted Survival Games](https://arxiv.org/abs/2111.08175)
with equal contribution from <a href="https://scholar.google.com/citations?user=WJnM24AAAAAJ&hl=en">Xintian Han</a> and 
<a href="https://marikgoldstein.github.io/">myself</a>.

This work presents a new training scheme for survival models that directly targets
evaluations the people use in practice. and as a theoretical aside, corresponds to a joint
estimator of the failure and censoring distribution that has not been previously studied.

$$\begin{align*}
	\texttt{failure player}: &\min_{F_{\theta}} \ell_F(F_\theta;G_\theta)\\
	\texttt{censor player}:	 &\min_{G_{\theta}} \ell_G(G_\theta;F_\theta)
\end{align*}
$$

(*For those not familiar with survival modeling, you will be in just a few moments by reading my* 
<a href="./survival.html">Survival Analysis Intro</a>!)

The work presents a new method, *IPCW (Inverse Probability of Censoring Weighted) Games*, for estimating survival
distributions, motivated by the following points:

- Survival models are evaluated on criteria like 
Brier Score (BS), Bernoulli Log Likelihood (BLL), and Calibration. These metrics
are more conducive to decision-making than likelihood is.

- In theory, Maximum Likelihood Estimation (MLE) for survival analysis is proper/consistent and asymptotically efficient

- Instead, optimizing directly for the above evaluations can
		improve performance on finite data in practice (while still mantaining some infinite-data properties of MLE)



The approach we present, unlike the usual MLE, makes use of a censoring model
(the distribution that causes missingness of times-to-event is usually not modeled, see <a href="./survival.html">survival intro</a>)
and plays a *game* between a *failure model player* and *censoring model player*.


## Outline
* * *

- Survival math setup 
- Brier Score (BS) under censoring
- BS hard to optimize directly
- Present Brier games as an example of IPCW Games
- Summarize some theoretical and empirical results comparing the games against MLE 


## Brief recap of survival setup
* * *

For each patient $$X$$, 
we are interesting in modeling the time-to-event (called the *failure time*) $$T|X$$ with CDF $$F$$.
There is also a censoring time $$C|X$$ with CDF $$G$$ for each patient.
Under right-censoring, we only observe $$(X,U,\Delta)$$ where $$U=\min(T,C)$$ and $$\Delta=1[T \leq C]$$ 
instead of $$(X,T,C)$$.
We assume i.i.d. data and random censoring $$T \perp C | X$$. Finally, let $$\overline{F}=1-F$$
and $$\overline{G}=1-G$$ and let $$\overline{\Delta}=1-\Delta$$. For an intro to the typical MLE objective
using this notation, <a href="./survival.html">see here</a>.





## Evaluating with Brier Score
* * *
Though trained with likelihood, survival models are evaluated under metrics like 
Concordance, Calibration, and Brier Score (BS). We focus on BS,
which measures both discrimination and calibration.
BS picks a particular time $$t$$ and considers the classification problem of whether or not a datapoint fails by $$t$$. It does so
by measuring the average squared error between the event status and modeled CDF $$F_{\theta}$$:

$$BS(t;F_\theta) = \mathbb{E} \Big[ \Big( F_{\theta}(t|X) - 1[T \leq t] \Big) ^2 \Big]$$ 

Training aside, censoring makes it challenging even *to estimate* $$BS(t;F_\theta)$$.
This is because,
for patients censored before time t, we do not observe $$1[T \leq t]$$. However, if the true censoring CDF $$G$$
were known, we could use re-weighting to estimate the BS (derivation in the paper):

$$\begin{align*}
BS_G(t;F_\theta) &=  \mathbb{E} \Big[ 
									\frac{\overline{F_\theta}(t|X)^2 \Delta 1[U \leq t]}{\overline{G}(U^{-}|X)}								
							+
									\frac{F_\theta(t|X)^2 1[U > t]}{\overline{G}(t|X)}
							\Big] 
\end{align*}
$$

This re-weighting scheme traces back to survey methodology from the 1950s and has the intuition that,
if an uncensored patient looks similar to patients who are likely to be censored, this uncensored point's
contribution should be up-weighted to account for the expected number of missing censored points similar to them.

Crucially, while this expectation equals the previous definition of $$BS(t;F_\theta)$$, Monte-Carlo estimates of it only require samples from the *observed data* $$X,U,\Delta$$ and not $$(T,C)$$.
## Re-weighted BS Needs a $$G$$ estimate
* * *

What stops us from using the re-weighted BS as an objective for our $$F_\theta$$ model? Unfortunately the 
$$G$$ required in the estimates
is the *true censoring distribution* rather than a model. But in practice, we do not know this distribution ahead of time 
and must use a model $$G_\theta$$, computing $$BS_{G_\theta}(t;F_\theta)$$ rather than $$BS_G(t;F_\theta)$$.
But, where does this model come from?


## Quick recap 
* * * 
We want to estimate survival model $$F_\theta$$ by minimizing BS since it measures 
both calibration and discrimination. Under censoring, the BS needs the true censoring distribution $$G$$.
We do not have $$G$$. So we can model
it with $$G_\theta$$ and then plug it into $$F_\theta$$'s objective. But, which objective do we use for $$G_\theta$$?

## Where does $$G_\theta$$ come from?
* * *

Modeling the censoring distribution is itself a *censored survival task* that mirrors the original problem: 
observed failures censor the censoring times because we do not observe $$C$$ when $$T \leq C$$. 
We also care that our censoring model $$G_\theta$$ be calibrated
because we need to use it to produce probabilities rather than just predictions.

Since we already know that BS optimizes calibration, can use BS to estimate $$G_\theta$$?
To do this we would need to use the re-weighted BS for $$G_\theta$$. This is like the above re-weighted BS
but with the roles of $$F$$ and $$G$$ swapped (and $$\Delta$$ flipped):

$$\begin{align*}
BS_F(t;G_\theta) &=  \mathbb{E} \Big[ 
									\frac{\overline{G_\theta}(t|X)^2 \overline{\Delta} 1[U \leq t]}{\overline{F}(U^{-}|X)}								
							+
									\frac{G_\theta(t|X)^2 1[U > t]}{\overline{F}(t|X)}
							\Big] 
\end{align*}
$$
 
Since we again do not know the true $$F$$, we replace it with $$F_\theta$$ and compute $$BS_{F_\theta}(t;G_\theta)$$.

This leads to what we call the **inverse-weighting dilemma**: 
- $$F_\theta$$'s objective needs a $$G_\theta$$ model
- if the $$G_\theta$$ estimate is bad, the objective does not equal the BS
- to get a good $$G_\theta$$ we need to optimize $$G_\theta$$'s BS
- $$G_\theta$$'s BS needs $$F$$, but we only have an $$F_\theta$$ model of questionable quality
- with access only to these models, what can we say about optimizing $$F_\theta$$'s BS?


## Simple Two-Timestep Model
* * *
For the remainder of the post, let us consider a two-timestep problem where $$T$$ and $$C$$ both only take values in $$\{1,2\}$$. In such discrete settings,
the $$BS$$ at the last time step is equal to $$0$$ for any model so the only thing to optimize is $$BS(t=1)$$. This corresponds to the fact
that there is only one parameter in each model.

## Resolving the Dilemma: First Attempt
* * *

A first try could be to optimize the sum of $$F_\theta$$ and $$G_\theta$$'s Brier Scores, with respect to both models, where all inverse-weights
are replaced by models. That is, $$G_\theta$$ shows up in $$F_\theta$$'s BS and vice-versa:

$$loss(t;F_\theta,G_\theta) = BS_{G_\theta}(t;F_\theta) + BS_{F_\theta}(t;G_\theta) $$


This is what we first considered to be a possible solution to this problem. We now show that this does not work.
We show the loss contours of this optimization for $$loss(t=1;F_\theta,G_\theta)$$ with respect to each model's single parameter $$P_\theta(T=1)$$ and $$P_\theta(C=1)$$:


<p style="text-align:center;">
<img src="assets/img/min_plot_legend.png" alt="min plot" width="350"/>
</p>

We see that the solution to this optimization is not at the true failure and censoring distribution.
This comes from the fact that $$BS_{G_\theta}(t;F_\theta)$$ is not a proper objective
for $$G_\theta$$ and vice versa. 

**In the rest of this post**, we present our solution to this problem, Inverse-Weighted Survival Games, 
which define an optimization that *follows gradients to the right solution (!)*. The gradient
field for the $$BS(t=1)$$ game, defined below, looks like:

<p style="text-align:center;">
<img src="assets/img/grad_plot.png" alt="grad plot" width="350" align="center"/>
</p>


## Inverse-Weighted Survival Games
* * *

To keep things simple let us stick with the 2-timestep setting where each model
only has one parameter. Let's look again at the BS for $$F_\theta$$ and $$G_\theta$$, and replace the weights with models

$$\begin{align*}
BS_{G_\theta}(t;F_\theta) &=  \mathbb{E} \Big[ 
									\frac{\overline{F_\theta}(t|X)^2 \Delta 1[U \leq t]}{\overline{G}_\theta(U^{-}|X)}								
							+
									\frac{F_\theta(t|X)^2 1[U > t]}{\overline{G}_\theta(t|X)}
							\Big] \\
BS_{F_\theta}(t;G_\theta) &=  \mathbb{E} \Big[ 
									\frac{\overline{G_\theta}(t|X)^2 \overline{\Delta} 1[U \leq t]}{\overline{F}_\theta(U^{-}|X)}								
							+
									\frac{G_\theta(t|X)^2 1[U > t]}{\overline{F}_\theta(t|X)}
							\Big] 
\end{align*}
$$

**The main and straightforward insight we observe is** that the models should not be optimized with respect to their role as inverse-weights,
since the $$F_\theta$$ BS does *not* provide a proper objective for $$G_\theta$$ and vice-versa.

To define the games in this example with BS, we define losses for each of the models

$$\begin{align*}
	\ell_F(t;F_\theta;G_\theta) &= BS_{G_\theta}(t;F_\theta) \quad \text{loss for F model}\\
	\ell_G(t;G_\theta;F_\theta) &= BS_{F_\theta}(t;G_\theta) \quad \text{loss for G model}
\end{align*}
$$

What makes this a *game* is that the loss function for each player depends on the parameters
of both players, but each player can only optimize their loss with respect to their own parameters:

$$\begin{align*}
	\texttt{failure player}: &\min_{F_{\theta}} \ell_F(t;F_\theta;G_\theta)\\
	\texttt{censor player}:	 &\min_{G_{\theta}} \ell_G(t;G_\theta;F_\theta)
\end{align*}
$$

To implement the games, each model must be treated as a constant when appearing in the weights for the other
model's loss during gradient computation.

For problems with more than two timesteps, one can define a game based on e.g. the summed Brier Score:

$$\begin{align*}
	\ell_F(F_\theta;G_\theta) &= \sum_t BS_{G_\theta}(t;F_\theta) \quad \text{loss for F model}\\
	\ell_G(G_\theta;F_\theta) &= \sum_t BS_{F_\theta}(t;G_\theta) \quad \text{loss for G model}
\end{align*}
$$

**More generally** these games can be constructed using two ingredients:  
- proper scoring rules: losses like Brier Score and Bernoulli Log Likelihood that have the true distribution as a minimizer
			when an expectation is taken over the data distribution  
- re-weighted estimators of these losses that allow computation under censoring


## Theoretical Results
* * * 

The paper has two theoretical results concerning *stationary points*. Stationary points of games
are where they stop: at such points, no player changes their parameters. They are the analog of minima
in optimization problems.

- **Theorem 1** When the loss used for each player's objective is proper, the games always have a stationary
	point at the true failure and censoring distributions. This means that the games will not move
		away from the true data distributions when they are reached.
- **Theorem 2** In the case of Summed Brier Score for discrete failure and censoring distributions,
	the stationary point at the true data distributions is unique.


For both theorems, when we refer to a loss we mean those defined with respect to expectations over the samples from the true
data distribution. 

The first theorem holds for any proper loss, for example Brier Score(t), 
Summed Brier Score,
Negative Bernoulli Log Likelihood(t) (similar to BS but with negated log
instead of square) and its summed variant, and some variants of AUC(t).
Interestingly, common variants of Concordance are not proper, so we do not study
games based on it.

Though the second theorem is specifically for discrete time data distributions with matching 
failure and censoring support, it can hold for continuous
distributions under more assumptions.

## Experiments

* * * 

To show that the theorems are useful in practice, we investigate IPCW games 
on simulations, a semi-simulation based on MNIST, and real data on cancer and critically-ill hospital patients.
We include the MNIST results and some cancer results here. 
The rest can be found [in the paper](https://arxiv.org/pdf/2111.08175.pdf).

### Survival MNIST

[Survival-MNIST](https://k-d-w.org/blog/2019/07/survival-analysis-for-deep-learning/)
draws times conditionally on MNIST label
$$Y$$. This means digits define risk groups (for event to happen sooner vs later) and $$T \perp X | Y$$. 
Times within a digit are i.i.d. The model only sees the image pixels $$X$$ as covariates so it must learn to 
classify digits (risk groups) to model times. We follow [X-CAL](https://arxiv.org/pdf/2101.05346.pdf) 
and use Gamma times: $$T ∼ Gamma(mean = 10(Y + 1))$$. We set the variance constant to $$0.05$$. 
Lower digit labels $$Y$$ yield earlier event times. $$C$$ is drawn similarly but with mean $$9.9(Y + 1)$$. 
Each random seed draws a new dataset. We use categorical models that model times as falling into
one of 50 ordinal bins.
<p style="text-align:center;">
<img src="assets/img/results_mnist.png" alt="grad plot" width="900" align="center"/>
</p>
Since this is a semi-simulation, we have the true failure times available and can measure 
uncensored BS and uncensored Bernoulli Log Likelihood (BLL) which is like BS but with log loss instead
of squared error. We also report concordance and the traditional NLL:
- The blue curve represents models trained with NLL
- The orange curve represents models trained with BS games
- The green curve represents models trained with BLL games
- The X axis shows the number of training points used to produce the models whose test evaluation is shown in the plot.

What we see in the plots is that the games bring the test set BS loss down using smaller amounts of training data
than is necessary for the traditional NLL-trained model. The games bring up concordance faster than the NLL model,
and we also see that better test NLL does not directly correspond to better test BS.

###  METABRIC

We also study survival times of patients in the 
Molecular Taxonomy of Breast Cancer International Consortium (METABRIC).
The plots are similar, except since it is real data, there are no ground truth times available for 
censored patients. This means that we cannot report uncensored BS or BLL. 
Following [Kvamme et al.](https://arxiv.org/pdf/1907.00825.pdf),
we use the Kaplan-Meier weighted estimates of these test metrics.
<p style="text-align:center;">
<img src="assets/img/results_metabric.png" alt="grad plot" width="900" align="center"/>
</p>
The results are similar, with the game methods needing less data to bring down the test set BS and BLL,
bring up the concordance, and in this case also bring down the NLL even though they do not optimize for it directly.

### Conclusion

As we have seen, when BS is valued as an evaluation of survival models, it should be used directly as 
a training objective. However, this does not come without challenges, as it is necessary to re-weight 
metrics like BS by a model of the censoring distribution. To tackle the problem of where this censoring model
comes from, we present a game method that uses the failure and censoring model in re-weighting estimates of
each other's training objectives. The result of these games are failure models that:
- like NLL, produce the true distributions with infinite data. 
- unlike NLL, based on the empirical evaluations explored in the work, produce
models with better BS, BLL, and Concordance in the finite-sample case.









<!--
![conc vs nll](assets/img/conc_vs_nll.png)
-->







