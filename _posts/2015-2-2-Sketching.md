---
layout: post
title: Sketchy Business Practices and Higher Throughput? An Examination of Applications of Sketching Algorithms.
category: Data Science

---

Here we will examine the implementation of Hyperloglog (a sketching algorithm) which estimates SET counts on a collection of streaming objects or even a static collection. The idea is simple. Taking the objects in the stream or collection you apply a set of hasing functions that map objects to a series of bits and then instead of having to compare objects you just compare bits thus greatly reducing space and computational complexity. You do sacrifice exactness, but for most business practices an accurate representation of the data will almost always beat out the brute force approach of obtaining exact counts.

Paper with implementation of the algorithm [here](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf).

{% highlight python %}
import shakespeare
import sys
import mmh3
import math
import collections

class HyperLogLog:
    def __init__(self, lgm):
        self.lgm = lgm
        self.size = 1 << lgm # allocation of bits proportional to the size allocation given
        self.data = [0]*self.size 
        self.alpham = (0.7213 / (1 + 1.079 / self.size)) * self.size * self.size # this is an approximation of the formula given by Equation 3 in the paper above
        
    def add(self, o):
        x = mmh3.hash(str(o), 0)

        a, b = 32-self.lgm, self.lgm

        i = x >> a
        v = self._reserveAllocation(x << b, a)
        
        self.data[i] = max(self.data[i], v)
        
    def cardinality(self):
        Z = self.alpham / sum([2**-v for v in self.data]) # indicator function as mentioned in the paper
        if Z <= 2.5 * self.size:
            zeros = float(self.data.count(0))
            return round(-self.size * math.log(zeros / self.size))
        else:
            return round(Z)
        
    def _reserveAllocation(self, x, size):
        v = 1
        while v<=size and not x&0x80000000:
            v+=1
            x<<=1
        return v

# sample input shakespeare corpus, can be replaced with other word sets

if __name__ == '__main__':
    H = HyperLogLog(16)
    for word in shakespeare.all_words():
        H.add(word)
        

    shakespeare.print_counter()
    print 'HyperLogLog counted', H.cardinality()
{% endhighlight %}
