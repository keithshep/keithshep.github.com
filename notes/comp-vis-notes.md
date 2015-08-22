---
title: Image Analysis/Computer Vision Notes
layout: notes
---

# Image Analysis/Computer Vision Notes

__Warning__: These notes are a work in progress and probably contain some errors. Please let me know at [keithshep@gmail.com](mailto:keithshep@gmail.com) know if you spot any.

## Resources

Books:
* *Practical Algorithms for Image Analysis, Second Edition* by O'Gorman, Sammon and Seul
* *Image Processing, The Fundamentals, Second Edition* by Petrou
* *Computer Vision: Algorithms and Applications* free PDF at http://szeliski.org/Book/
* *Bayesian Core: A practical approach to computational bayesian statistics* by martin and Robert (there is a chapter 8 on "Image Analysis")
* *Computer Vision: models learning and inference* free PDF, answers and slides at http://www.computervisionmodels.com/

Online Machine Vision Courses/Videos:

* *Computer Vision: The Fundamentals* https://www.coursera.org/course/vision

## Linear Filtering

correlation: $ g(i, j) = \sum_{k,l} f(i+k, i+j)h(k,l) $

convolution: $ g(i, j) = \sum_{k,l} f(i-k, i-j)h(k,l) $
