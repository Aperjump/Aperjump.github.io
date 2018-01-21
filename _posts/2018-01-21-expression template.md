---
layout: post
title: "The Annotated uBLAS Sources 2"
description: "my notes on reading uBLAS code1"
categories: [C++]
tags: [C++, Template]
redirect_from:
  - /2018/01/21/
---

## Expression Instantiation
This passage will focus on the instantiation process of one vector `ublas::vector<float, ublas::unbounded_array<float>` through `gdb`. 
I find test cases in `/boost_1.29.0/libs/numeric/ublas/test1/` directory, and compiled them with `bjam`. I will extract main executing path and make a brief summary. 
```
First Initialization
--------------------------------
v1 = (boost::numeric::ublas::vector<float, boost::numeric::ublas::unbounded_array<float> > &) @0x7fffffffe040: {<boost::numeric::ublas::vector_expression<boost::numeric::ublas::vector<float, boost::numeri
c::ublas::unbounded_array<float> > >> = {<No data fields>}, size_ = 3, data_ = {size_ = 3, data_ = 0x69c030}}
v1.size_ = 3
v1.data_ = {size_ = 3, data_ = 0x69c030}
v1.data_.data_ = (boost::numeric::ublas::unbounded_array<float>::pointer) 0x69c030

v2: (boost::numeric::ublas::vector<float, boost::numeric::ublas::unbounded_array<float> > &) @0x7fffffffe060: {<boost::numeric::ublas::vector_expression<boost::numeric::ublas::vector<float, boost::numeri
c::ublas::unbounded_array<float> > >> = {<No data fields>}, size_ = 3, data_ = {size_ = 3, data_ = 0x69c050}}
v2.size_ = 3
v2.data_ = {size_ = 3, data_ = 0x69c030}
v1.data_.data_ = (boost::numeric::ublas::unbounded_array<float>::pointer) 0x69c050)
--------------------------------
Copy assginment operator v1 = v2
data () = v.data ();
--------------------------------

```
