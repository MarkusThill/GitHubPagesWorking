---
layout: post
title: Obfuscating a Function &#8211; How not to write Code
modified:
categories:
description:
tags: []
image:
  feature: 530s.jpg
  credit: Designed by nikitabuida / Freepik
  creditlink: http://www.freepik.com
comments:
share:
date: 2017-10-11T15:01:50+02:00
---

Some time ago I had to write a pretty simple function, which converts an integer value into some new "format". The function ended up to be some easy-to-read code. So -- for some reasons which I do not remember -- I decided to obfuscate the purpose of the function slightly and I ended up with the following:

{% highlight C %}

#include <stdio.h>

char* whatDoIdo(unsigned int x, char *t) {
    int i=7,j=0,v=5000,p,z[7]={0x49,0x56,0x58,0x4c,0x43,0x44,0x4d};
    while(!(t[j]='\0')&&x&&(v/=5/((i--+1)%2+1))+1) {
        while(x>=v&&(x-=v)+1&&(t[j++]=z[i])+1);
        while(i&&x>=(p=v-v*(i%2+1)/10)&&(x-=p)+1&&(t[j++]=z[i-2+(i%2)])+1
          &&(t[j++]=z[i])+1);
    }
    return t;
}

int main() {
    unsigned int x = 123;
    char t[200] = {0};
    printf("%d : %s", x, whatDoIdo(x,t));
    return 0;
}
{% endhighlight %}
<!--- %* -->

If you run the code for different values of `x` in the function `main()` you will discover the purpose of function `whatDoIdo()` pretty fast. If you are more dedicated, then you can try to decrypt the code and guess it's functionality without even running it once. Certainly, you should prevent writing code like this at work or university, since your colleagues/classmates or others who have to read your code most likely won't appreciate this kind of programming style.

<!--more-->

In case that you do not have a compiler at hand, you can take a look at some sample outputs that should appear familiar:

{% highlight plaintext %}
1  : I
5  : V
50 : L
{% endhighlight %}

Any guess? If not, then take a look at some more outputs:
{% highlight plaintext %}
87   : LXXXVII
123  : CXXIII
1846 : MDCCCXLVI
{% endhighlight %}

There is even a [annual contest for obfuscated C code](http://www.ioccc.org/) on [http://www.ioccc.org/](http://www.ioccc.org/). In case you are interested in submitting an entry for the next contest, the degree of the obfuscation should be significantly higher than in this trivial example. The winning entries of the past years are really worth taking a look at.
