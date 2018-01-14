---
layout: post
title: "The Annotated uBLAS Sources 1"
description: "my notes on reading uBLAS code"
categories: [C++]
tags: [C++, Template]
redirect_from:
  - /2018/01/13/
---

## 1. Intro to the series
Hi, I'm a finance student but have great enthusiam in C++. Currently, I'm reading the source code of `Boost.uBLAS`. Starting with `vector_expression.hpp` and `vector.hpp`, I find the authors repeatedly using **CRTP(curiously recurring template pattern)** to define `vector`, `unit_vector` and `zero_vector` class. So, I believe it necessary to collect more information on this and make a brief summary. 
This blog series will focuse on different implementation of `Boost.uBLAS` library and potential method to accelerate it. Most of the source code comes from the `boost-1.29.0`, the first version of `Boost.uBLAS`, and at the end, I will compare its differences with later version.
The source code can be found [here](https://github.com/boostorg/ublas/releases?after=boost-1.34.0-beta1).
## 2. CRTP

```
//boost/numeric/uBLAS/vector_expression.hpp
template<class E>
struct vector_expression {
    typedef E expression_type;
    typedef vector_tag type_category;
    typedef const vector_range<const E> const_vector_range_type;
    typedef vector_range<E> vector_range_type;
    ...// many typdefs
    BOOST_UBLAS_INLINE
    const expression_type &operator () () const {
        return *static_cast<const expression_type *> (this);
    }
    BOOST_UBLAS_INLINE
    expression_type &operator () () {
        return *static_cast<expression_type *> (this);
    }
    ...
};
//boost/numeric/uBLAS/vector.hpp
template<class T, class A>
class vector:
    public vector_expression<vector<T, A> > {
    ...
    private:
        size_type size_;
        array_type data_;
};
template<class T>
class unit_vector:
    public vector_expression<unit_vector<T> > {
        ...
    private:
    size_type size_;
    size_type index_;
    static value_type zero_;
    static value_type one_;
};
template<class T>
    class zero_vector:
    public vector_expression<zero_vector<T> > {
    ...
    private:
    size_type size_;
    static value_type zero_;
};
```
It's basic form is that derived class inherit from one template base class, but the template paramter is the derived class itself.
```
template <typename T>
class Base
{
    ...
};
 // use the derived class itself as a template parameter of the base class
class Derived : public Base<Derived>
{
    ...
};
```
This pattern can be used to implement static polymorphism. Mostly, base class vill have the same interface, but it will use `static_cast` to transform its type to its template parameter. Inheritant class will call its base class interface and through type casting, this function call will actually calling their own functions. Through this mechanism, compiler can determine which function to call during compile time, which certainly brings us efficiency. 
Another way to achieve polymorphism is what we call **dynamic polymorphism**, but this method is implemented by virtual table and is really slow.
```
template <typename T>
class Base
{
public:
    void method() {
        static_cast<T*>(this)->method();
    }
};
class Derived1 : public Base<Derived1>
{
public:
    void method() {
        std::cout << "Derived1 method" << std::endl;
    }
};
class Derived2 : public Base<Derived2>
{
public:
    void method() {
        std::cout << "Derived2 method" << std::endl;
    }
};
int main()
{
    Derived1 d1;
    Derived2 d2;
    d1.method();
    d2.method();
    return 0;
}
```
I'd like to quote some words from [Jonathan Boccara](https://www.fluentcpp.com/2017/05/16/what-the-crtp-brings-to-code/), who said:

> 
With the CRTP the situation is radically different. The derived class does not express the fact it “is a” base class. Rather, it expands its interface by inherting from the base class, in order to add more functionality. In this case it makes sense to use the derived class directly, and to never use the base class.
Therefore the base class is not the interface, and the derived class is not the implementation. Rather, it is the other way around: the base class uses the derived class methods . In this regard, **the derived class offers an interface to the base class**. This illustrates again the fact that inheritance in the context of the CRTP can express quite a different thing from classical inheritance.

## 3. Usage in `Boost.uBLAS`
```
int main () {
    using namespace boost::numeric::ublas;
    vector<std::complex<double> > v (3);
    for (int i = 0; i < v.size (); ++ i) 
        v (i) = std::complex (i, i);
    std::cout << conj (v) << std::endl;
}
--------------------------------------
vector<std::complex<double> > 
        | // class vector : public vector_expression<vector<T, A> >
        | // T = complex<double>, A = unbounded_array<comeplex<double> >
vector<std::complex<double> >::size_ = complex<double>
        |
conj(v)
        | // typename vector_unary_traits<E, scalar_conj<typename E::value_type> >::result_type conj (const vector_expression<E> &e)
typedef BOOST_UBLAS_TYPENAME vector_unary_traits<E, scalar_conj<BOOST_UBLAS_TYPENAME E::value_type> >::expression_type expression_type;
        | // E = vector<complex<duoble> >
vector_unary_traits<E, scalar_conj<typename E::value_type> >
        | // E::value_type = complex<T>
typedef vector_unary<typename E::const_closure_type, F> expression_type;
        | // in `template<class E, class F> vector_unary_traits`
        | // F = scalar_conj<E::value_type>
scalar_conj<complex<double> >
        | //     struct scalar_conj: public scalar_unary_functor<T> {
        | //     typedef typename scalar_unary_functor<T>::argument_type argument_type;
        | //    typedef typename scalar_unary_functor<T>::result_type result_type;
        | //    result_type operator () (argument_type t) const {
        | //    return type_traits<result_type>::conj(t);
        | //};
struct type_traits<std::complex<double> > // traits Specialization
        | typeof(t) = complex<double>
return std::conj (t)
```
This is one instantialization path of `conj(t)` function. But it's main idea is find return type and parameter type of the function by calling one template function. 
Through its parameter type`vector<complex<double> >`, this function instantialize one `vector_unary_traits`, and finally find the correct return type. 
The `CRTP` pattern here mainly provides interface.
```
template<class E> 
    BOOST_UBLAS_INLINE
    typename vector_unary_traits<E, scalar_conj<typename E::value_type> >::result_type
    conj (const vector_expression<E> &e) {
        typedef BOOST_UBLAS_TYPENAME vector_unary_traits<E, scalar_conj<BOOST_UBLAS_TYPENAME E::value_type> >::expression_type expression_type;
        return expression_type (e ());
    }
```
This function receive `vector_expression<E>` variable and use it to deduct its return type. Also `vector_expression` defines one hierachy of class types. 