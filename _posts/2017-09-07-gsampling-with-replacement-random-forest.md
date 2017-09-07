---
layout: post
title: Sampling with Replacement and Random Forests
modified:
categories:
description:
tags: []
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2017-09-07T19:37:32+02:00
---

Often one has to consider the problem where a sample has to be drawn of a set containing $$n$$ elements, where the sample also has the size $$n$$. For instance Random Forests sample the training data $$n$$ times with replacement for each decision tree. One question arises in this case: How many unique training examples can we expect in the sample? Since we sample with replacement, several examples will be drawn twice most likely. To calculate the expected number of unique examples, one can proceed as follows:
First we define a random Variable $$X_i$$, $$i \in \{1,\ldots, n \}$$ which defines the events "Example $$i$$ was $$(X_i=1)$$ / was not $$(X_i=0)$$ selected by the sampling process". Based on the probability of these events, we can formulate an expected value for each individual which shows how many times an example is selected on average:

$$
\begin{align}
	E[X_i] 	&= 0\cdot P(X_i=0) + 1\cdot P(X_i=1) \\
			&= P(X_i=1) \\
			&= 1- P(X_i=0)
\end{align}
$$

The probability $$P(X_i=0)$$ can be computed easier than $$P(X_i=1)$$. Since we sample the set $$n$$ times, the probability of \emph{not} selecting element $$i$$ in any step is:

$$
\begin{align}
	P(X_i=0) &= \Big(\frac{n-1}{n}\Big)^n
\end{align}
$$

The complementary probability function $$P(X_i=1)$$ expresses how likely it is to draw an example at least once from the set. This is described with:

$$
\begin{align}
	E[X_i]	&= 1- P(X_i=0) \\
			&= 1- \Big(\frac{n-1}{n}\Big)^n
\end{align}
$$

If we have $$n$$ individuals in the set, we can expect to have the following number of unique examples in the sample. The random variable $$Y$$ describes the number of unique items in one sample of size $$n$$:

$$
\begin{align}
	E[Y] &= E \Bigg[ \sum_{i=1}^n X_i\Bigg] \\
		 &=  \sum_{i=1}^n E \big[X_i\big] \\ 	
		 &=  \sum_{i=1}^n  \Bigg(1- \Big(\frac{n-1}{n}\Big)^n\Bigg) \\
		 &=  n -  n\Big(\frac{n-1}{n}\Big)^n \\
		 &=  n -  \frac{(n-1)^n}{n^{n-1}}
\end{align}
$$

It is also possible to compute the fraction of the unique examples of the whole sample:

$$
\begin{align}
	E[Y_\%] &= \frac{E[Y]}{m}\\  
			&= \frac{n -  \frac{(n-1)^n}{n^{n-1}}}{n} \\
			&= 1- \frac{(n-1)^n}{n\cdot n^{n-1}} \\
			&= 1- \frac{(n-1)^n}{n^{n}} \\
			&= 1- \Big(\frac{n-1}{n}\Big)^n \\
			&= 1- \Big(1-\frac{1}{n}\Big)^n
\end{align}
$$

Interestingly, for large $$n$$, this series converges to:

$$
\begin{align}
	\lim_{n \to \infty} E[Y_\%] &= \lim_{n \to \infty} 1- \Big(1-\frac{1}{n}\Big)^n \\
								&= 1 - \exp (-1) \\
								&\approx 0.6321206
\end{align}
$$
