---
layout: post
title: Separating High-Dimensional Data with given Points for the Decision Boundary
modified:
categories: [ML]
description: Recently, someone gave me a strange problem, which seemed to be quite trivial on the first glance (and actually is. However, it took me a little time to put my rusty knowledge of vector and matrix algebra together) We have quite a few of n-dimensional data-points (exact knowledge of the actual data is not required here), which belong to one of two classes. For a few cases we know the class, but for many cases we do not. This might appear like a typical classification task, if it was not for one additional curiosity. we actually already know quite some data-points which lie on (or very close to) the decision boundary. This turns our original problem somehow into a regression problem, since we can estimate a hypersurface that fits the decision boundary. You also have to think about some things in this case as well, but in this blog post we want to look at another approach.
tags: [Classification, Multidimensionality]
image:
  feature: 2254s.jpg
  credit: Designed by jcomp / Freepik
  creditlink: http://www.freepik.com
comments:
share:
date: 2017-10-06T20:42:49+02:00
---

\\(
   \def\matr#1{\mathbf #1}
   \def\tp{\mathsf T}
\\)

Recently, someone gave me a strange problem, which seemed to be quite trivial on the first glance (and actually is. However, it took me a little time to put my rusty knowledge of vector and matrix algebra together):
We have quite a few of $$n$$-dimensional data-points $$\vec x_k \in \mathbb{R}^n$$ (exact knowledge of the actual data is not required here), which belong to one of two classes. For a few cases we know the class, but for many cases we do not. This might appear like a typical classification task, if it was not for one additional curiosity: we actually already know quite some data-points which lie on (or very close to) the decision boundary. This turns our original problem into some sort of regression problem, since we can estimate a hypersurface that fits the decision boundary. You also have to think about several problems in this case as well (e.g., how to incorporate those data-points into the model, which do not lie in the boundary region but for which a class label is known), but in this blog post we want to look at another approach, which in the end provides a linear decision boundary.
<!--more-->

Since we had more than $$n-1$$ of such points, my first idea was to simply find a hyperplane that fits through $$n-1$$ of the given points (assuming all are linearly independent) in order to get a simple decision boundary and to evaluate the performance of this simple model. There are several things to do for this:

1. We have to find the support vector and the direction vectors of the hyperplane (parametric form).
2. A relation is required, which can decide on which side of the plane a new (with an unknown class) point is located. The parametric form is not ideal for this purpose, so that an alternative has to be found.
3. Ideally, a distance (in this case a simple euclidean distance) to the plane should be computed, which can be an indication for the certainty of a classification decision.

So lets get started: In the simplest case one has to find a line which passes through two points $$P_1$$ and $$P_2$$ described by the vectors $$\vec p_1$$ and $$\vec p_2$$. This is illustrated in the following figure.

![Support Vector and Direction Vector for the Parametric equation of a plane]({{ site.url }}/images/2017-10-06-separating-high-dimensional-data-with-given-points-on-the-cutting-hyperplane/supportvec.png){: .image-center width="400px"}

A parametric description for this line can be easily found to be:

$$
\vec x = \vec p_1 + t (\vec p_1 - \vec p_2),
$$

where $$\vec x$$ is any vector which points to the given line.
Accordingly, a plane in three dimensions can be expressed with three points ($$\vec p_1, \vec p_2, \vec p_3$$):

$$
\vec x = \vec p_1 + t (\vec p_1 - \vec p_2) + s (\vec p_1 - \vec p_2)
$$

More generally, for the $$n$$-dimensional case one can write:

$$
\begin{equation}
\vec x = \vec p_1 + \sum_{i=2}^n t_i (\vec p_1 - \vec p_i)
\label{eq:parametricPlane}
\end{equation}
$$

The above parametric form is easy to compute, but using it as a decision boundary for classification does not seem to be straight-forward. We have to find a way to figure out, on which side of the hyperplane a point is located. Ideally we would want a decision boundary (which is for example used for logistic regression) in the form of:

$$
w_0 + \sum\limits_{i=1}^{n} w_i x_i = 0
$$

where $$\vec w \in \mathbb{R}^{n+1}$$ represents the weights of the classifier. Normally, these weights are learnt (e.g. with some gradient descent appproach), in this case these weights are already implicitly given by the points, for which we know that these are on the decision boundary. We just have to find a suitable transformation of equation \eqref{eq:parametricPlane}.

For this purpose, let us first find a normal vector $$\vec n_0$$, which stands perpendicular on the hyperplane. The reason for this step will become clearer later. Formally, $$\vec n_0$$ has to be orthogonal to all direction vectors $$(\vec p_1 - \vec p_i)$$ in equation \eqref{eq:parametricPlane}. One approach (although not always the best for numerical reasons) to find $$\vec n_0$$ is to compute the cross product of all direction vectors. This is described in another [blog post]({% post_url 2017-10-06-a-generalization-of-the-vector-cross-product %}).
Typically, the normal vector is normalized so that $$\lvert \vec n_0 \rvert = 1$$.

Now let us reformulate equation \eqref{eq:parametricPlane} a little (remember that the normal vector was orthogonal to all direction vectors $$(\vec p_1 - \vec p_i)$$ on the hyperplane):

$$
\begin{align}
\vec x &= \vec p_1 + \sum_{i=2}^n t_i (\vec p_1 - \vec p_i)  \\
\vec x - \vec p_1 &=  \sum_{i=2}^n t_i (\vec p_1 - \vec p_i) \\
\vec n_0 (\vec x - \vec p_1) &=  \vec n_0 \sum_{i=2}^n t_i (\vec p_1 - \vec p_i) \\
\vec n_0 (\vec x - \vec p_1) &=  \vec n_0 \sum_{i=2}^n t_i (\vec p_1 - \vec p_i) \\
\vec n_0 (\vec x - \vec p_1) &=   \sum_{i=2}^n t_i \underbrace{(\vec p_1 - \vec p_i) \vec n_0}_{=0} \\
\vec n_0 (\vec x - \vec p_1) &=   0 \\
\underbrace{\vec n_0 \vec x}_{\sum_{i=1}^n (n_0)_ i x_i } - \vec n_0 \vec p_1 &=   0 \\
\sum_{i=1}^n \underbrace{(n_0)_ i}_{w_i} x_i  \underbrace{- \vec n_0 \vec p_1}_{w_0} &=   0 \label{eq:parametricPlaneReform1} \\
{w_0} + \sum_{i=1}^n {w_i} x_i &=   0 \label{eq:parametricPlaneReform}
\end{align}
$$

This is exactly the equation that we attempted to receive.

For the sake of completeness, let us look at another way to find above equation:
Once the normal vector is known, another algebraic description of the hyperplane can be found in a slightly different way. Only a few simple vector algebra rules are required. The figure below shows a simple illustration for two dimensions.

![Normal Vector perpendicular to a Hyperplane]({{ site.url }}/images/2017-10-06-separating-high-dimensional-data-with-given-points-on-the-cutting-hyperplane/hessenormal.png){: .image-center width="400px"}

 The vector $$d \cdot \vec n_0$$ (where $$d$$ is a scalar value) is perpendicular to the given line (or hyperplane in higher dimensions), so that we can find the relation:

$$
\begin{equation}
\cos \alpha = \frac{\lvert d \cdot \vec n_0 \rvert}{\lvert \vec x \rvert} = \frac{d}{\lvert \vec x \rvert}
\label{eq:cosAlpha}
\end{equation}
$$

Furthermore, we know that

$$
\begin{equation}
\vec n_0^\tp \vec x = \cos \alpha \cdot \lvert \vec n_0 \rvert  \lvert \vec x \rvert = \cos \alpha \cdot \lvert \vec x \rvert
\label{eq:dotProduct}
\end{equation}.
$$

If we now insert \eqref{eq:cosAlpha} into \eqref{eq:dotProduct} we get the simple expression:

$$
\begin{align}
\vec n_0^\tp \vec x &= d \label{eq:hesse}\\
\sum_{i=1}^n (n_0)_i x_ i - d &= 0 \\
\sum_{i=1}^n w_i x_ i + w_0 &= 0 \\
 \vec w \begin{pmatrix} 1 \\ \vec x \end{pmatrix} &= 0
\end{align}
$$

You will notice, that equation \eqref{eq:hesse} is actually the same as \eqref{eq:parametricPlaneReform1} -- \eqref{eq:parametricPlaneReform}, where $$w_i = (n_0)_ i$$ and $$w_0 = -d$$.


Now not much left is to do. Since we have the decision boundary in a nicer form now, we can find a way to figure out on which side of decision boundary an arbitrary point is located, ideally connected with a distance from the boundary. Let us look at a vector (data-point) $$\vec r$$ which does not lie on the decision boundary, as shown in the figure below.

![Normal Vector perpendicular to a Hyperplane]({{ site.url }}/images/2017-10-06-separating-high-dimensional-data-with-given-points-on-the-cutting-hyperplane/hessedistance.png){: .image-center width="400px"}

In this example the vector $$\vec r$$ has a positive distance $$d_r$$ from the decision boundary. Hence, we can express $$\vec r$$ in terms of some vector $$\vec x$$ -- which points to the decision boundary -- and the normal vector $$\vec n_0$$ (assuming for now that we know the actual value of $$d_r$$):

$$
\begin{equation}
\vec x = \vec r - d_r \vec n_0
\label{eq:newRVec}
\end{equation}
$$

In this case $$d_r$$ is positive, which means that $$\vec r$$ is on the side of the decision boundary which is indicated by the direction of $$\vec n_0$$. If $$d_r$$ was negative, $$\vec r$$ would be located in the other side of the decision boundary. Since $$\vec x$$ is a vector on the line, we can insert \eqref{eq:newRVec} into \eqref{eq:hesse} and get the following:

$$
\begin{align}
\vec n_0 (\vec r - d_r \vec n_0) &= d \\
\vec n_0 \vec r - d_r \vec n_0 \vec n_0 &= d \\
\vec n_0 \vec r - d_r \lvert \vec n_0 \lvert^2 &= d \\
\vec n_0 \vec r - d_r  &= d \\
d_r  &= \vec n_0 \vec r - d  \\
d_r  &=  \sum_{i=1}^n {w_i} r_i + {w_0},
\label{eq:hesseDist}
\end{align}
$$
where a comparison to equations \eqref{eq:parametricPlaneReform1} -- \eqref{eq:parametricPlaneReform} and \eqref{eq:hesse} reveals that $$\vec n_0$$ can be expressed in terms of the weights $$w_1, \ldots, w_n$$ and $$w_0 = -d$$.

With this last equation \eqref{eq:hesseDist} we have found a way to compute the euclidean distance $$d_r$$ of any point $$\vec r$$ to the decision boundary. Furthermore, the sign of $$d_r$$ indicates on which side of the decision boundary a point is located. This is essential for classifying new data-points.

Back to our original problem: The approach described above provides a linear decision boundary when a sufficient number of points on the decision boundary are known ($$n-1$$ points when the input-space has $$n$$ dimensions).
In summary, the following steps have to be performed in order to make it work:
1. Select exactly $$n-1$$ linearly independent data points from the set of points on the decision boundary
2. For these points, compute the direction vectors on the hyperplane
3. With the direction vectors, compute a normal vector that is orthogonal to all direction vectors
4. With the normal vector, determine the weights $$\vec w \in \mathbb{R}^{n+1}$$ of a linear classifier based on Eq.\eqref{eq:parametricPlaneReform1}
5. Classify new points using the model determined in 4.  

 Obviously, this approach does not deliver optimal results in many cases but can be used as some kind of baseline for more advanced algorithms. For our particular problem, it actually worked quite well to describe the decision boundary by a hyperplane which passed through $$n-1$$ data-points in the boundary region.
