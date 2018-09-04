---
layout: post
title: "How to debug a multi-thread program"
description: "How to debug a multi-thread program"
categories: [Debug]
tags: [lldb,Debug]
redirect from:
  - 2018/09/04/
---

## How to debug a multi-thread program. 
This passage will discuss the usage of `lldb` for multi-threading program debug. You can find my code on my github [code](https://github.com/Aperjump/blog_code/tree/master/How-to-degbug-A-multithreaded-program)
The code implements a simple producer-consumer pattern written in C, and use `pthread` to create new threads. Although you can compile it without any problem, there are four bugs in the code and our mission is to find them all using `lldb`. 
### lldb
For `gdb` users, some commands may different, but there is a command transformation [table](https://lldb.llvm.org/lldb-gdb.html) between `gdb` and `lldb`. 
#### Basic Commands
```
# set breakpoint for xx function
(lldb)breakpoint set -name xx
# set breakpoint for method in class
(lldb)breakpoint set -method xx
# set breakpoint in file F line L
(lldb)breakpoint set -file F -line L
# set breakpoint in function address
(lldb)breakpoint set -a funcaddr
# list all breakpoints
(lldb)breakpoint list
# delete break point
(lldb)breakpoint delete index
------------------------------
# run
(lldb)r
# next
(lldb)n
# step int
(lldb)s
# step out
(lldb)f
# continue
(lldb)c
------------------------------
# watch some variable change when written
(lldb)watchpoint set variable val
------------------------------
# variable in current frame
(lldb)frame variable
(lldb)fr v
# print variable
(lldb)p val
------------------------------
# find all frames
(lldb)thread backtrace all
(lldb)bt all
# current frame
(lldb)thread backtrace
(lldb)bt
# select frame X
(lldb)frame select X
# thread in current frame
(lldb)thread list
# select thread X
(lldb)thread select X
----------------------------
# evaluate a expression E
(lldb)expression E
(lldb)e E
# note expression can be used to dynamically change a value
```
`lldb` share most commands with `gdb`. 
Note : Although `watchpoint` seems like a powerful tool, it's rather limited. On my MacBook Pro(x86_64), there are only 4 hardware watchpoint registers, each at most 8 bytes. So there can be only 48 byte range support. If you read the code, the most important structure is `queue_t`, since it includes one `pthread_mutex_lock`, which is 64 bytes, `queue_t` cannot fit in our watchlist registers. 
### program
(1) Failed to join producer thread: Invalid argument
If we run the program without any modification, we will get this error. So why does this happen? 
Jumping into the code
```
* thread #1, queue = 'com.apple.main-thread', stop reason = step over
    frame #0: 0x000000010000178e producer_consumer`main(argc=1, argv=0x00007ffeefbff548) at producer_consumer.c:72
   69     /* Join threads, handle return values where appropriate */
   70
   71     result = pthread_join(producer_thread, NULL);
-> 72     if (0 != result) {
   73       fprintf(stderr, "Failed to join producer thread: %s\n", strerror(result));
   74       pthread_exit(NULL);
   75     }
Target 0: (producer_consumer) stopped.
(lldb) p result
(int) $3 = 3
```
Here we can see `result=3`, indicates we are trying to join a thread which doesn't exist. If we print `g_num_prod`, it returns 1, meaning that our `producer_routine` has not stopped (only producer_routine will change `g_num_prod` value). 
```
(lldb) bt all
* thread #1, queue = 'com.apple.main-thread', stop reason = step over
  * frame #0: 0x000000010000177d producer_consumer`main(argc=1, argv=0x00007ffeefbff548) at producer_consumer.c:71
    frame #1: 0x00007fff720aa015 libdyld.dylib`start + 1
    frame #2: 0x00007fff720aa015 libdyld.dylib`start + 1
  thread #2
    frame #0: 0x00007fff721f1306 libsystem_kernel.dylib`swtch_pri + 10
    frame #1: 0x00007fff723c322e libsystem_pthread.dylib`sched_yield + 11
    frame #2: 0x0000000100001ae4 producer_consumer`producer_routine(arg=0x00007ffeefbff4c8) at producer_consumer.c:133
    frame #3: 0x00007fff723c2661 libsystem_pthread.dylib`_pthread_body + 340
    frame #4: 0x00007fff723c250d libsystem_pthread.dylib`_pthread_start + 377
    frame #5: 0x00007fff723c1bf9 libsystem_pthread.dylib`thread_start + 13
  thread #3
    frame #0: 0x00007fff721faa46 libsystem_kernel.dylib`__psynch_mutexwait + 10
    frame #1: 0x00007fff723c2b9d libsystem_pthread.dylib`_pthread_mutex_lock_wait + 83
    frame #2: 0x00007fff723c04c8 libsystem_pthread.dylib`_pthread_mutex_lock_slow + 253
    frame #3: 0x00007fff72134cfa libsystem_c.dylib`flockfile + 31
    frame #4: 0x00007fff7213d728 libsystem_c.dylib`vfprintf_l + 28
    frame #5: 0x00007fff7213b7cb libsystem_c.dylib`printf + 187
    frame #6: 0x0000000100001c39 producer_consumer`consumer_routine(arg=0x00007ffeefbff4c8) at producer_consumer.c:172
    frame #7: 0x00007fff723c2661 libsystem_pthread.dylib`_pthread_body + 340
    frame #8: 0x00007fff723c250d libsystem_pthread.dylib`_pthread_start + 377
    frame #9: 0x00007fff723c1bf9 libsystem_pthread.dylib`thread_start + 13
  thread #4
    frame #0: 0x00007fff721fb2ba libsystem_kernel.dylib`__write_nocancel + 10
    frame #1: 0x00007fff7213c796 libsystem_c.dylib`_swrite + 87
    frame #2: 0x00007fff721353bd libsystem_c.dylib`__sflush + 87
    frame #3: 0x00007fff72137aa8 libsystem_c.dylib`__sfvwrite + 816
    frame #4: 0x00007fff72141738 libsystem_c.dylib`__vfprintf + 16158
    frame #5: 0x00007fff72166059 libsystem_c.dylib`__v2printf + 473
    frame #6: 0x00007fff7213d742 libsystem_c.dylib`vfprintf_l + 54
    frame #7: 0x00007fff7213b7cb libsystem_c.dylib`printf + 187
    frame #8: 0x0000000100001b4a producer_consumer`consumer_routine(arg=0x00007ffeefbff4c8) at producer_consumer.c:148
    frame #9: 0x00007fff723c2661 libsystem_pthread.dylib`_pthread_body + 340
    frame #10: 0x00007fff723c250d libsystem_pthread.dylib`_pthread_start + 377
    frame #11: 0x00007fff723c1bf9 libsystem_pthread.dylib`thread_start
```
We can see just before `pthread_join` there are still 4 frames. The reason here is the use of `pthread_detach()` will make a thread cannot join. 

(2) Segment Fault
After the first modification, my program will block at
```
(lldb) n
Process 64207 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = step over
    frame #0: 0x0000000100000846 producer_consumer`main(argc=1, argv=0x00007ffeefbff548) at producer_consumer.c:82
   79       fprintf(stderr, "Failed to join consumer thread: %s\n", strerror(result));
   80       pthread_exit(NULL);
   81     }
-> 82     printf("\nPrinted %lu characters.\n", *(long*)thread_return);
   83     free(thread_return);
   84
   85     pthread_mutex_destroy(&queue.lock);
Target 0: (producer_consumer) stopped.
```
If we evaluate expression `*(long*)thread_return` in `lldb`
```
(lldb) e *(long*)thread_return
error: Couldn't apply expression side effects : Couldn't dematerialize a result variable: couldn't read its memory
```
Meaning that return variable in `customer_routine` has some problem. If we look at source code, the return variable `count` is actually a local variable. We need to change it to heap space. 

(3) Segment Fault (2)
At this time, if you run the program, we will find some time the program runs correctly, but sometimes, it fall back to segment fault. 
```
Process 80389 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = step over
    frame #0: 0x0000000100000778 producer_consumer`main(argc=1, argv=0x00007ffeefbff548) at producer_consumer.c:73
   70     /* Join threads, handle return values where appropriate */
   71
   72     result = pthread_join(producer_thread, NULL);
-> 73     if (0 != result) {
   74       fprintf(stderr, "Failed to join producer thread: %s\n", strerror(result));
   75       pthread_exit(NULL);
   76     }
Target 0: (producer_consumer) stopped.
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = step over
  * frame #0: 0x0000000100000778 producer_consumer`main(argc=1, argv=0x00007ffeefbff548) at producer_consumer.c:73
    frame #1: 0x00007fff720aa015 libdyld.dylib`start + 1
    frame #2: 0x00007fff720aa015 libdyld.dylib`start + 1
(lldb) bt all
* thread #1, queue = 'com.apple.main-thread', stop reason = step over
  * frame #0: 0x0000000100000778 producer_consumer`main(argc=1, argv=0x00007ffeefbff548) at producer_consumer.c:73
    frame #1: 0x00007fff720aa015 libdyld.dylib`start + 1
    frame #2: 0x00007fff720aa015 libdyld.dylib`start + 1
  thread #3
    frame #0: 0x00007fff721faa46 libsystem_kernel.dylib`__psynch_mutexwait + 10
    frame #1: 0x00007fff723c2b9d libsystem_pthread.dylib`_pthread_mutex_lock_wait + 83
    frame #2: 0x00007fff723c04c8 libsystem_pthread.dylib`_pthread_mutex_lock_slow + 253
    frame #3: 0x00007fff72134cfa libsystem_c.dylib`flockfile + 31
    frame #4: 0x00007fff7213d728 libsystem_c.dylib`vfprintf_l + 28
    frame #5: 0x00007fff7213b7cb libsystem_c.dylib`printf + 187
    frame #6: 0x0000000100000c54 producer_consumer`consumer_routine(arg=0x00007ffeefbff4c8) at producer_consumer.c:174
    frame #7: 0x00007fff723c2661 libsystem_pthread.dylib`_pthread_body + 340
    frame #8: 0x00007fff723c250d libsystem_pthread.dylib`_pthread_start + 377
    frame #9: 0x00007fff723c1bf9 libsystem_pthread.dylib`thread_start + 13
  thread #4
    frame #0: 0x00007fff721faa46 libsystem_kernel.dylib`__psynch_mutexwait + 10
    frame #1: 0x00007fff723c2b9d libsystem_pthread.dylib`_pthread_mutex_lock_wait + 83
    frame #2: 0x00007fff723c04c8 libsystem_pthread.dylib`_pthread_mutex_lock_slow + 253
    frame #3: 0x00007fff72134cfa libsystem_c.dylib`flockfile + 31
    frame #4: 0x00007fff7213d728 libsystem_c.dylib`vfprintf_l + 28
    frame #5: 0x00007fff7213b7cb libsystem_c.dylib`printf + 187
    frame #6: 0x0000000100000c54 producer_consumer`consumer_routine(arg=0x00007ffeefbff4c8) at producer_consumer.c:174
    frame #7: 0x00007fff723c2661 libsystem_pthread.dylib`_pthread_body + 340
    frame #8: 0x00007fff723c250d libsystem_pthread.dylib`_pthread_start + 377
    frame #9: 0x00007fff723c1bf9 libsystem_pthread.dylib`thread_start + 13
```
After joining `producer_routine`, we investigate current stack state. It can be seen there are current three threads executing, and we can change to thread 3.
```
(lldb) thread select 3
* thread #3
    frame #0: 0x00007fff721faa46 libsystem_kernel.dylib`__psynch_mutexwait + 10
libsystem_kernel.dylib`__psynch_mutexwait:
->  0x7fff721faa46 <+10>: jae    0x7fff721faa50            ; <+20>
    0x7fff721faa48 <+12>: movq   %rax, %rdi
    0x7fff721faa4b <+15>: jmp    0x7fff721f1ae9            ; cerror_nocancel
    0x7fff721faa50 <+20>: retq
```
These are assembly code, since our problem is linked to `pthread.h`, those program has been compiled. Here we need to jump to our code frame. 
Program will stop on this
```
(lldb) n
Process 90312 stopped
* thread #4, stop reason = EXC_BAD_ACCESS (code=1, address=0x0)
    frame #0: 0x0000000100000c21 producer_consumer`consumer_routine(arg=0x00007ffeefbff4c8) at producer_consumer.c:170
   167        else
   168          queue_p->front->next->prev = NULL;
   169
-> 170        queue_p->front = queue_p->front->next;
   171        pthread_mutex_unlock(&queue_p->lock);
   172
   173        /* Print the character, and increment the character count */
Target 0: (producer_consumer) stopped.
(lldb) e queue_p->front
(queue_node_s *) $0 = 0x0000000100600020
(lldb) e queue_p->front->next
(queue_node_s *) $1 = 0x0000000000000000
```
Apparently, some other thread steals `queue_p->front->next` away, we need to add another lock at the end of each loop. 
(4) Modify Global Variable
The third problem has nothing to do with `lldb`. When modifying global variable in a thread, we need use mutex to protect it, the change is made in `producer_routine`. 
