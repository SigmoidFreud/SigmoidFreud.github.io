---
layout: post
title: Quora Answer Classification Problem
category: Data Science

---

This problem featured [here](http://www.quora.com/challenges#answer_classifier) seemed interesting and it was a good 
place for me to try something new because of the mix of categorical and continuous data fields. I implemented a feature ranking algorithm featured 
[here](http://perso.uclouvain.be/michel.verleysen/papers/kdir11gd.pdf). It is a filter wrapper approach that measures the importance
of each feature and ranks them accordingly. 

The results I got and all of the code is featured in the following [notebook](http://nbviewer.ipython.org/github/SigmoidFreud/quoraDataChallenge/blob/master/quoraChallenge/quoraChallenge.ipynb)
