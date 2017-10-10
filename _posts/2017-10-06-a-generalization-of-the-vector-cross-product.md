---
layout: post
title: A Generalization of the Vector Cross Product
modified:
categories: [Vector Algebra]
description: Imagine, you have two 3-dimensional vectors $$\vec a$$ and $$ \vec b$$ and you require a third vector, which is orthogonal to the first two vectors. How would you compute this vector? In a simple solution you could compute the cross product of $$\vec a$$ and $$ \vec b$$ or vice versa.
tags: [Vectors, Cross Product, High Dimensions, Generalization, Multi Dimensionality]
image:
  feature: odrat188s.jpg
  credit: Designed by Bedneyimages / Freepik
  creditlink: http://www.freepik.com
comments:
share:
date: 2017-10-06T20:33:46+02:00
---

\\(
   \def\matr#1{\mathbf #1}
   \def\tp{\mathsf T}
\\)

Imagine you have two 3-dimensional vectors $$\vec a$$ and $$ \vec b$$ and you require a third vector, which is orthogonal to the first two vectors. How would you compute this vector? Typically, one would compute the cross product of $$\vec a$$ and $$ \vec b$$ or vice versa. This is easy in three dimensions since we can use the well known relation

$$
\vec a \times \vec b =
\begin{pmatrix}
a_2b_3 - a_3b_2 \\
a_3b_1 - a_1b_3 \\
a_1b_2 - a_2b_1 \\
\end{pmatrix}.
$$

Also for two dimensions the case is quite simple. However, instead of two vectors $$\vec a$$ and $$ \vec b$$ you only have one vector $$\vec a$$ and the orthogonal vector is formally just a linear mapping:

$$
\begin{pmatrix}
a_1 \\
a_2 \\
\end{pmatrix} \mapsto
\begin{pmatrix}
a_2 \\
-a_1 \\
\end{pmatrix}
$$

You can easily check that the inner product of both vectors is zero.
But what about higher dimensions? For example, how would you define the cross product in 5 dimensions or maybe even in 100 dimensions?

<!--more-->

As it turns out, it is not that difficult to compute the cross product in any dimension $$n \leq 2$$.
If we are in a space $$\mathbb{R}^n$$, we need $$n-1$$ vectors, each with $$n$$ elements, that are pairwise linearly independent (note that if two or more vectors are colinear then the cross product will be zero). Hence, we have a set of vectors

$$
\vec u_1 =\begin{pmatrix}
u_{11} \\
u_{21} \\
\vdots \\
u_{n1}
\end{pmatrix},
\vec u_2= \begin{pmatrix}
u_{12} \\
u_{22} \\
\vdots \\
u_{n2}
\end{pmatrix},
\cdots,
\vec u_{n-1} = \begin{pmatrix}
u_{1(n-1)} \\
u_{2(n-1)} \\
\vdots \\
u_{n(n-1)}
\end{pmatrix}.
$$

Let us write the cross product as:

$$
{⨉}_{j=1}^{n-1} \vec u_j = \vec u_1 \times \vec u_2 \times \cdots \times \vec u_{n-1}
$$

Now let us arrange all vectors $$\vec u_1 \cdots \vec u_{n-1}$$ in a matrix. Additionally, we add a standard basis vector $$\vec e_i$$ for dimension $$i$$ as a first column in the matrix:

$$
\matr U^i =  \begin{bmatrix}
 \vec e_i & \vec u_1 & \cdots & \vec u_{n-1}
\end{bmatrix} =

\begin{bmatrix}
\mathbf e_1 & u_{11} & \cdots & u_{1(n-1)}\\
\mathbf e_2 & u_{21} & \cdots & u_{2(n-1)}\\
\vdots & \vdots & \ddots & \vdots\\
\mathbf e_n & u_{n1} & \cdots & u_{n(n-1)}
\end{bmatrix}
$$

Note that the standard unit vector $$\vec e_i$$ is of magnitude 1 and is parallel to axis $$i$$ of the coordinate system. Hence, for every dimension we have a slightly different matrix $$\matr U^i$$.

Finally, we can express the cross product as a determinant for each dimension. We skip the math behind the derivation for this formula, since its understanding is not really required for applying the formula. Remember that the cross product is a vector with $$n$$ elements. Each element along axis $$i$$ is then separately computed with

$$
\Big({⨉}_{j=1}^{n-1} \vec u_j \Big ) _ {i} = \mbox{det}(\matr U_i) = |\matr U_i|
$$

Overall, we receive a vector in the following form:

$$
{⨉}_{j=1}^{n-1} \vec u_j  =
\begin{pmatrix}
  \mbox{det}(\matr U_1) \\
  \mbox{det}(\matr U_2) \\
  \vdots \\
  \mbox{det}(\matr U_n) \\
\end{pmatrix}
$$

Since the first column in $$\matr U_i$$ only contains one element different from zero, the determinant can be conveniently computed using the minors of matrix $$\matr U_i$$ and Laplace's formula.


Let us do this exemplarily for our initial example with the vectors

$$
\vec a =\begin{pmatrix}
a_{1} \\
a_{2} \\
a_3
\end{pmatrix},
\vec b =\begin{pmatrix}
b_{1} \\
b_{2} \\
b_3
\end{pmatrix}.
$$

We first create the matrix $$\matr U_i$$, which is:

$$
\matr U_i  =\begin{bmatrix}
\mathbf e_1 & a_{1} & b_{1}\\
\mathbf e_2 & a_{2} & b_{2}\\
\mathbf e_3 & a_3 & b_3
\end{bmatrix}.
$$

Then, we can compute the first element of the cross product vector for axis $$i=1$$:

$$
\begin{align*}
\mbox{det}(\matr U_1) &=
\begin{vmatrix}
\mathbf 1 & a_{1} & b_{1}\\
\mathbf 0 & a_{2} & b_{2}\\
\mathbf 0 & a_3 & b_3
\end{vmatrix} \\
& =
1 \cdot \begin{vmatrix}
 a_{2} & b_{2}\\
 a_3 & b_3
\end{vmatrix} -
0 \cdot \begin{vmatrix}
 a_{1} & b_{1}\\
 a_3 & b_3
\end{vmatrix} +
0 \cdot  \begin{vmatrix}
 a_{1} & b_{1}\\
 a_{2} & b_{2}
\end{vmatrix} \\
& =
a_2 b_3 - a_3 b_2
.
\end{align*}
$$

For the other two axis $$i=2$$ and $$i=3$$ we follow the same procedure and get:

$$
\begin{align*}
\mbox{det}(\matr U_2) &=
\begin{vmatrix}
\mathbf 0 & a_{1} & b_{1}\\
\mathbf 1 & a_{2} & b_{2}\\
\mathbf 0 & a_3 & b_3
\end{vmatrix} \\
& =
0 \cdot \begin{vmatrix}
 a_{2} & b_{2}\\
 a_3 & b_3
\end{vmatrix} -
1 \cdot \begin{vmatrix}
 a_{1} & b_{1}\\
 a_3 & b_3
\end{vmatrix} +
0 \cdot  \begin{vmatrix}
 a_{1} & b_{1}\\
 a_{2} & b_{2}
\end{vmatrix} \\
& =
a_3 b_1 - a_1 b_3
\end{align*}
$$

$$
\begin{align*}
\mbox{det}(\matr U_3) &=
\begin{vmatrix}
\mathbf 0 & a_{1} & b_{1}\\
\mathbf 0 & a_{2} & b_{2}\\
\mathbf 1 & a_3 & b_3
\end{vmatrix} \\
& =
0 \cdot \begin{vmatrix}
 a_{2} & b_{2}\\
 a_3 & b_3
\end{vmatrix} -
0 \cdot \begin{vmatrix}
 a_{1} & b_{1}\\
 a_3 & b_3
\end{vmatrix} +
1 \cdot  \begin{vmatrix}
 a_{1} & b_{1}\\
 a_{2} & b_{2}
\end{vmatrix} \\
& =
a_1 b_2 - a_2 b_1
.
\end{align*}
$$

Summarizing the previous results, we receive for the cross product $$\vec a \times \vec b $$:

$$
\vec a \times \vec b =
\begin{pmatrix}
  \mbox{det}(\matr U_1) \\
  \mbox{det}(\matr U_2) \\
  \mbox{det}(\matr U_3) \\
\end{pmatrix}
=
\begin{pmatrix}
  a_2 b_3 - a_3 b_2 \\
  a_3 b_1 - a_1 b_3 \\
  a_1 b_2 - a_2 b_1 \\
\end{pmatrix}
$$

which is exactly the same as we have already noted in the beginning.


Note that the base package in $$R$$ provides a function $$crossprod()$$ which, however, does not compute the cross product as described in this blog post. For this reason I provide a $$R$$-function here, which efficiently computes the cross product for arbitrary dimensions.

{% highlight R %}
opX <- function(...) {
  u <- list(...)
  #
  # check length of all vectors
  #
  len <- unlist(lapply(u, length))
  if(!all(len == len[1])) stop("All vectors must be of same length!")
  len <- len[1]
  #
  # Check, if correct number of vectors is provided
  #
  if(length(u) != (len - 1) ) stop("For vector length ",len," you have to provide ",len-1, " vectors!")
  U <- do.call(cbind, u)

  #
  # Compute the determinants based on the minors of U and Laplace's formula
  # The determinants form the new vector which is the result of the cross-product
  #
  sapply(1:len, function(i) (-1)^(i + 1) * det(as.matrix(U[-i,])))
}

#
# Run an example
#
a <- c(1,2,3,4)
b <- c(5,6,7,8)
c <- c(9,10,-11,12)

print(opX(a,b,c))

#
# Check the orthogonality property
#
print(t(a) %*% opX(a,b,c))
print(t(b) %*% opX(a,b,c))
print(t(c) %*% opX(a,b,c))
{% endhighlight %}
<!--- %* -->
