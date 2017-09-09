---
layout: post
title: Gradient of the Softmax Function with Cross-Entropy Loss
modified:
categories:
description:
tags: []
image:
  feature: abstract-1s.png
  credit: dargadgetz
  creditlink: https://www.dargadgetz.com/ios-8-abstract-wallpaper-pack-for-iphone-5s-5c-and-ipod-touch-retina/
comments:
share:
date: 2017-09-07T19:29:40+02:00
---

Some neat intro here.

The general Softmax function for a unit $$z_j$$ is defined as:
\begin{eqnarray}
o_j = \frac{\mbox{e}^{z_j}}{\sum_k \mbox{e}^{z_k}}
\end{eqnarray}
The cross-entropy loss for a softmax unit with $$p$$ output units $$o_j$$ and targets $$y^*_j$$ is defined as:

$$
\begin{eqnarray}
E=-\sum_j^{p} y^*_j\cdot \mbox{log}(o_j)
\end{eqnarray}
$$

In order to compute the gradient of $$E$$ with respect to $$z_j$$, we can start with:

$$
\begin{align}
\frac{\partial E}{\partial z_i}&=- \sum_j^{p} y^*_j\cdot  \frac{\partial}{\partial z_i}\mbox{log}(o_j)\\
&=- \sum_{j \neq i}^{p} y^*_j\cdot  \frac{\partial}{\partial z_i}\mbox{log}(o_j) - y^*_i\cdot  \frac{\partial}{\partial z_i}\mbox{log}(o_i)
\label{eq:partialSoftmax}
\end{align}
$$

Now we compute both partial derivatives of $$o_j$$ and $$o_i$$, which lead to different results:

$$
\begin{align}
  \frac{\partial}{\partial z_i} o_j &= \frac{\partial}{\partial z_i} \frac{\mbox{e}^{z_j}}{\sum_k \mbox{e}^{z_k}}\\
  &= \mbox{e}^{z_j} \frac{\partial}{\partial z_i} \Bigg(\sum_k \mbox{e}^{z_k} \Bigg)^{-1}\\
  &= -\mbox{e}^{z_j} \Bigg(\sum_k \mbox{e}^{z_k} \Bigg)^{-2} \mbox{e}^{z_i}\\
  &= -o_j \cdot o_i
  \label{eq:partialOj}
\end{align}
$$

and

$$
\begin{align}
  \frac{\partial}{\partial z_i} o_i &= \frac{\partial}{\partial z_i} \frac{\mbox{e}^{z_i}}{\sum_k \mbox{e}^{z_k}} \\
  &= \frac{\mbox{e}^{z_i}}{\sum_k \mbox{e}^{z_k}} + \mbox{e}^{z_i}\frac{\partial}{\partial z_i} \frac{1}{\sum_k \mbox{e}^{z_k}}\\
  &= \frac{\mbox{e}^{z_i}}{\sum_k \mbox{e}^{z_k}} - \mbox{e}^{z_i} \Big(\sum_k \mbox{e}^{z_k}\Big)^{-2} \mbox{e}^{z_i}\\
  &= o_i - o_i^2
  \label{eq:partialOi}
\end{align}
$$

With  these 2 partial derivatives we can now simplify Eq.\eqref{eq:partialSoftmax}:

$$
\begin{align}
\frac{\partial E}{\partial z_i}&=-\sum_{j \neq i}^{p} y^*_j\cdot  \frac{\partial}{\partial z_i}\mbox{log}(o_j) - y^*_i\cdot  \frac{\partial}{\partial z_i}\mbox{log}(o_i)\\
&= -\sum_{j \neq i}^{p} y^*_j \frac{1}{o_j} \frac{\partial}{\partial z_i}o_j - y^*_i \frac{1}{o_i} \frac{\partial}{\partial z_i}o_i
\label{eq:partialSoftmax2}
\end{align}
$$

We insert \eqref{eq:partialOj} and \eqref{eq:partialOi} into \eqref{eq:partialSoftmax2}:

$$
\begin{align}
\frac{\partial E}{\partial z_i}
&= \sum_{j \neq i}^{p} y^*_j \frac{1}{o_j} o_j \cdot o_i - y^*_i \frac{1}{o_i} (o_i - o_i^2)\\
&= \sum_{j \neq i}^{p} y^*_j   o_i - y^*_i  (1 - o_i)\\
&= o_i\sum_{j \neq i}^{p} y^*_j    - y^*_i   + y^*_i  o_i \\
&= o_i \underbrace{\sum_{j =1}^{p} y^*_j }_{=1}   - y^*_i  \\
&= o_i  - y^*_i
\end{align}
$$
