---
layout: post
title: "The Annotated uBLAS Sources 2_Vector Expression Template"
description: "my notes on reading uBLAS code2"
categories: [C++]
tags: [C++, Template]
redirect_from:
  - /2018/01/21/
---

## Vector Expression Template
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
v1.data_ = (boost::numeric::ublas::vector<float, boost::numeric::ublas::unbounded_array<float> >::const_array_type &) @0x7fffffffe048: {size_ = 3, data_ = 0x69c030}
v2_data_ = (boost::numeric::ublas::vector<float, boost::numeric::ublas::unbounded_array<float> >::const_array_type &) @0x7fffffffe068: {size_ = 3, data_ = 0x69c050}
```
This copy assignment actually calls 
`unbounded_array &operator = (const unbounded_array &a)`, then 
`std::copy (a.data_, a.data_ + a.size_, data_);`
**vector copy assignment operator is deep copy.**
```
--------------------------------
assign temporary 
swap(v)
--------------------------------
v1.data_ = (boost::numeric::ublas::vector<float, boost::numeric::ublas::unbounded_array<float> >::const_array_type &) @0x7fffffffe048: {size_ = 3, data_ = 0x69c050}
v2_data_ = (boost::numeric::ublas::vector<float, boost::numeric::ublas::unbounded_array<float> >::const_array_type &) @0x7fffffffe068: {size_ = 3, data_ = 0x69c030}
```
`assign_temporary` and `swap` function will only change pointers in two `data_`.
```
--------------------------------
v2 = -v1
--------------------------------
v2 = (boost::numeric::ublas::vector<float, boost::numeric::ublas::unbounded_array<float> > &) @0x7fffffffe060: {<boost::numeric::ublas::vector_expression<boost::numeric::ublas::vector<float, boost::numer
ic::ublas::unbounded_array<float> > >> = {<No data fields>}, size_ = 3, data_ = {size_ = 3, data_ = 0x69c050}}
```
Here will use the first expression template. After a long template parameter deduction, we can find its expression type only changes result type. 
```
typename vector_unary_traits<E, scalar_negate<typename E::value_type> >::result_type 
operator - (const vector_expression<E> &e) {
    typedef typename vector_unary_traits<E, scalar_negate<typename E::value_type> >::expression_type expression_type;
    return expression_type(e());
}
template<class E, class F>
struct vector_unary_traits {
    typedef vector_unary<typename E::const_closure_type, F> expression_type;
    //typedef const vector_unary<E, F> const_closure_type;
#ifdef BOOST_UBLAS_USE_ET
    typedef expression_type result_type; 
#else
    typedef vector<typename F::result_type> result_type;
#endif
};
template<class T>
struct scalar_negate:
    public scalar_unary_functor<T> {
    typedef typename scalar_unary_functor<T>::argument_type argument_type;
    typedef typename scalar_unary_functor<T>::result_type result_type;

    BOOST_UBLAS_INLINE
    result_type operator () (argument_type t) const {
        return - t;
    }
};
BOOST_UBLAS_INLINE
const expression_type &operator () () const {
    return *static_cast<const expression_type *> (this);
}
```
If we try to access `v2.data_.data_[1]`, you will find its value has changed to `-1`, which happens during the copy assignment operator through `std::copy`.

```
--------------------------------
v2 = ublas::conj(v1);
--------------------------------
template<class E> 
BOOST_UBLAS_INLINE
typename vector_unary_traits<E, scalar_conj<typename E::value_type> >::result_type
conj (const vector_expression<E> &e) {
    typedef BOOST_UBLAS_TYPENAME vector_unary_traits<E, scalar_conj<BOOST_UBLAS_TYPENAME E::value_type> >::expression_type expression_type;
    return expression_type (e ());
}
template<class T>
struct scalar_conj:
    public scalar_unary_functor<T> {
    typedef typename scalar_unary_functor<T>::argument_type argument_type;
    typedef typename scalar_unary_functor<T>::result_type result_type;

    BOOST_UBLAS_INLINE
    result_type operator () (argument_type t) const {
        return type_traits<result_type>::conj (t);
    }
};
```
Here `scalar_conj` transfer the operation to a range of `type_traits` specialization. We can always find one in `/boost/numeric/ublas/traist.hpp` file.
```
--------------------------------
v3 = v1 + v2
--------------------------------
template<class E1, class E2>
BOOST_UBLAS_INLINE
typename vector_binary_traits<E1, E2, scalar_plus<typename E1::value_type, 
                                                  typename E2::value_type> >::result_type
operator + (const vector_expression<E1> &e1, 
            const vector_expression<E2> &e2) {
    typedef BOOST_UBLAS_TYPENAME vector_binary_traits<E1, E2, scalar_plus<BOOST_UBLAS_TYPENAME E1::value_type, 
      BOOST_UBLAS_TYPENAME E2::value_type> >::expression_type expression_type;
    return expression_type (e1 (), e2 ());
}
template<class T1, class T2>
struct scalar_plus:
    public scalar_binary_functor<T1, T2> {
    typedef typename scalar_binary_functor<T1, T2>::argument1_type argument1_type;
    typedef typename scalar_binary_functor<T1, T2>::argument2_type argument2_type;
    typedef typename scalar_binary_functor<T1, T2>::result_type result_type;

    BOOST_UBLAS_INLINE
    result_type operator () (argument1_type t1, argument2_type t2) const {
        return t1 + t2;
    }
};
```
This `operator +` works as before, it won't evaluate `v1 + v2` until assignment. Here comes the pro and cons of lazy evaluation, the idea of which is to delay evaluating one expression until truly needed. It does relieve memory burnden and computation resource, but when meed large loop especially we need to access each element, this method is too slow compared to modern GPU parallel computing. The penalty is less obvious on `vector` type, when we come to `matrix` class, I will dive into this topic. 
```
--------------------------------
v3 = value_type(1) * v2;
--------------------------------
template<class T1, class E2>
BOOST_UBLAS_INLINE
typename vector_binary_scalar1_traits<T1, E2, scalar_multiplies<T1, typename E2::value_type> >::result_type
operator * (const T1 &e1, 
            const vector_expression<E2> &e2) {
    typedef BOOST_UBLAS_TYPENAME vector_binary_scalar1_traits<T1, E2, scalar_multiplies<T1, BOOST_UBLAS_TYPENAME E2::value_type> >::expression_type expression_type;
    return expression_type (e1, e2 ());
}
--------------------------------
t = ublas::inner_prob(v1, v2);
--------------------------------
// inner_prod (v1, v2) = sum (v1 [i] * v2 [i]
template<class E1, class E2>
BOOST_UBLAS_INLINE
typename vector_scalar_binary_traits<E1, E2, vector_inner_prod<typename E1::value_type, 
          typename E2::value_type,
          typename promote_traits<typename E1::value_type,
          typename E2::value_type>::promote_type> >::result_type 
inner_prod (const vector_expression<E1> &e1, 
            const vector_expression<E2> &e2) {
    typedef BOOST_UBLAS_TYPENAME vector_scalar_binary_traits<E1, E2, vector_inner_prod<BOOST_UBLAS_TYPENAME E1::value_type, 
        BOOST_UBLAS_TYPENAME E2::value_type,
        BOOST_UBLAS_TYPENAME promote_traits<BOOST_UBLAS_TYPENAME E1::value_type, 
        BOOST_UBLAS_TYPENAME E2::value_type>::promote_type> >::expression_type expression_type;
    return expression_type (e1 (), e2 ());
}
template<class T1, class T2, class TR>
struct vector_inner_prod: 
    public vector_scalar_binary_functor<T1, T2, TR> {
    typedef typename vector_scalar_binary_functor<T1, T2, TR>::size_type size_type ;
    typedef typename vector_scalar_binary_functor<T1, T2, TR>::difference_type difference_type;
    typedef typename vector_scalar_binary_functor<T1, T2, TR>::value_type value_type;
    typedef typename vector_scalar_binary_functor<T1, T2, TR>::result_type result_type;

    template<class E1, class E2>
    BOOST_UBLAS_INLINE
    result_type operator () (const vector_expression<E1> &e1, 
                             const vector_expression<E2> &e2) const { 
        result_type t (0);
        size_type size (BOOST_UBLAS_SAME (e1 ().size (), e2 ().size ()));
#ifndef BOOST_UBLAS_USE_DUFF_DEVICE
        for (size_type i = 0; i < size; ++ i)
            t += e1 () (i) * e2 () (i);
#else
        size_type i (0);
        DD (size, 4, r, (t += e1 () (i) * e2 () (i), ++ i));
#endif
        return t; 
    }
```