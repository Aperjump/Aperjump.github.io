---
layout: post
title: "Explanation on VC Dimension"
description: "Explanantion on VC Dimension"
categories: [Machine Learning]
tags: [Machine Learning]
redirect_from:
  - /2018/02/15/
---

## VC Dimension Intro
VC theory is probably the first hard concept we encountered in Machine Learning theory, so I'd like to make a one user-friendly introduction for this topic.
There are two main issues in machine learning:
(1) Algorithm design: how we generate rules to interpret data 
(2) Genralization : how to guarantee model performance on data we haven't see
VC Dimension is in the second category, its' not one specific algorithm but some high-level rule for algorithm design. 
After you find some data and use whatever algorithm learned in class, there must be one question you want to ask: will my model still work for future data?

