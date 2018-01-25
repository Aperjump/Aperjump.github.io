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
This `matrix_unary1_traits` type can nest multiple levels of expression. In `matrix_unary1`, `matrix_unary1<E,F>` is defined as `const_closure_type`, and through the overriding version of `operator ()`, `expression()` can initialize its value through calling `matrix_unary1::operator ()`. 
```
----------------
m1 = conj(m2);
----------------
template<class E> 
BOOST_UBLAS_INLINE
typename matrix_unary1_traits<E, scalar_conj<typename E::value_type> >::result_type
conj (const matrix_expression<E> &e) {
    typedef BOOST_UBLAS_TYPENAME matrix_unary1_traits<E, scalar_conj<BOOST_UBLAS_TYPENAME E::value_type> >::expression_type expression_type;
    return expression_type (e ());
}
----------------
m1 = m1 + m2;
----------------
template<class E1, class E2>
BOOST_UBLAS_INLINE
typename matrix_binary_traits<E1, E2, scalar_plus<typename E1::value_type, 
      typename E2::value_type> >::result_type
operator + (const matrix_expression<E1> &e1, 
            const matrix_expression<E2> &e2) {
    typedef BOOST_UBLAS_TYPENAME matrix_binary_traits<E1, E2, scalar_plus<BOOST_UBLAS_TYPENAME E1::value_type, 
      BOOST_UBLAS_TYPENAME E2::value_type> >::expression_type expression_type;
    return expression_type (e1 (), e2 ());
}
-----------------
m2 = ublas::trans(m1);
----------------
// (trans m) [i] [j] = m [j] [i]
template<class E>
BOOST_UBLAS_INLINE
typename matrix_unary2_traits<E, scalar_identity<typename E::value_type> >::result_type
trans (const matrix_expression<E> &e) {
    typedef BOOST_UBLAS_TYPENAME matrix_unary2_traits<E, scalar_identity<BOOST_UBLAS_TYPENAME E::value_type> >::expression_type expression_type;
    return expression_type (e ());
}
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
};
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

s