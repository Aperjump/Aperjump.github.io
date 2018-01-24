---
layout: post
title: "The Annotated uBLAS Sources 4 Matrix Expression Template"
description: "my notes on reading uBLAS code4"
categories: [C++]
tags: [C++, Template]
redirect_from:
  - /2018/01/23/
---
## Matrix Expression Template
matrix class has three member `size_t size1_`, `size_t size2_`, `array_type data_`. The last element is just the same as `vector` class array_type and is default to be `unbounded_array`. 
```
----------------
m1 = m2;
----------------
matrix &operator = (const matrix &m) { 
    BOOST_UBLAS_CHECK (size1_ == m.size1_, bad_size ());
    BOOST_UBLAS_CHECK (size2_ == m.size2_, bad_size ());
    size1_ = m.size1_;
    size2_ = m.size2_;
    data () = m.data ();
    return *this;
} // This function can have potential improvement by GPU.
----------------
m1.swap(m2);
----------------
// Swapping
BOOST_UBLAS_INLINE
void swap (matrix &m) {
    // Too unusual semantic.
    // BOOST_UBLAS_CHECK (this != &m, external_logic ());
    if (this != &m) {
        // Precondition for container relaxed as requested during review.
        // BOOST_UBLAS_CHECK (size1_ == m.size1_, bad_size ());
        // BOOST_UBLAS_CHECK (size2_ == m.size2_, bad_size ());
        std::swap (size1_, m.size1_);
        std::swap (size2_, m.size2_);
        data ().swap (m.data ());
    }
}
----------------
m1 = -m2;
----------------
template<class E>
BOOST_UBLAS_INLINE
typename matrix_unary1_traits<E, scalar_negate<typename E::value_type> >::result_type
operator - (const matrix_expression<E> &e) {
    typedef BOOST_UBLAS_TYPENAME matrix_unary1_traits<E, scalar_negate<BOOST_UBLAS_TYPENAME E::value_type> >::expression_type expression_type;
    return expression_type (e ());
}
template<class E, class F>
struct matrix_unary1_traits {
    typedef matrix_unary1<typename E::const_closure_type, F> expression_type;
#ifdef BOOST_UBLAS_USE_ET
    typedef expression_type result_type; 
#else
    typedef matrix<typename F::result_type> result_type;
#endif
};
template<class E, class F>
class matrix_unary1:
    public matrix_expression<matrix_unary1<E, F> > {
public:
#ifdef BOOST_UBLAS_ENABLE_PROXY_SHORTCUTS
    BOOST_UBLAS_USING matrix_expression<matrix_unary1<E, F> >::operator ();
#endif
    typedef E expression_type;
    typedef F functor_type;
    typedef typename E::size_type size_type;
    typedef typename E::difference_type difference_type;
    typedef typename F::result_type value_type;
    typedef value_type const_reference;
    typedef const_reference reference;
    typedef const value_type *const_pointer;
    typedef const_pointer pointer;
    typedef const matrix_unary1<E, F> const_closure_type;
    typedef typename E::orientation_category orientation_category;
    typedef typename E::const_iterator1 const_iterator1_type;
    typedef typename E::const_iterator2 const_iterator2_type;
    typedef unknown_storage_tag storage_category;
};
```