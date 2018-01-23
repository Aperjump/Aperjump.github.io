---
layout: post
title: "The Annotated uBLAS Sources 3 Vector Proxy"
description: "my notes on reading uBLAS code3"
categories: [C++]
tags: [C++, Template]
redirect_from:
  - /2018/01/22/
---
## Vector Proxy 
Under `boost 1.29`, vector proxy is one part of `vector_expression` class. 
```
//vector_expression
BOOST_UBLAS_INLINE
    const_vector_range_type operator () (const range<> &r) const {
        return const_vector_range_type (operator () (), r);
    }
    BOOST_UBLAS_INLINE
    vector_range_type operator () (const range<> &r) {
        return vector_range_type (operator () (), r);
    }
    BOOST_UBLAS_INLINE
    const_vector_slice_type operator () (const slice<> &s) const {
        return const_vector_slice_type (operator () (), s);
    }
    BOOST_UBLAS_INLINE
    vector_slice_type operator () (const slice<> &s) {
        return vector_slice_type (operator () (), s);
    }
```
Every time calling `expression_type(range)`, it will pass to these functions and instantiate one `range` or `slice` type. So it's necessary to find how these classes work. 
```
----------------------
v1 = v2;
----------------------
BOOST_UBLAS_INLINE
vector_range &operator = (const vector_range &vr) {
    // FIXME: the ranges could be differently sized.
    // std::copy (vr.begin (), vr.end (), begin ());
    vector_assign<scalar_assign<value_type, value_type> > () (*this, vector<value_type> (vr));
    return *this;
}
```
This function will first treat `vr` as a `vector_expression`, then convert it to a `vector`, and finally assign its value to `*this`. This is a convoluted way and I believe there are methods to cut off the road. 
```
//functional.hpp
template<class T1, class T2>
struct scalar_assign:
    public scalar_binary_assign_functor<T1, T2> {
    typedef typename scalar_binary_assign_functor<T1, T2>::argument1_type argument1_type;
    typedef typename scalar_binary_assign_functor<T1, T2>::argument2_type argument2_type;
    typedef assign_tag assign_category;

    BOOST_UBLAS_INLINE
    void operator () (argument1_type t1, argument2_type t2) const {
        t1 = t2;
    }
};
//scalar_assign will be functor_type
//vector_assign.hpp
void operator () (V &v, const vector_expression<E> &e) {
    typedef typename vector_assign_traits<BOOST_UBLAS_TYPENAME V::storage_category,
    assign_category,
    BOOST_UBLAS_TYPENAME E::const_iterator::iterator_category>::storage_category storage_category;
#ifndef BOOST_UBLAS_ENABLE_SPECIALIZED_ASSIGN
    operator () (v, e, storage_category ());
#else
    evaluate_vector_assign (functor_type (), v, e, storage_category ());
#endif
}
```
This is a `functor`, and inside this `operator ()` function will call `operator () (v, e, storage_category ())`, this function call one of the two functions:
```
void iterating_assign (V &v, const vector_expression<E> &e) {
    typedef typename V::difference_type difference_type;
    difference_type size (BOOST_UBLAS_SAME (v.size (), e ().size ()));
    typename V::iterator it (v.begin ());
    BOOST_UBLAS_CHECK (v.end () - it == size, bad_size ());
    typename E::const_iterator ite (e ().begin ());
    BOOST_UBLAS_CHECK (e ().end () - ite == size, bad_size ());
#ifndef BOOST_UBLAS_USE_DUFF_DEVICE
    while (-- size >= 0)
        functor_type () (*it, *ite), ++ it, ++ ite;
#else
    DD (size, 2, r, (functor_type () (*it, *ite), ++ it, ++ ite));
#endif
}
// Indexing case
template<class V, class E>
// This function seems to be big. So we do not let the compiler inline it.
// BOOST_UBLAS_INLINE
void indexing_assign (V &v, const vector_expression<E> &e) {
    typedef typename V::difference_type difference_type;
    difference_type size (BOOST_UBLAS_SAME (v.size (), e ().size ()));
#ifndef BOOST_UBLAS_USE_DUFF_DEVICE
    for (difference_type i = 0; i < size; ++ i)
        functor_type () (v (i), e () (i));
#else
    difference_type i (0);
    DD (size, 2, r, (functor_type () (v (i), e () (i)), ++ i));
#endif
}
```
See? there is one loop inside, that sequential loop makes this assignment operator slow. 