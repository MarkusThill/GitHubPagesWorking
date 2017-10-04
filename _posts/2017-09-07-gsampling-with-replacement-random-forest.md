---
layout: post
title: Sampling with Replacement and its Relationship to Random Forests
modified:
categories: [Stats, ML]
description: Often one has to consider the problem where a sample has to be drawn of a set containing n elements, where the sample also has the size. For instance Random Forests sample the training data n times with replacement for each decision tree. One question arises in this case How many unique training examples can we expect in the sample? Since we sample with replacement, several examples will be drawn twice most likely. To calculate the expected number of unique examples, one can proceed as follows. First we define a random Variable Xi which defines the events Example i was (Xi 1) or  was not (Xi 0) selected by the sampling process. Based on the probability of these events, we can formulate an expected value for each individual which shows how many times an example is selected on average
tags: []
image:
  feature: 177s.jpg
  credit: Designed by osaba / Freepik
  creditlink: http://www.freepik.com
comments:
share:
date: 2017-09-07T19:37:32+02:00
---
Consider the following problem: We have a bowl containing $$n$$ different balls, all labeled with a unique number. We randomly pick a ball from the bowl, note its number and then put the ball back into the bowl. The order in which the balls are picked is not relevant. In total, $$k$$ balls are picked in the same way. In combinatorics this problem is typically referred to as combinations with repetition (or sampling with replacement).
The number of possible samples which can be drawn from the bowl is:

$$
\binom{n+k-1}{k}
$$

One particular problem arises when we choose $$n=k$$, hence, from a set of $$n$$ unique elements we randomly draw a sample of size $$n$$ (always putting the elements back into the set again).
The image below illustrates the problem. On the left side of the figure we have a set of four unique elements. Several possible samples with $$k=n$$ drawn from this set are shown on the right side of the arrow. In total, actually 35 possible combinations exist, using the above formula.  
![Picture description]({{ site.url }}/images/samplereplace.jpg){: .image-center }


But what does this have to do with random forests, as suggested by the title of this blog post?

<!--more-->

I stumbled across this problem some time ago, when I was looking at the training process of random forests (RF). Most random forest implementations sample the training data (with $$n$$ training examples) $$n$$ times with replacement for each decision tree. I was thinking about one (actually simple) question which arises in this case: How many unique training examples can we expect in each sample? Since we sample with replacement, most likely several examples will be drawn twice. So the probability to receive a sample with all $$n$$ unique elements is comparably small (assuming a sufficiently large training set). To be precise, the probability is actually $$\frac{n!}{n^n}$$. At the same time, a sample containing only one unique element has an even smaller probability of $$n\Big(\frac{1}{n}\Big)^n = n^{-(n-1)}$$. Intuitively, one could expect that around 50% of all training examples might appear in the sample. As we will see in the following, this number is actually slightly higher.
To get a first impression, we can run a simple simulation in R.

{% highlight R %}
# How often do we want to repeat the simulation?
nSimulations <- 10000

runSimulation <- function(n) {
  # Function that draws a sample of size n from a set of n values
  # and returns the fraction of unique values within the sample
  sampleWithRep <- function(n) {
    # sample the set n times with replacement
    p <- sample(1:n, n, replace = T)

    # compute the fraction of unique values in this sample
    length(unique(p)) / n
  }

  # Run simulation many times and print the mean value for the
  # fraction of unique values within all simulated samples
  mean(sapply(rep(n,nSimulations), sampleWithRep))
}

# Sufficiently large set
n <- 10000

cat("On average, each sample contained", (runSimulation(n) * 100.0),
    "% of the values of the original set.")
{% endhighlight %}
<!--- %* -->
The above R-script should give you a value of approx. 63%, which indicates, that (for a sufficiently large training set, e.g. $$n=10\,000$$) roughly around 2/3 of the original training examples will appear in the sample. To show that this value of around 63% actually converges for increasing $$n$$, we can run the following code (using the function defined above):

{% highlight R %}
require(ggplot2)
require(scales) # For nicer axis labels

# Number of simulations for each value of n
nSimulations <- 10000

# generate sequence 1, 2, ..., 9, 10, 20, ..., 90, 100, 200, ..., 900, 1000, ..., 90 000
Ns <- c(t(1:9) %o% 10^(0:4))

# Run simulation for different n's
frac <- unlist(mclapply(Ns, runSimulation, mc.cores = 23))

# Zip
result <- data.frame(n = Ns, frac = frac)

# Average of the last 10 values, for convergence line
convergeVal <- round(mean(frac[(length(frac) - 10):length(frac)]), 4)

p <- ggplot(data = result, aes(x = n, y = frac)) +
  geom_line() +
  geom_point(size=2.5) +
  scale_x_log10(breaks = 10^(1:5), labels = comma, limits=c(1,1.2e5)) +
  scale_y_continuous(breaks=c(1, 0.9, 0.8, 0.7, convergeVal), labels = comma) +
  ylab("fraction of unique values") +
  theme_bw() +
  theme(axis.text=element_text(size=20),
        axis.title=element_text(size=20,face="bold"))
plot(p)
{% endhighlight %}
<!--- %* -->
Note that I run this code on a multicore system, since the computation on a single core would require a lot of time. The result of the above code is shown in the following graph:

![Picture description]({{ site.url }}/images/convergenceSampling.png){: .image-center }

Obviously, for a training set and sample size of $$n=1$$, for 100% of the cases we have one unique value. As we can see, for larger $$n$$ the curve converges to the value 0.6321, which might already appear familiar to you (e.g., from some problems in physics). We will see later, where this value comes from.

After performing several simulations we can now calculate the expected number of unique examples, for which one can proceed as follows:
First we define a random variable $$X_i$$, $$i \in \{1,\ldots, n \}$$ which defines the events "example $$i$$ was $$(X_i=1)$$ / was not $$(X_i=0)$$ selected by the sampling process". Based on the probability of these events, we can formulate an expected value for each individual which shows how many times an example is selected on average:

$$
\begin{align}
	E[X_i] 	&= 0\cdot P(X_i=0) + 1\cdot P(X_i=1) \\
			&= P(X_i=1) \\
			&= 1- P(X_i=0)
\end{align}
$$

The probability $$P(X_i=0)$$ can be computed easier than $$P(X_i=1)$$. Since we sample the set $$n$$ times, the probability of not selecting element $$i$$ in any step is:

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

Hence, random forests leave out roughly 1/3 of the cases in the training sample, which is used later for a so called out-of-bag (OOB) error estimate. 
