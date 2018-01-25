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

In this blog I will try to discuss the structure of `boost::ublas`. After analyzing many function of `vector` and `matrix`, I begin to grasp the idea of this project and try to break it down with an example of `ublas::prod`. 

```
m3 = ublas::prod(m1, m2);
```
`ublas` break the idea of matrix, vector operation into different concepts layers:

- expression
- unary, binary expression
- unary, binary functor
- operation

There are many auxillary parts besides these, like iterator, storage, and assignment. Since their implementation implies lower-level facilities, I will not discuss them in great detail and leave them to readers. 

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
Though Scary at first, this function only add two paramters `storage_category` and `orientation_category()` to `prob`. Next, our leading actor comes in:
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
In previous passage, I have seen this function multiple times, and they share some common properties:
1. use `traits` to determine `expression_type` and `result_type`.
2. return one `expression_type`, and it doesn't have  evaluation function. 

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
Two main features is `promote_traits` and `expression_type`. The first one is simpler, since it converts two `value_type` to one larger type, for example one float type and one double type will promoted to double. 
`expression_type` is a recursive definition, cause `const_closure` type is just the expression type itself. and with `matrix_vector_binary2`, it treats the last type `matrix_vector_prod2<T1, T2, promote_type>` a functor. 

### 2. from functor to binary/unary class
Here, we have finished the type deduction step for compiler, and need to initialize some real classes. 
As we have seen, `expression_type` is a `matrix_matrix_binary` class and this class has one `operator ()` which accepts two `matrix_expression` type. 
```
matrix_matrix_binary (const expression1_type &e1, const expression2_type &e2): 
    e1_ (e1), e2_ (e2) {}
```
This two copy assignment operator will not actually do the copy work, their evaluation happens when assigned to the left hand side `matrix`. 

### 3. from binary/unary class to matrix class 
This function will do the rest part:
```
template<class AE>
BOOST_UBLAS_INLINE
matrix (const matrix_expression<AE> &ae):
    size1_ (ae ().size1 ()), size2_ (ae ().size2 ()), data_ (ae ().size1 () * ae ().size2 ()) { 
    matrix_assign<scalar_assign<value_type, BOOST_UBLAS_TYPENAME AE::value_type> > () (*this, ae);
}
```
After copying all the data from the expression, it calls one `matrix_assign` function. We have seen this before and the rest comes easily:
```
// Dispatcher
template<class M, class E>
BOOST_UBLAS_INLINE
void operator () (M &m, const matrix_expression<E> &e) {
    typedef typename matrix_assign_traits<BOOST_UBLAS_TYPENAME M::storage_category,
      assign_category,
      BOOST_UBLAS_TYPENAME E::const_iterator1::iterator_category,
      BOOST_UBLAS_TYPENAME E::const_iterator2::iterator_category>::storage_category storage_category;
    // FIXME: can't we improve the dispatch here?
    // typedef typename E::orientation_category orientation_category;
    typedef typename M::orientation_category orientation_category;
#ifndef BOOST_UBLAS_ENABLE_SPECIALIZED_ASSIGN
    operator () (m, e, storage_category (), orientation_category ());
#else
    evaluate_matrix_assign (functor_type (), m, e, storage_category (), orientation_category ());
#endif
}
```
`evaluate_matrix_assign` did most of the work and transfer to `index_matrix_assign` or `iterating_matrix_assign`. 
```
void evaluate_matrix_assign (const F &f, M &m, const matrix_expression<E> &e, dense_proxy_tag, C c) {
    typedef F functor_type;
    typedef C orientation_category;
#ifdef BOOST_UBLAS_USE_INDEXING
    indexing_matrix_assign (functor_type (), m, e, orientation_category ());
#elif BOOST_UBLAS_USE_ITERATING
    iterating_matrix_assign (functor_type (), m, e, orientation_category ());
#else
    typedef typename M::difference_type difference_type;
    difference_type size1 (BOOST_UBLAS_SAME (m.size1 (), e ().size1 ()));
    difference_type size2 (BOOST_UBLAS_SAME (m.size2 (), e ().size2 ()));
    if (size1 >= BOOST_UBLAS_ITERATOR_THRESHOLD &&
        size2 >= BOOST_UBLAS_ITERATOR_THRESHOLD)
        iterating_matrix_assign (functor_type (), m, e, orientation_category ());
    else
        indexing_matrix_assign (functor_type (), m, e, orientation_category ());
#endif
}
```
And inside its loop, this function will call `functor_type () (m (i, j), e () (i, j));`
This fnction resides in `matrix_matirx_binary` class.
```
BOOST_UBLAS_INLINE
const_reference operator () (size_type i, size_type j) const { 
    return functor_type () (e1_, e2_, i, j);
}
```

### 4. from matrix_class to functor
The last part is the evlaution of functor class. 
With `matrix_matrix_prob` type, we will call its `operator ()`:
```
template<class E1, class E2>
BOOST_UBLAS_INLINE
result_type operator () (const matrix_expression<E1> &e1,
     const matrix_expression<E2> &e2,
     size_type i, size_type j) const {
    size_type size = BOOST_UBLAS_SAME (e1 ().size2 (), e2 ().size1 ());
    result_type t (0);
#ifndef BOOST_UBLAS_USE_DUFF_DEVICE
    for (size_type k = 0; k < size; ++ k)
        t += e1 () (i, k) * e2 () (k, j);
#else
    size_type k (0);
    DD (size, 4, r, (t += e1 () (i, k) * e2 () (k, j), ++ k));
#endif
    return t;
}
```
This function will access each element in `e1` and `e2`, and sum them up to get one element. This is slow and I believe I can think of method to solve this performance bottleneck with GPU. 