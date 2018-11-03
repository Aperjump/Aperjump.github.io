---
layout: post
title: "C++ concurrency Notes2"
description: "C++ concurrency Notes2"
categories: [concurrency]
tags: [concurrency]
redirect from:
  - /2018/11/03/
---
## C++ concurrency Notes1

## C++ Concurrency: Synchronizing Operations
`std::this_thread::sleep_for(std::chrono::milliseconds(100));`
The most basic mechanism for waiting for an event to be triggered by another thread is the **condition variable**. When a thread has determined that the condition is satisfied, it can then notify one or more of the thread waiting on the condition variable in order to wake them up and allow them to continue processing. 
`std::condition_variable`, `std::condition_variable_any`
They are declared in `<condition_variable>` header. 
The first one can only work with `mutex`, second can work with multiple kinds of `mutex`
`condition_variable_any` requires more resources, and should not be preferred unless some additional flexibility is required. 
```
std::mutex mut;
std::queue<data_chunk> data_queue;
std::condition_variable data_cond;
void data_preparation_thread()
{
    while(more_data_to_prepare())
    {
        data_chunk const data=prepare_data();
        {
            std::lock_guard<std::mutex> lk(mut);
            data_queue.push(data);
        }
        data_cond.notify_one();
    }
}
void data_processing_thread()
{
    while(true)
    {
        std::unique_lock<std::mutex> lk(mut);
        data_cond.wait(
            lk,[]{return !data_queue.empty();});
        data_chunk data=data_queue.front();
        data_queue.pop();
        lk.unlock();
        process(data);
        if(is_last_chunk(data))
            break;
    }
}
```
the simple `[]{return !data_queue.empty();}` lambda function checks to see if the `data_queue` is not `empty()`— that is, there’s some data in the queue ready for processing.
Here we use `unique_lock` instead of `lock_guard` because we need to unlock the mutex while it's waiting and lock it again afterward. 
When the waiting thread reacquires the mutex and checks the condition, if it isn't in direct response to a notification from another thread, it's called a **spurious wake**. 

### Build a thread-safe queue 
operations:
`empty(), size()`
`push(), pop()`
`front(), back()`
Let’s provide two variants on pop(): `try_pop()`  which tries to pop the value from the queue but always returns immediately (with an indication of failure) even if there wasn’t a value to retrieve; and `wait_and_pop()`, which waits until there’s a value to retrieve.
```
#include <memory>
template<typename T>
class threadsafe_queue
{
public:
    threadsafe_queue();
    threadsafe_queue(const threadsafe_queue&);
    threadsafe_queue& operator=(
        const threadsafe_queue&) = delete;
    void push(T new_value);
    bool try_pop(T& value);
    std::shared_ptr<T> try_pop();
    void wait_and_pop(T& value);
    std::shared_ptr<T> wait_and_pop();
    bool empty() const;
};
```
### Waiting for one-off events with futures
`future` If a thread needs to wait for a specific one-off event, it somehow obtains a **future** representing that event. The thread can periodically wait on the future for short periods of time to see if the event has occurred while performing some other task between checks. 
Two futures in `<future>` library:
unique future: `std::future<>` and shared future: `std::shared_future<>`
An instance of `std::future` is the one and only instance that refers to the same event, whereas multiple instances of `std::shared_future` may refer to the same event(all instances become available at same time)
#### return values from background task
use `std::async`(in `<future>`) to start an asynchronous task for which you don't need the result right away. `std::async` returns a `std::future` object, which will eventually hold the return value of the function. 
```
#include <future>
#include <iostream>
int find_answer_to_universe();
void do_something();
int main() {
	std::future<int> universe_ans = std::async(find_answer_to_universe);
	do_something();
	std::cout << "The answer is " << universe_ans.get() << std::endl;
}
```
`std::async(std::launch::async, Y(), 1.2)`: the function must be run on its own thread. 
`std::async(std::launch::deferred, baz, std::ref(x))` : the function call is to be deferred until either `wait()` or `get()` is called on the future. 
#### Associating a task with a future
`std::packaged_task<>` ties a future to a function or callable object. When the `std::packaged_task<>` object is invoked, it calls the associated function or callable object and makes the `future` ready. The template parameter for the `std::packaged_task<>` class template is a function signature. 
The return type of the specified function signature identifies the type of the `std::future<>` returned from the `get_future()` member function, whereas the argument list of the function signature is used to specify the signature of the packaged task’s function call operator. 
```
template<>
class packaged_task<std::string(std::vector<char>*,int)>
{
public:
    template<typename Callable>
    explicit packaged_task(Callable&& f);
    std::future<std::string> get_future();
    void operator()(std::vector<char>*,int);
};
```
The `std::packaged_task` object is a callable object, and it can be wrapped in a `std::function` object, passed to a `std::thread` as the thread function. 
```
#include <deque>
#include <mutex>
#include <future>
#include <thread>
#include <utility>
std::mutex m;
std::deque<std::packaged_task<void()> > tasks;
bool gui_shutdown_message_received();
void get_and_process_gui_message();
void gui_thread()
{
    while(!gui_shutdown_message_received())
    {
        get_and_process_gui_message();
        std::packaged_task<void()> task;
        {
            std::lock_guard<std::mutex> lk(m);
            if(tasks.empty())
                continue;
            task=std::move(tasks.front());
            tasks.pop_front();
        }
        task();
    }
}
std::thread gui_bg_thread(gui_thread);
template<typename Func>
std::future<void> post_task_for_gui_thread(Func f)
{
    std::packaged_task<void()> task(f);
    std::future<void> res=task.get_future();
    std::lock_guard<std::mutex> lk(m);
    tasks.push_back(std::move(task));
    return res;
}
```
#### `std::promise`
In application with large numbers of network connections, it's common to have a small number of threads handling the connections, with each thread dealing with multiple connections at once. 
`std::promise<T>` provides a means of setting a value that can be later be read through an associated `std::future<T>`. 
Waiting thread can be block on the `future` while the thread providing the data could use the `promise` half of the pairing to set the associated value and make the future ready.

#### handling exception
If the function call invoked as part of `std::async` throws an exception, that exception is stored in the future in place of a stored value, the future becomes ready, and a call to `get()` rethrows that stored exception. 
```
extern std::promise<double> some_promise;
try
{
    some_promise.set_value(calculate_value());
}
catch(...)
{
    some_promise.set_exception(std::current_exception());
}
```
or you can use
```
some_promise.set_exception(std::make_exception_ptr(std::logic_error("foo ")));
```
#### waiting from multiple threads
`std::shared_future` is designed for concurrent code requires that multiple threads can wait for the same event. 
`std::shared_future` is **copyable** and is constructed from `future`
```
std::promise<int> p;
std::future<int> f(p.get_future());
assert(f.valid());
std::shared_future<int> sf(std::move(f));
assert(!f.valid());
assert(sf.valid());
std::promise<std::string> p;
std::shared_future<std::string> sf(p.get_future());
```
The transfer of ownership is implicit for rvalues, so you can construct a `std::shared_future` directly from the return value of the `get_future()` member function of a `promise` object. 

### Time
two kinds of timeout:
(1) duration
(2) real time
The variants that handle the duration-based timeouts have a `_for` suffix, and those that handle the absolute timeouts have an `_until` suffix.
#### Clocks
(1) now:
`std::chrono::system_clock::now()` 
type: `time_point`
(2) steady clock
`std::chrono::steady_clock`
`std::chrono::system_clock`(represents the "real-time" clock of the system and provides functions for converting its time points to and from `time_t` values. )
`std::chrono::high_resolution_clock` provides the smallest possible tick period(the highest possible resolution) of all the library-supplied clocks. 
#### Duration
`std::chrono::duration<>`
The first template parameter is the type of the representation(such as 	`int`, `long`, `double`) and the second is a fraction specifying how many seconds each unit of the duration represents. 
`std::chrono::duration<short, std::ratio<60, 1>>`: number of minutes represented by `short`. 
`nanoseconds, microseconds, milliseconds, seconds, minutes, and hours.` can be found in `std::chrono`
```
using namespace std::chrono_literals;
auto one_day=24h;
auto half_an_hour=30min;
auto max_time_between_messages=30ms;
```
```
std::chrono::milliseconds ms(54802);
std::chrono::seconds s=
    std::chrono::duration_cast<std::chrono::seconds>(ms);
```
The result is truncated rather truncated rather than rounded. 
```
std::future<int> f=std::async(some_task);
if(f.wait_for(std::chrono::milliseconds(35))==std::future_status::ready)
    do_something_with(f.get());
```
`std::future_status::timeout` time out
`std::future_status::ready` ready
`std::future_status::deferred` deferred
The time for a duration-based wait is measured using a steady clock internal to the library. 
#### Time points
`std::chrono::time_point<>`
first template parameter: which clock
second template parameter: unit of measurement. 
The value of a time point is the length of time since 00:00 Jan 1 1970. 
`std::chrono::time_point<std::chrono::system_clock, std::chrono::minutes>`
`std::chrono::high_resolution _clock:: now() + std::chrono::nanoseconds(500)`
```
#include <condition_variable>
#include <mutex>
#include <chrono>
std::condition_variable cv;
bool done;
std::mutex m;
bool wait_loop()
{
    auto const timeout= std::chrono::steady_clock::now()+
        std::chrono::milliseconds(500);
    std::unique_lock<std::mutex> lk(m);
    while(!done)
    {
        if(cv.wait_until(lk,timeout)==std::cv_status::timeout)
            break;
    }
    return done;
}
```
### Synchronization Patterns
#### FP
The term **functional programming** refers to a style of programming where the result of a function call depends solely on the parameter to that function and doesn't depend on any external state. 

#### Actor Model
there are several discrete actors in the system(each running on a separate thread), which send messages to each other to perform the task at hand, and there's no shared state except that which is directly passed via messages. 


