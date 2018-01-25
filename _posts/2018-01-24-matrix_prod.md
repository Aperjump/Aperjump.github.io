---
layout: post
title: "The Annotated uBLAS Sources 5 Matrix_Prod"
description: "my notes on reading uBLAS code5"
categories: [C++]
tags: [C++, Template]
redirect_from:
  - /2018/01/24/
---
## Matrix Prod

This passage I try to discuss the structure of `boost::ublas`. After analyzing many functionality of `vector` and `matrix`, I begin to grasp the idea of this project and try to break it down with an exaple of `ublas::prod`. 


```
m3 = ublas::prod(m1, m2);
```
`ublas` break the idea of matrix, vector operation into different concepts layers:

- expression
- unary, binary expression
- unary, binary functor
- operation

There are many auxillary parts besides these, like iterator, storage, and assignment. And I will try to group them together to form a big picture. 

### 1. from operation to functor
`ublas:prob(matrix, matrix)` calls 
```
template<class E1, class E2>
BOOST_UBLAS_INLINE
typename matrix_vector_binary2_traits<typename E1::value_type, E1,
      typename E2::value_type, E2>::result_type
prod (const vector_expression<E1> &e1,
      const matrix_expression<E2> &e2) {
    typedef BOOST_UBLAS_TYPENAME matrix_vector_binary2_traits<BOOST_UBLAS_TYPENAME E1::value_type, E1,
      BOOST_UBLAS_TYPENAME E2::value_type, E2>::storage_category storage_category;
    typedef BOOST_UBLAS_TYPENAME matrix_vector_binary2_traits<BOOST_UBLAS_TYPENAME E1::value_type, E1,
      BOOST_UBLAS_TYPENAME E2::value_type, E2>::orientation_category orientation_category;
    return prod (e1, e2, storage_category (), orientation_category ());
}
```
Scary at first, but this function only add two paramters `storage_category` and `orientation_category()` to `prob`. Next, our leading actor comes in:
```
template<class E1, class E2>
BOOST_UBLAS_INLINE
typename matrix_vector_binary2_traits<typename E1::value_type, E1,
      typename E2::value_type, E2>::result_type
prod (const vector_expression<E1> &e1,
      const matrix_expression<E2> &e2,
      unknown_storage_tag,
      column_major_tag) {
    typedef BOOST_UBLAS_TYPENAME matrix_vector_binary2_traits<BOOST_UBLAS_TYPENAME E1::value_type, E1,
      BOOST_UBLAS_TYPENAME E2::value_type, E2>::expression_type expression_type;
    return expression_type (e1 (), e2 ());
}
```
In previous passage, I have seen this function multiple times, and they share one common interface:
1. use `traits` to determine `expression_type` and `result_type`.
2. return one `expression_type`, and it plays the evaluation part. 