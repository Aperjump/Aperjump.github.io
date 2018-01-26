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
template<typename K, typename V>
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
      val(0); key(0); left_child(NULL);
      right_child(NULL); parent(NULL); child(NULL);
    }
    else {
      copy(*other);
    }
    return *this;
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
