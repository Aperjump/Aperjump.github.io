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
