---
layout: post
title: True Parallelism vs Python
date: 2023-11-24 13:59:00
description: Breaking stereotypes about parallelism in python with examples. 
disqus_comments: true
related_posts: false

---


# Introduction

### Goals

The purpose of this text is to find answers to the following questions (yes, these questions are not about the same thing...):
- is parallelism possible in python?
- is true parallelism possible in python?
- is true parallelism possible in python for cpu bound tasks?


At the same time we will get acquainted with GIL and understand why it is so important. We will also learn why it may be useful to avoid it. And of course, we'll learn how to do it (since it's important). At the end, I will talk a bit about funny features of synchronization primitives in Python (I came across them while studying the topic, then I have to share them with someone).


### True Parallelism

Let's begin by defining what *real parallelism* is. I will take it to mean the possibility of executing several program instructions at once at a particular moment of time. This can be done, for example, when there are several processor cores. 

More detailed: in the end, any program is represented as a set of some machine operations, and, obviously, someone has to execute them (it can be a TPU / GPU; but in our case it will be a CPU device, aka processor). In modern systems, a CPU has several cores at once. And I will call true parallelism the case when we achieve a gain in the speed of program execution due to their use.


### Non-true Parallelism

One might ask, then what is non-true parallelism? For example, if we execute 2 programs on one processor core and switch between them so quickly that the user does not notice anything, it can be called parallelism - from the user's point of view 2 programs work simultaneously. But in reality there is only one program running at each moment of time and such parallelism is not true. It. is. an. illusion. But it can be useful. And yet it's completely inappropriate for a scenario where we need to speed up a *single heavy task* (unfortunately, this case isn't rare).


# Threads vs Process

The two main classes of tasks are *cpu and io (input/output) bound* tasks. The first class requires CPU work most of the time, while in the second type of tasks waiting for some third-party services (drive, database, etc.) to respond plays a significant role. 

There is a fundamental difference between io and cpu bound tasks: *io bound tasks can be easily and efficiently parallelized* if you pause execution of one code branch at the moment of waiting for the response of some third-party service and switch to another. In case of a cpu bound task this trick **will not work** - if we give the processor core to someone else, the progress of our execution will not move anywhere and we will **increase** the total running time.

Let's make sure I'm not making anything up.


### Example 1 - io bound task

As an io bound task, we will use a function that ~~just sleeps~~ simulates waiting for an external service to respond.

```python
def io_task() -> 'None':
    for i in range(N_ITER):
        time.sleep(0.00000001)
```

**results (thread):**
```markdown
100%|██████████| 24/24 [01:09<00:00,  2.89s/it]
[# threads = 1, # tasks = 24, workload = io]
Total time 69.4541 s
------------------------------------------------------------
100%|██████████| 24/24 [00:45<00:00,  1.89s/it]
[# threads = 2, # tasks = 24, workload = io]
Total time 45.4068 s
------------------------------------------------------------
```

**results (process):**
```markdown
100%|██████████| 24/24 [01:09<00:00,  2.90s/it]
[# processes = 1, workload = io]
Total time 69.5136 s
------------------------------------------------------------
100%|██████████| 24/24 [00:48<00:00,  2.01s/it]
[# processes = 2, workload = io]
Total time 48.2397 s
------------------------------------------------------------
```

**Conclusion:**
io bound tasks can be accelerated in python by both threads and processes

### Example 2 -- cpu bound task

Here we will use another function, which loads the processor with ~~useless~~ additions of numbers

```python
def cpu_task() -> 'int':
    res: 'int' = 0
    for i in range(N_ITER):
        if not i % 2023:
            res += i    
    return res
```

**results (thread):**
```markdown
100%|██████████| 24/24 [00:00<00:00, 87.65it/s] 
[# threads = 1, # tasks = 24, workload = cpu]
Total time 0.6310 s
------------------------------------------------------------
100%|██████████| 24/24 [00:00<00:00, 139.97it/s]
[# threads = 2, # tasks = 24, workload = cpu]
Total time 0.6166 s
------------------------------------------------------------
```

**results (process):**
```markdown
100%|██████████| 24/24 [00:00<00:00, 35.09it/s]
[# processes = 1, workload = cpu]
Total time 0.7013 s
------------------------------------------------------------
100%|██████████| 24/24 [00:00<00:00, 68.16it/s]
[# processes = 2, workload = cpu]
Total time 0.3638 s
------------------------------------------------------------
```


**Conclusions:**

But cpu bound tasks are not accelerated by creating additional threads (at least in python). So I didn't lie to you.

### Results

Ok, we get it. There are some problems with threads in Python. Then just use processes and be happy. But wait... Let's see in real time how much memory is used to support a python process:

```python
def print_mem():
    total_memory = 0
    for proc in psutil.process_iter(['pid', 'name', 'memory_info']):
        if 'Python' in proc.info['name']:
            total_memory += proc.info['memory_info'].rss
    print(f"Total Python's memory: {total_memory / 1024 / 1024} MB")
```

```markdown
Total Python's memory: 61.703125 MB
```

That's on the order of `20mB` per process (I had 3 process when i called it)! That's a lot, to put it mildly. Maybe it will be possible to make cpu bound task friends with threads?

# Threads & cpu bound task

To really realize what is the reason for such interaction of threads and cpu bound tasks in python, we should dive into the process of python code execution in more detail. It will not be easy. It will be difficult. It will be *uninteresting*. But we will do it. Then it will become obvious what is the bottleneck in the entire system. And to get around that limitation, we'll resort to a *secret technique*...

### `Python`s code execution 

The process of code execution in Python can be broken down into several main steps:

**1. In first, we just write the code, for example, in the `file.py` file:**

```python
print('The real science')
```

**2. Secondly, this code is compiled into some intermediate representation (byte code), which is something between source code and assembly language; in our case we will get a file `file.pyc` with the following contents:**

```markdown
 0 LOAD_GLOBAL              0 (print)
 2 LOAD_CONST               1 ('The real science.')
 4 CALL_FUNCTION            1
 6 POP_TOP
 8 LOAD_CONST               0 (None)
10 RETURN_VALUE
```

**3. After that, Python Virtual Machine is started, which, reading instructions from the `.pyc` file, generates hardware-specific machine code:**

```markdown
11101011101101010110
```

Now we can see that to correctly translate instructions from byte code to machine code, we need a separate program -- *interpreter*. And for each process, this interpreter is *unique*. Therefore, at no point in time can multiple python threads execute instructions on the processor - each of those instructions needs the help of the interpreter, which is one. Moreover, each thread acquires a special *Global Interpreter Lock* to, for example, *simplify memory management*: if we know that only one thread is running at any given moment, it is easy to keep track of the reference count of all objects, and as a consequence - it's very easy for us to understand which objects can be removed.


### Avoiding GIL via `Cython`

Good. Now we can see that the root of the problem is the need to interpret byte code. What if we somehow write code that can be turned into machine code without the need for interpretation? For example, `C++` code does not require this extra step. Indeed, we can use a special `Cython` tool that allows us to write code using `C`-like syntax that can be executed bypassing the interpreter. 

What do we end up with? If we have the ability to write code that won't lock the interpreter (e.g. using `Cython`), then already **multiple python threads can be executed on different cores at the same time**! And this is exactly what we wanted. Let's try to do all this.

/* I did it all honestly and posted the notebook [here](https://github.com/zinchse/parallel_python/tree/main). The visualization of the result is roughly as follows: */

<figure style="display: block; margin: auto; text-align: center;">
    <img src="/assets/img/parallel_python_post/cython_vs_python.png" alt="Cython vs Python" width="500px">
    <figcaption>
        An example of accelerating Python's threads 
        <br>
        by using Cython.
    </figcaption>
</figure>



# Remark

This [repository](https://github.com/zinchse/parallel_python/tree/main) contains code that reproduces all the previously mentioned points. Also there you can see some ~~bugs~~ features of the implementation of locks and semaphores in python, with which you can a) steal money and b) break the synchronization primitives. Good luck!

**Used resources:**
- [python's doc](https://docs.python.org/3/library/concurrency.html)
- [video 1](https://www.youtube.com/watch?v=z7WIm0iZcOU&ab_channel=%D0%94%D0%B8%D0%B4%D0%B6%D0%B8%D1%82%D0%B0%D0%BB%D0%B8%D0%B7%D0%B8%D1%80%D1%83%D0%B9%21)
- [video 2, very hot!](https://www.youtube.com/watch?v=nR8WhdcRJwM&t=3141s&ab_channel=ComputerScienceCenter)
- [video 3](https://www.youtube.com/watch?v=w2-QoVB1TnQ&ab_channel=ComputerScienceCenter)
- [gksfgks](https://www.geeksforgeeks.org/internal-working-of-python/)
