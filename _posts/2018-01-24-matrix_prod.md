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
// Dispatcher
template<class E1, class E2>
BOOST_UBLAS_INLINE
typename matrix_matrix_binary_traits<typename E1::value_type, E1,
     typename E2::value_type, E2>::result_type
prod (const matrix_expression<E1> &e1,
      const matrix_expression<E2> &e2) {
    typedef BOOST_UBLAS_TYPENAME matrix_matrix_binary_traits<BOOST_UBLAS_TYPENAME E1::value_type, E1,
     BOOST_UBLAS_TYPENAME E2::value_type, E2>::storage_category storage_category;
    typedef BOOST_UBLAS_TYPENAME matrix_matrix_binary_traits<BOOST_UBLAS_TYPENAME E1::value_type, E1,
     BOOST_UBLAS_TYPENAME E2::value_type, E2>::orientation_category orientation_category;
    return prod (e1, e2, storage_category (), orientation_category ());
}
```
Scary at first, but this function only add two paramters `storage_category` and `orientation_category()` to `prob`. Next, our leading actor comes in:
```
template<class E1, class E2>
BOOST_UBLAS_INLINE
typename matrix_matrix_binary_traits<typename E1::value_type, E1,
     typename E2::value_type, E2>::result_type
prod (const matrix_expression<E1> &e1,
      const matrix_expression<E2> &e2,
      unknown_storage_tag,
      unknown_orientation_tag) {
    typedef BOOST_UBLAS_TYPENAME matrix_matrix_binary_traits<BOOST_UBLAS_TYPENAME E1::value_type, E1,
     BOOST_UBLAS_TYPENAME E2::value_type, E2>::expression_type expression_type;
    return expression_type (e1 (), e2 ());
}
```
In previous passage, I have seen this function multiple times, and they share some common property:
1. use `traits` to determine `expression_type` and `result_type`.
2. return one `expression_type`, and it plays the evaluation part. 

What does traits look like:
```
template<class T1, class E1, class T2, class E2>
struct matrix_matrix_binary_traits {
    typedef unknown_storage_tag storage_category;
    typedef unknown_orientation_tag orientation_category;
    typedef typename promote_traits<T1, T2>::promote_type promote_type;
    typedef matrix_matrix_binary<typename E1::const_closure_type,
     typename E2::const_closure_type,
     matrix_matrix_prod<T1, T2, promote_type> > expression_type;
#ifdef BOOST_UBLAS_USE_ET
    typedef expression_type result_type;
#else
    typedef matrix<promote_type> result_type;
#endif
};
```
Two main features is `promote_traits` and `expression_type`. The first one is simpler, and it converts two `value_type` to one larger type, such as one float type and one double will promoted to double. 
`eexpression_type` is a recursive definition, cause `const_closure` type is just the expression type itself. and with `matrix_vector_binary2`, it treats the last type `matrix_vector_prod2<T1, T2, promote_type>` a functor. 
### 2. from functor to binary/unary class
Here, we have finished the type deduction step for compiler, and need to initialize some real classes. 
As we have seen, `expression_type` is a `matrix_matrix_binary` class and this class has one `operator ()` which accepts two `matrix_expression` type. 
```
matrix_matrix_binary (const expression1_type &e1, const expression2_type &e2): 
    e1_ (e1), e2_ (e2) {}
```
This two copy assignment operator will not actually do the copy work, their evaluation happens when assigned to the left hand side `matrix`. 
### 3. from binary/unary class to matrix class 