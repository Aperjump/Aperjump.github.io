---
layout: post
title: "C++ concurrency Notes1"
description: "C++ concurrency Notes1"
categories: [concurrency]
tags: [concurrency]
redirect from:
  - /2018/11/02/
---
## C++ concurrency Notes1

### thread launching
`std::thread` works with any callable type.
```
class back_ground_task
{
public:
	void operator()() const {
		do_something();
	}
}
back_ground_task f;
std::thread my_thread(f);
```
```
std::thread my_thread((background_task()));
std::thread my_thread{background_task()};
```
**the most vexing parse**
>  anything that could be considered as a function declaration, the compiler should parse it as a function declaration.

```
struct B
{
    explicit B(int x){}
};
 
struct A
{
    A (B const& b){}
    void doSomething(){}
};
 
int main()
{    
    A a(B(x));
    
    a.doSomething();
}
```
is interpreted as a function declaration: it would be a function called a, that takes by value a parameter of type B called x and that returns an object of type A by value.

If you don't wait for your thread to finish, then you need to ensure that the data accessed by the thread is valid until the thread has finished with it. 

### argument passing to a thread function
**Bear in mind, by default, the arguments are copied into internal storage, where they can be accessed by the newly created thread of execution, and then passed to the callable object or function as rvalues as if they were temporaries**
This is done even if the corresponding parameter in the function is expecting a reference. 
`std::thread` constructor copies the supplied values as is, without converting to the expected argument type. 
If you really want to pass a reference, use `std::ref`
```
class X 
{
public:
	void do_lengthy_work();
};
X my_x;
std::thread t(&X::do_lengthy_work, &my_x);
```

### choose number of threads
`std::thread::hardware_concurrency()`
this function returns `0` if the information is not available. 
It's worth noting that because you can't return a value directly from a thread, you must pass in a reference to the relevant entry in the `result` vector. 
```
template<typename Iterator,typename T>
struct accumulate_block
{
    void operator()(Iterator first,Iterator last,T& result)
    {
        result=std::accumulate(first,last,result);
    }
};
template<typename Iterator,typename T>
T parallel_accumulate(Iterator first,Iterator last,T init)
{
    unsigned long const length=std::distance(first,last);
    if(!length)
        return init;
    unsigned long const min_per_thread=25;
    unsigned long const max_threads=
        (length+min_per_thread-1)/min_per_thread;
    unsigned long const hardware_threads=
        std::thread::hardware_concurrency();
    unsigned long const num_threads=
        std::min(hardware_threads!=0?hardware_threads:2,max_threads);
    unsigned long const block_size=length/num_threads;
    std::vector<T> results(num_threads);
    std::vector<std::thread>  threads(num_threads-1);
    Iterator block_start=first;
    for(unsigned long i=0;i<(num_threads-1);++i)
    {
        Iterator block_end=block_start;
        std::advance(block_end,block_size);
        threads[i]=std::thread(
            accumulate_block<Iterator,T>(),
            block_start,block_end,std::ref(results[i]));
        block_start=block_end;
    }
    accumulate_block<Iterator,T>()(
        block_start,last,results[num_threads-1]);
    std::for_each(threads.begin(),threads.end(),
        std::mem_fn(&std::thread::join));             @10
    return std::accumulate(results.begin(),results.end(),init);
}
```
### thread id
`std::thread::id`
two ways to get this id:
(1) use `thread.get_id()`
(2) in a thread use `std::this_thread::get_id()`
The comparison operators provide a total order for all non-equal values of `std::thread::id`. 
thread IDs could be used as keys into associative containers where specific data needs to be associated with a thread and alternative mechanisms such as thread-local storage aren’t appropriate.


## Sharing Data
### problem
#### race condition
a race condition is anything where the outcome depends on the relative ordering of execution of operations on two or more threads. 
Race conditions are generally timing sensitive, they can often disappear entirely when the application is run under debugger, because the debugger affects the timing of the program. 

several solutions:
(1) wrap data with a protection mechanism
(2) lock-free programming
(3) transaction -> **software transactional memory**

### `mutex`
Before accessing a shared data structure, you lock the mutex associated with that data, and when you’ve finished accessing the data structure, you unlock the mutex.
`#include <mutex>`
`std::mutex some_mutex`
`std::lock_guard<std::mutex> guard(some_mutex)`
Under C++17, using **class template argument deduction** we can write it as
`std::lock_guard guard(some_mutex)`
**Don't pass pointers and references to protected data outside the scope of the lock, whether by returning them from a function, storing them in externally visible memory, or passing them as arguments to user-supplied functions. **
### deadlock
The common advice for avoiding deadlock is to always lock the two mutexes in the same order. 
C++ has `std::lock` a function that can lock two or more mutexes at once without risk of deadlock. 
```
void swap(X& lhs, X& rhs) {
	if (&lhs == &rhs)
		return;
	std::lock(lhs.m, rhs.m);
	std::lock_guard<std::mutex> lock_a(lhs.m, std::adopt_lock);
	std::lock_guard<std::mutex> lock_b(rhs.m, std::adopt_lock);
	do_something();
}
```
The `std::adopt_lock` parameter is supplied in adddition to the mutex to indicate to the `std::lock_guard` objects that mutexes are already locked, and they should adopt the ownership the existing lock on the mutex rather than attempt to lock the mutex in the constructor. 
If there is a failure when locking one of the lock, another lock release immediately. (**lock or nothing**)
C++ 17 has `std::scoped_lock<>` -> **variadic template**
same as `std::lock_guard<>`
### `unique_lock`
`unique_lock` has more flexibility than `lock_guard`
an `unique_lock` doesn't always own the mutex that it's associated with. 
(1) you can pass `std::adopt_lock` as a second argument to the constructor to have the lock object manage the lock on a mutex
(2) you can also pass `std::defer_lock` as second argument to indicate that the mutex should remain unlocked on construction. 
The lock can then be acquired later by calling `lock()` on the `std::unique_lock` object. 
```
class some_big_object;
void swap(some_big_object& lhs,some_big_object& rhs);
class X
{
private:
    some_big_object some_detail;
    std::mutex m;
public:
    X(some_big_object const& sd):some_detail(sd){}
    friend void swap(X& lhs, X& rhs)
    {
        if(&lhs==&rhs)                               
            return;
        std::unique_lock<std::mutex> lock_a(lhs.m,std::defer_lock);
        std::unique_lock<std::mutex> lock_b(rhs.m,std::defer_lock);
        std::lock(lock_a,lock_b);
        swap(lhs.some_detail,rhs.some_detail);
    }
};
```
### transfer mutex onwership
since `unique_lock` don't have to own mutexes, the ownership of a mutex can be transferred between instances by moving the instances around. 
```
std::unique_lock<std::mutex> get_lock()
{
    extern std::mutex some_mutex;
    std::unique_lock<std::mutex> lk(some_mutex);
    prepare_data();
    return lk;
}
void process_data()
{
    std::unique_lock<std::mutex> lk(get_lock());
    do_something();
}
```
### protecting shared data during initialization
C++ standard provides `std::once_flag` and `std::call_once`. Rather than locking a mutex and explicitly checking the pointer, every thread can use `std::call_once`, safe in the knowledge that the pointer will have been initialized by some thread by the time `std::call_once` returns. 
```
std::shared_ptr<some_resource> resource_ptr;
std::once_flag resource_flag;
void init_resource()
{
    resource_ptr.reset(new some_resource);   
}
void foo()
{
    std::call_once(resource_flag,init_resource);
    resource_ptr->do_something();
}
```
### protecting rarely updated data structures
There is a new mutex called `shared_mutex` and `shared_timed_mutex` -> read-writer mutex, allows for two different kind of usage: exclusive access by a single writer thread or shared concurrent access by multiple reader thread. 
```
#include <map>
#include <string>
#include <mutex>
#include <shared_mutex>
class dns_entry;
class dns_cache
{
    std::map<std::string,dns_entry> entries;
    mutable std::shared_mutex entry_mutex;
public:
    dns_entry find_entry(std::string const& domain) const
    {
        std::shared_lock<std::shared_mutex> lk(entry_mutex);
        std::map<std::string,dns_entry>::const_iterator const it=
            entries.find(domain);
        return (it==entries.end())?dns_entry():it->second;
    }
    void update_or_add_entry(std::string const& domain,
                             dns_entry const& dns_details)
    {
        std::lock_guard<std::shared_mutex> lk(entry_mutex);
        entries[domain]=dns_details;
    }
};
```
### recursive lock
`std::recursive_mutex`you can acquire multiple locks on a single instance from the same thread. You must release all your locks before the mutex can be locked by another thread, so if you call `lock()` three times, you must also call `unlock()` three times. 



