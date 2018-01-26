---
layout: post
title: "Fibonacci Heap Implementation"
description: "mFibonacci Heap Implementation"
categories: [C++]
tags: [C++, Algorithm]
redirect_from:
  - /2018/01/24/
---

## 1. Supporting Operations
Fibonacci heap supports following operations(min-heap):

 - `new_heap()` constructor
 - `insert(x)` insert element `x` into heap
 - `union(H)` create one new heap combining `*this` and `H`
 - `root()` return a pointer to min/max pointer on root
 - `extract()` delete min node, and return its pointer
 - `destroy(&Node)` delete element x from heap, note: x needs to be a pointer 
 - `search(x)` search element one by one, if use the `destory(x)` needs to find the element first
 - `degrade_key(&Node, k)` degrade key value, first argument needs to be pointer

 Fibonacci heap has very desirable amortized cost: except `extract()`, `destroy(&Node)` have `O(lgn)` complexity, `search()` has `O(n)` complexity, others all maintain an `O(1)` complexity.

## 2. Implementation
Fibonacci heap has a collection of min-heap ordered trees. 

![Fibonacci1](https://raw.githubusercontent.com/Aperjump/Aperjump.github.io/master/_picture/2018-01-25-Fibonacci_Heap/Fibonacci1.PNG)

As we can see, each node should have a pointer pointing to parent and a pointer to child node. Each child node should link a circle using double linked list. 
```
ttemplate<typename K, typename V>
class Node {
public:
  typedef V value_type;
  typedef K key_type;
  typedef Node<key_type, value_type>* pointer;
  Node() : val(), key(), left_child(NULL), right_child(NULL),
    parent(NULL), child(NULL), degree(1), mark(false) { }
  Node(key_type key, value_type val) : key(key), val(val), left_child(NULL), 
    right_child(NULL),parent(NULL), child(NULL), degree(1), mark(false) { }
  // When copy construct from another Node, I think it inappropriate 
  // to copy Node pointers, if you really need to full copy Node, use 
  // deep_copy method explicitly
  template<typename K, typename V>
  Node(Node<K,V>& node) : val(node.val), key(node.key), left_child(NULL),
    right_child(NULL), parent(NULL), child(NULL),
    degree(1), mark(false) { }
  pointer operator=(pointer other)
  {
    if (other == NULL)
    {
      val = 0;
      key = 0; 
      left_child = NULL;
      right_child = NULL; 
      parent = NULL; 
      child = NULL;
      degree = 1;
      mark = false;
    }
    else {
      copy(*other);
    }
    return this;
  }
  key_type get_key() {
    return key;
  }
  value_type get_val() {
    return val;
  }
  void copy(Node node);
  void deep_copy(Node node);

  value_type val;
  key_type key;
  pointer left_child;
  pointer right_child;
  pointer parent;
  pointer child; // pointer to any of the child
  int degree; // number of child
  bool mark; // whether lost a child
};
```
I tried to separate `key_type` and `val_type`, since I think it a common practice for `key-value` pairs. And I distinguish two copies, one only copies the value, and the other copyies both value and pointer(but seems no use though). 
```
template<typename K, typename V, typename func = min_heap<K> >
class Fibonacci_heap_min : public Fibonacci_heap<K, V, func> {
public:
  Fibonacci_heap_min() : _root(NULL), num(0) { }
  // copy constructor treat the node as root
  Fibonacci_heap_min(Fibonacci_heap<K, V, func>::node_type* node) : _root(node), num(1) { }
  void min() { return root(); }
private:
  Fibonacci_heap<K, V, func>::node_type* _root;
  int num;
};
template<typename K, typename V, typename func = max_heap<K> >
class Fibonacci_heap_max : public Fibonacci_heap<K, V, func> {
public:
  Fibonacci_heap_max() : _root(NULL), num(0) { }
  // copy constructor treat the node as root
  Fibonacci_heap_max(Fibonacci_heap<K, V, func>::node_type* node) : _root(node), num(1) { }
  void max() { return root(); }
private:
  Fibonacci_heap<K, V, func>::node_type* _root;
  int num;
};
```
Next, I created two `Fibonacci_heap` refinements, and their only use is to pass that functor prameters. And two functors are simple:
```
template<typename T>
class min_heap {
public:
  bool operator() (T left, T right) { return left < right; }
};
template<typename T>
class max_heap {
public:
  bool operator() (T left, T right) { return left > right; }
};
```
Finally, this is Fibonacci heap:
```
template<typename K, typename V, typename func>
class Fibonacci_heap {
public:
  typedef V value_type;
  typedef K key_type;
  typedef func functor_type;
  typedef Node<key_type, value_type> node_type;
  Fibonacci_heap() : _root(NULL), num(0), maxdegree(0) { }
  // copy constructor treat the node as root
  Fibonacci_heap(node_type* node) : _root(node), num(1), maxdegree(1) { }
  void insert(node_type* x) {
    if (_root == NULL)
      _root = x;
    else {
      connect(_root, x);
      if (func()(x->get_key(), _root->get_key()))
        _root = x;
    }
    num++;
  }
  // root will return min/max value determined by functor
  node_type* root() { return _root; }
  // combine two root, compare key, add num
  template<typename K, typename V, typename funct>
  int combine(Fibonacci_heap<K, V, funct>* right) {
    connect_circle(_root, right->_root);
      if (func()(right->_root->get_key(), _root->get_key()))
        _root = right->_root;
    num += right->num;
    _root->degree++;
    maxdegree = (_root->degree > maxdegree ? _root->degree: maxdegree);
    return num;
  }
  node_type* extract() {
    node_type* ret_root = (node_type*)malloc(sizeof(node_type));
    ret_root->key = _root->key;
    ret_root->val = _root->val;
    if (_root != NULL) {
      for (node_type* begin = _root->child;
        begin != _root->child; begin = begin->right_child)
        connect(_root, begin);
      remove(_root);
      // Now the node isn't min/max
      constructroot();
      num--;
    }
    return ret_root;
  }
private:

  void connect(node_type* left, node_type* right)
  {
    if (left->left_child == NULL)
    {
      left->left_child = right;
      left->right_child = right;
      right->left_child = left;
      right->right_child = left;
    }
    else {
      left->left_child->right_child = right;
      right->left_child = left->left_child;
      left->left_child = right;
      right->right_child = left;
    }
  }
  void connect_circle(node_type* left, node_type* right)
  {
    if (left == NULL || right == NULL)
    {
      perror("left or right hand side is NULL, combine error\n");
      exit(1);
    }
    else {
      left->left_child->right_child = right->left_child;
      right->left_child->right_child = left->left_child;
      left->left_child = right;
      right->left_child = left;
    }
  }
  void remove(node_type* point)
  {
    point->left_child->right_child = point->right_child;
    point->right_child->left_child = point->left_child;
    if (point->left_child == point->right_child)
      _root = point;
    if (point->left_child == NULL)
      _root = NULL;
    _root = point->right_child;
    point->right_child = NULL;
    point->left_child = NULL;
  }
  void constructroot()
  {
    node_type* tmpheap = (node_type *)malloc(sizeof(node_type) * (maxdegree + 1));
    for (node_type* iter = _root;
      iter != _root; iter = iter->right_child)
    {
      int current_degree = iter->degree;
      while  (&tmpheap[current_degree] != NULL)
      {
        if (func()(iter->key, (&tmpheap[current_degree])->key))
          linknode(iter, &tmpheap[current_degree]);
        else 
          linknode(&tmpheap[current_degree], iter);
        tmpheap[current_degree] = (node_type*)NULL;
        current_degree++;
      }
      tmpheap[iter->degree] = *iter;
    }
    for (int i = 0; i < maxdegree + 1; i++)
    {
      if (func()(_root->get_key(), (&tmpheap[i])->get_key()))
        _root = &tmpheap[i];
    }
  }
  void linknode(node_type* left, node_type* right)
  {
    left->child = right;
    right->parent = left;
    right->left_child = NULL;
    right->right_child = NULL;
  }
  node_type* _root;
  int num;
  int maxheapnum;
  int maxdegree;
};
```
Its main difficulties is the `extract()` function. This picture can briefly summarize its operation:
![Fibonacci2](https://raw.githubusercontent.com/Aperjump/Aperjump.github.io/master/_picture/2018-01-25-Fibonacci_Heap/Fibonacci2.PNG)
![Fibonacci2](https://raw.githubusercontent.com/Aperjump/Aperjump.github.io/master/_picture/2018-01-25-Fibonacci_Heap/Fibonacci3.PNG)
And this is my test code:
```
#include <iostream>
#include <cstdio>
#include <cstdlib>
#include "Fibo.h"
int main() {
  using namespace std;
  Node<int, int> Node_1(3, 4);
  printf("Node1 is (%d, %d)\n", Node_1.get_key(), Node_1.get_val());
  Node<int, int> Node_2{ 2, 7 };
  printf("Node2 is (%d, %d)\n", Node_2.get_key(), Node_2.get_val());
  Node<int, int> Nodetest[] = { {1,3}, {4,5}, {7,8}, {7, 9}, {8, 3} };
  Node<int, int> Nodetest_2[] = { {2,4}, {3,5}, {7,9}, {1,10} };
  printf("Nodelist is (%d, %d)\n", Nodetest[2].get_key(), Nodetest[2].get_val());
  Fibonacci_heap<int,int, min_heap<int> > fibo(&Node_1);
  fibo.insert(&Node_2);
  for (int i = 0; i < 5; i++)
  {
    fibo.insert(&Nodetest[i]);
  }
  printf("root of fibo is (%d,%d)\n", fibo.root()->get_key(), fibo.root()->get_val());
  Fibonacci_heap<int, int, min_heap<int> > fibo2(&Node_1);
  for (int i = 0; i < 4; i++)
  {
    fibo2.insert(&Nodetest_2[i]);
  }
  printf("root of fibo2 is (%d,%d)\n", fibo2.root()->get_key(), fibo2.root()->get_val());
  fibo.combine(&fibo2);
  printf("root of fibo is (%d,%d)\n", fibo.root()->get_key(), fibo.root()->get_val());
  Node<int, int>* new_point = fibo.extract();
  printf("new_point is (%d, %d)\n", new_point->get_key(), new_point->get_val());
  Node<int, int>* new_point2 = fibo.extract();
  printf("new_point is (%d, %d)\n", new_point2->get_key(), new_point2->get_val());
  Fibonacci_heap_min<int, int> a(&Node_1);
  a.insert(&Node_2);
  for (int i = 0; i < 5; i++)
  {
    a.insert(&Nodetest[i]);
  }
  printf("root of a is (%d,%d)\n", a.root()->get_key(), a.root()->get_val());
  Node<int, int>* new_point3 = a.extract();
  printf("new_point3 is (%d, %d)\n", new_point3->get_key(), new_point3->get_val());

}
/*
Node1 is(3, 4)
Node2 is(2, 7)
Nodelist is(7, 8)
root of fibo is(1, 3)
root of fibo2 is(1, 10)
root of fibo is(1, 3)
new_point is(1, 3)
new_point is(2, 7)
root of a is(1, 3)
new_point3 is(1, 3)
*/
```