---
layout: post
title: HackerRank problem in machine learning Expert Level. For the record I am not an expert.
category: Data Science

---

I just took a crack at this problem and see what I could do. I had decent and impressive results if I do say so myself. I have not made a formal submission and I have written a decent amount of code for this. I did some munging and divided students into categories. The idea is that students who take the same classes are likely to be better predictors of eachother in terms of ability. I used Support Vector Classification. There are many other to choose from Multinomial Logistic Regression, Random Forest, Extra Trees, etc. I did some PCA and some cross validation to find optimal numbers of features when I did the dimensionality reduction. Finally, I trained some classifiers and saw the final results.
  
Link to the coding challenge [here](https://www.hackerrank.com/challenges/predict-missing-grade)

The entire Ipython notebook is given here where also all the code is written [here](http://nbviewer.ipython.org/github/SigmoidFreud/HackerRankML/blob/master/SVCMathGradePredictor.ipynb).
