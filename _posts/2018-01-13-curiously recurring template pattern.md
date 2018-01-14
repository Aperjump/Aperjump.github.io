---
layout: post
title: "The Annotated uBLAS Sources"
description: "my notes on reading uBLAS code"
categories: [C++]
tags: [C++, Template]
redirect_from:
  - /2018/01/13/
---

## 1. Intro to the series
Hi, I'm a finance student but with great enthusiam in C++. Currently, I'm reading the source code of `Boost.uBLAS`. Starting with `vector_expression.hpp` and `vector.hpp`, I find the authors repeatedly using **CRTP(curiously recurring template pattern)** to define `vector`, `unit_vector` and `zero_vector` class. So, I believe it necessary to collect more information on this and make a brief summary. 
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