---
layout: post
title:  "Quantifying uncertainty in probability predictions"
date:   2019-02-24 13:00:00
type: post
---

Suppose you're interested in knowing the chances of an event $$X$$ occuring (e.g., $$X =$$ "*a nuclear strike over any populated area in the year 2019*").
When making predictions about events with binary outcomes (either the event happens or it doesn't), people generally report a single probability (e.g., a 2% chance of $$X$$ occurring).
But, you may wonder, why not report a confidence interval (e.g., 2% $$\pm$$ 0.5%), or a distribution of probabilities (e.g., a Beta distribution) to reflect uncertainty?

For example, this question comes up with prediction markets, where the market price can be interpreted as the best estimate of the probability of the event $$X$$ occuring.
But there are no confidence intervals on this market price.
Or consider models for classification, such as logistic regression or other machine learning algorithms, which produce predicted probabilities for each possible class (e.g., $$X_i =$$ "*transaction $$i$$ is fraudulent*").
In both of these cases, we face the same issue with representing uncertainty --- how does the market/model express confidence in its predicted probabilities?

In this post, I'll explain why this question stems from a fundamental confusion:
**it's a misconception to think that a predicted probability is a point estimate that doesn't convey any uncertainty.**
Below, I'll show that there are two distinct sources of uncertainty that are being conflated here, and that one or both can be used to express uncertainty.

## Two types of uncertainty

The key distinction here is between:

1. Uncertainty over the outcome, $$X$$ vs $$\neg X$$
	* Also known as [aleatoric uncertainty](https://en.wikipedia.org/wiki/Uncertainty_quantification#Aleatoric_and_epistemic_uncertainty)
	* (FYI: the symbol "$$\neg$$" is the negation operator and can be read as "not")
2. Uncertainty over model parameters which are used to generate a prediction for the outcome
	* Also known as [epistemic uncertainty](https://en.wikipedia.org/wiki/Uncertainty_quantification#Aleatoric_and_epistemic_uncertainty)

Let's unpack each case in depth.

### Uncertainty over outcomes

When we aren't working with a model, we only have the first source of uncertainty to deal with.
But it isn't obvious where the uncertainty lies: if we say $$Pr[X] = 0.02$$, it may appear that we've just given a point estimate.
But recall that this is a binary outcome space, i.e., the only possible outcomes are $$X$$ or $$\neg X$$.
So the full probability distribution (over the two possible outcomes) can be summarized by one probability, $$p := Pr[X]$$ (which implies $$1-p = Pr[\neg X]$$).
As we've provided a full probability distribution over the outcome space, it's not possible to say anything more --- any uncertainty must be embedded in this distribution.

Intuitively, probabilities near 0 or 1 reflect a high degree of certainty.
A prediction without any uncertainty at all would just be a *yes or no* answer, i.e., a predicted probability of 0 or 1.
It would just state which outcome will occur, with no notion of uncertainty or hedging.

More precisely, confidence in a probability prediction is reflected by how extreme it is *relative to a baseline or prior belief*.
To see this, suppose that there is an event $$X$$ that is very likely to occur, and that a prediction market has given $$X$$ a predicted probability of 0.97.
If you are maximally uncertain/ignorant about $$X$$, what probability do you assign?
Intuitively, you *hedge your bets* and stick to 0.97.
Here, 0.97 is the baseline, which you can treat as your prior probability.
Given this prior information, a prediction of 0.97 reflects maximal uncertainty.
(If you didn't have any prior information whatsoever, you would go with 0.5.)

Then, if you have some new information about $$X$$, you can update your prior to get a posterior.
If your information provides strong evidence in favor of $$X$$, then your posterior probability might jump up to, say, 0.997.
On the other hand, if your information strongly supports $$\neg X$$, then your posterior might drop to, say, 0.78.
Thus, your degree of confidence is revealed by the degree to which your probability moves away from the baseline and toward 0 or 1.

You can use Bayes' Theorem to play with some numbers yourself.
Denote your prior by $$p := Pr[X]$$, and assume you've used your information $$D$$ to compute the likelihood $$q := Pr[D \mid X]$$.
Then compute the posterior:

$$
Pr[X \mid D] = \frac{pq}{pq + (1-p)(1-q)}
$$

If you play with this formula for different values of $$p$$ and $$q$$, you'll get a sense of how the information in the likelihood $$q$$ updates the posterior probability.
Notice that when $$q=1/2$$, it cancels itself out and the posterior reduces to $$p$$, the prior.
Thus, when your information is uninformative, it leaves your prior belief unchanged.
Furthermore, you can show that for any prior belief $$p$$, if $$q > 0.5$$, then your posterior will be pushed upward from your prior (and vice versa).
That is, any information in favor of $$X$$ will increase your confidence in $$X$$; even if $$p=0.999$$, a value of $$q=0.501$$ will still increase your certainty in the outcome.

Judging the quality of probability predictions is simple: just check that they're [calibrated](https://scikit-learn.org/stable/modules/calibration.html).
For example, predictions made with, 80% confidence should be correct 80% of the time.
With enough completed predictions, you can plot a [reliability diagram](https://www.metoffice.gov.uk/research/climate/seasonal-to-decadal/gpc-outlooks/user-guide/interpret-reliability) to assess the calibration of the predictions.

### Uncertainty over models

When we *are* working with a model, we also have a second source of uncertainty --- that of the model.
This uncertainty is reflected in the posterior distribution over the parameters (at least for Bayesians --- frequentists would use the sampling distribution of the parameter estimator to derive confidence intervals / standard errors).
Because these parameters are used to produce predictions, their uncertainty propagates through to produce additional uncertainty over the outcome.

This is *meta*-uncertainty: uncertainty over the model which produces the uncertain prediction of the outcome.
(In fact, you can have higher levels of meta-uncertainty by including uncertainty over any hyperparameters of the model.)

For example, here's a specification for a Bayesian logistic regression model, where I've put an informative prior on the model coefficients:

$$\begin{aligned}
y_i &\sim Bernoulli(p_i) \\
\log\left(\frac{p_i}{1 - p_i}\right) &= \beta_0 + x_{i1} \beta_1 + x_{i2} \beta_2 + \ldots + x_{iK} \beta_K \\
\beta_k &\sim \mathcal{N}(0,1.5), \; k = 1,2,\ldots,K
\end{aligned}$$

You can see how the uncertainty from the model's prior (Normal distribution) propagates through, adding to the uncertainty in the likelihood (Bernoulli distribution).
As a result, we get a full density for $$p_i$$ over the interval $$(0,1)$$.
The spread of this density reflects model uncertainty, whereas the distance from the baseline probability reflects the degree of confidence in the prediction of the outcome.
The key thing to realize is that these two sources of uncertainty are orthogonal.

For example, you could simultaneously have a very confident prediction of the outcome but with a lot of model uncertainty --- a widely spread posterior distribution that is far away from the prior probability (where the prior is marked by the small X):

	  Confident prediction, uncertain model
	⠀⢸⠀⠀⠀⠀⠀⠀⠀⢀⠤⠤⣄⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⠀⠀⠀⠀⠀⡔⠁⠀⠀⠀⠘⢄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⠀⠀⠀⠀⡸⠀⠀⠀⠀⠀⠀⠀⠱⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⠀⠀⠀⢰⠁⠀⠀⠀⠀⠀⠀⠀⠀⠘⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⠀⠀⢀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⠀⠀⡸⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⠀⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⠀⢰⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⢄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⠀⡎⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⢀⠇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢣⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⢸⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢣⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⡎⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢢⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠣⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⢰⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⢄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⢸⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠢⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⡎⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠑⢄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠑⠢⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠀⢸⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠒⠤⢄⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸
	⠒⢺⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠛⠛⠛⠒⠒⠒⠒⠒⠒⠒X⠒⠒⠒⠒⠒⠒⠒⢺

Or you could have a very uncertain prediction of the outcome with very little model uncertainty --- a tightly distributed posterior distribution centered on the prior probability:

	  Uncertain prediction, confident model
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⡀⠀⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⡇⠀⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⡇⠀⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⡇⠀⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⡇⠀⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⢱⠀⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⢸⠀⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⠃⢸⠀⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⠀⢸⠀⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⠀⢸⠀⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⠀⢸⠀⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⠀⢸⠀⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⠀⠘⡄⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡎⠀⠀⡇⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⠀⠀⡇⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⠀⠀⡇⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⠇⠀⠀⡇⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⠀⠀⠀⢣⠀⠀⠀⠀⠀⡇
	⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡜⠀⠀⠀⢸⠀⠀⠀⠀⠀⡇
	⠒⡗⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠓⠒X⠒⠚⠓⠒⠒⠒⠒⡗

## Summing up

The argument I've made here can be summed up as:

1. A probability prediction is a full probability distribution and so it inherently quantifies uncertainty --- it's a misconception to think that you need a confidence interval to express uncertainty
2. Model uncertainty propagates through to produce an additional (but orthogonal) layer of uncertainty over the outcome

This may seem obvious in retrospect, but it's always good to gain clarity on the fundamentals, where intuition can overpower reason.
Here, intuitions such as "a scalar prediction must be a point estimate" and "you need a confidence interval to express uncertainty" are highly misleading.

In this case, understanding the distinct sources of uncertainty has resolved some confusion I had about prediction markets and machine learning model predictions.
The upshot is that (if I'm comfortable ignoring model uncertainty), I need only be concerned that the probability predictions are [calibrated](https://scikit-learn.org/stable/modules/calibration.html).
This is straightforward to check: for prediction markets, you just need some historical data on outcomes; for classifier models, you can check on a holdout dataset.
Then you can be comfortable that the predictions are accurately quantifying uncertainty.
