---
layout: post
title: Zalando's Fashion-MNIST Dataset
modified:
categories: [ML, Programming]
description:
tags: []
image:
  feature: OGF1PV0s.jpg
  credit: Designed by Benzoix / Freepik
  creditlink: http://www.freepik.com
comments:
share:
date: 2017-10-12T15:55:12+02:00
---
Recently, Zalando research published a new dataset, which is very similar to the well known [MNIST database of handwritten digits](http://yann.lecun.com/exdb/mnist/). The dataset is designed for machine learning classification tasks and contains in total 60 000 training and 10 000 test images (gray scale) with each 28x28 pixel. Each training and test case is associated with one of ten labels (0--9). Up till here Zalando's dataset is basically the same as the original handwritten digits data. However, instead of having images of the digits 0--9, Zalando's data contains (not unsurprisingly) images with 10 different fashion products. Consequently, the dataset is called [Fashion-MNIST dataset](https://github.com/zalandoresearch/fashion-mnist), which can be downloaded from [GitHub](https://github.com/zalandoresearch/fashion-mnist).
The data is also featured on [Kaggle](https://www.kaggle.com/zalando-research/fashionmnist). A few examples are shown in the following image, where each row contains one fashion item.

![A few examples of Zalandos Fashion-MNIST data are shown in this image, where each row contains one fashion item.]({{ site.url }}/images/2017-10-12-zalandos-fashion-mnist-dataset/zalando10x10.jpeg){: .image-center width="400px"}

<!--more-->

The 10 different class labels are:

- 0 T-shirt/top
- 1 Trouser
- 2 Pullover
- 3 Dress
- 4 Coat
- 5 Sandal
- 6 Shirt
- 7 Sneaker
- 8 Bag
- 9 Ankle boot

According to the authors, the Fashion-MNIST data is intended to be a direct drop-in replacement for the old MNIST handwritten digits data, since there were several issues with the handwritten digits. For example, it was possible to correctly distinguish between several digits, by simply looking at a few pixels. Even with linear classifiers it was possible to achieve high classification accuracy. The Fashion-MNIST data promises to be more diverse so that machine learning (ML) algorithms have to learn more advanced features in order to be able to separate the individual classes reliably.

Since the structure of the files in the Fashion-MNIST data is the same as for the old MNIST data, it is easy to load the new data with the older code. Using `R` the following functions should do the job and load the training and test cases including class labels into the global environment. The code was provided by [Brendan O'Connor](https://gist.github.com/brendano/39760):

{% highlight R %}
# Load the Zalando MNIST-Fashion (or recognition) dataset into R
# Fashion MNIST: https://github.com/zalandoresearch/fashion-mnist
# Handwritten digits: http://yann.lecun.com/exdb/mnist/
# assume you have all 4 files and gunzip'd them
# creates train$n, train$x, train$y  and test$n, test$x, test$y
# e.g. train$x is a 60000 x 784 matrix, each row is one digit (28x28)
# call:  show_digit(train$x[5,])   to see a digit.
# brendan o'connor - gist.github.com/39760 - anyall.org

load_mnist <- function() {
  load_image_file <- function(filename) {
    ret = list()
    f = file(filename,'rb')
    readBin(f,'integer',n=1,size=4,endian='big')
    ret$n = readBin(f,'integer',n=1,size=4,endian='big')
    nrow = readBin(f,'integer',n=1,size=4,endian='big')
    ncol = readBin(f,'integer',n=1,size=4,endian='big')
    x = readBin(f,'integer',n=ret$n*nrow*ncol,size=1,signed=F)
    ret$x = matrix(x, ncol=nrow*ncol, byrow=T)
    close(f)
    ret
  }
  load_label_file <- function(filename) {
    f = file(filename,'rb')
    readBin(f,'integer',n=1,size=4,endian='big')
    n = readBin(f,'integer',n=1,size=4,endian='big')
    y = readBin(f,'integer',n=n,size=1,signed=F)
    close(f)
    y
  }
  train <<- load_image_file('train-images-idx3-ubyte')
  test <<- load_image_file('t10k-images-idx3-ubyte')

  train$y <<- load_label_file('train-labels-idx1-ubyte')
  test$y <<- load_label_file('t10k-labels-idx1-ubyte')  
}


show_digit <- function(arr784, col=gray(12:1/12), ...) {
  image(matrix(arr784, nrow=28)[,28:1], col=col, ...)
}
{% endhighlight %}

Additionally, I added a function (quick & dirty code) to visualize a sample of training images. The function basically samples the training data and shows random images in a squared grid. If desired, the images are arranged in a way that each row contains only cases from one class:

{% highlight R %}
library(reshape2)
library(ggplot2)


showExamples <- function(nExamplesClass = 10, randomize = F) {
  classes <- sort(unique(train$y)) # a vector of all classes
  nClasses <- length(classes) # Number of classes
  iExamples <- as.matrix(sapply(classes,
                                function(i) {
                                  iLab <- which(train$y == i)
                                  sample(x = iLab, size = nExamplesClass, replace = F)
                                }))
  if(randomize) {
    iExamples <- matrix(sample(x = iExamples, size = length(iExamples), replace = F),
                  nrow = dim(iExamples)[1])
  }

  data <- array(train$x[iExamples, ], dim = c(nExamplesClass * nClasses, 28, 28))
  plotData <- melt(data, varnames = c("image", "x", "y"), value.name = "pixel")
  nrows <- if(randomize) round(sqrt(nExamplesClass * nClasses)) else 10
  p <- ggplot(plotData, aes(x = x, y = y, fill = pixel)) +
    geom_tile() +
    scale_y_reverse() +
    facet_wrap(~ image, nrow = nrows) +
    theme_bw()+
    theme(
      panel.spacing = unit(0, "lines"),
      axis.text = element_blank(),
      axis.ticks = element_blank(),
      strip.background = element_blank(),
      strip.text.x = element_blank(),
      panel.grid.major = element_blank(),
      panel.grid.minor = element_blank(),
      legend.position   = "none",
      axis.title.x = element_blank(),
      axis.text.x = element_blank(),
      axis.ticks.x = element_blank(),
      axis.title.y = element_blank(),
      axis.text.y = element_blank(),
      axis.ticks.y = element_blank()
    ) + scale_fill_gradient(low = "black", high = "white")

  plot(p)
}
{% endhighlight %}

For example, with `showExamples(20, T)` we can plot a 20x20 matrix with 400 random images:
![A few examples of Zalandos Fashion-MNIST data are shown in this image, randomly sampled from the training data.]({{ site.url }}/images/2017-10-12-zalandos-fashion-mnist-dataset/zalando20x20rand.jpeg)

For an upcoming post, I will apply some classical learning algorithms to this data and investigate how well they perform. The results are likely to be worse than for the original MNIST data.

Overall, the Fashion-MNIST dataset seems to be an interesting test-bed for beginners in machine learning to apply different already existing algorithms. And also more advanced data scientists could use this data for a fast first evaluation of new approaches. However, also this dataset is too simple to deliver reliable/comparable performance measures for new machine learning algorithms. Although I did not test it yet myself, I expect that simple convolutional neural networks would achieve an accuracy of around 90% on the Fashion-MNIST test data. More advanced neural networks can probably achieve an accuracy higher than 95%. For comparison, according to Zalando research, the human performance is about 84%.

Can you recognize some fashion items in the following matrix (although they are that tiny)? Our poor algorithms sometimes have to do this millions of times per day &#9786;.


![A few examples of Zalandos Fashion-MNIST data are shown in this image, randomly sampled from the training data.]({{ site.url }}/images/2017-10-12-zalandos-fashion-mnist-dataset/zalando30x30rand.jpeg)
