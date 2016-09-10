---
layout: post
title: A Mini Scheduler
---

A while ago I was asked the following question: design a miniature executor service, or ```Scheduler```, that can 
asynchronously schedule ```Runnable``` tasks to run after a certain amount of time. I thought
this was an interesting programming exercise, so I decided to take some time and implement
it in my free time. In this post I'll break down my solution and detail my voluntary design
decisions. 

# Scheduler API

The ```Scheduler``` API is simple and consists of the following three functions:

1. ```void Start(void)```: Start the service asynchronously in the background. This could have been a synchronous
function that used the calling thread for the actual scheduling, but the goal is to create a single 
scheduler that can manage multiple tasks 

2. ```void Stop(void)```: Stop the scheduling service and allow any pending tasks to complete. Alternatively,
this could have killed every task in the queue. However, I consider a submitted task as one that must
be run, regardless of its place in the queue. 

3. ```void runAfter(long msec, Runnable task)```: Run the given task after the specified amount of milliseconds. 
The time is not a wallclock value. It's the offset from the current time (i.e., System.currentTimeMillis())
into the future. So calling this with a value of 5L is equivalent to requesting the task to be run
5ms in the future. 

The constructor for the scheduler is empty (for reasons I'll explain later) so I omitted it from this list. 
And that's it. No fluff. Just a simple scheduler. 

# Main Tasks

There are three main problems the scheduler must solve: (1) managing the queue of tasks to be run and
(2) scheduling the order of tasks to run, and (3) performing the task execution. I'll explain my solution to both 
of these problems separately below.

## (1) Task Management

At a given time T, the scheduler needs to know whether it should execute a task (or two) at that time. 
The easiest way to do this is to keep track of the absolute time T' when a given task should be run and 
then compare T' to T. If the scheduler keeps the tasks in sorted order, then its less work on the scheduler 
to identify the next task(s) to be run at time T. A naive solution might then be to use a sorted list to
store the tasks. But that leads to linear time insertion, which is something we want to avoid if we're submitting
a lot of tasks. Another approach would be to use a binary search tree with nodes equal to the absolute schedule
time of the tasks. However, if tasks are inserted in increasing order, the tree ends up skewed and we return
to the linear time lookup and insertion. 

The best approach is to use a minimum heap: this gives us O(1) complexity to find the minimum task in the
pool, and logarithmic time to manage the structure after tasks are deleted and as they are added. In my solution,
I don't actually use a heap, though. I use a priority queue, which is implemented as a heap under the hood.
Priority is given to tasks with the smallest absolute schedule time. I use Java's ```PriorityQueue``` implementation
specialized with my own ```Task``` class, which encapsulates the schedule time and the runnable task, 
that implements the ```Comparable``` interface. This allows the priority queue to maintain the proper order
of the tasks. The ```Task``` class is shown below.

{% gist 44b5be70c98ad4d4ba7fc45a8c911a19 %}

## (2) Scheduling

This is the most difficult part about the scheduler. At a high level, we want the scheduler to only be active
when there is a task to run. That means we don't want to use polling or a busy wait to check when the next
available task(s) are ready to run. So what's the solution? We make use of locks and condition variables. 
When the scheduler is woken up to do something it must first acquire a lock on the task queue. If 
the task queue is empty, it silently goes to sleep and releases its lock using the condition variable. 
This is shown below.

{% gist 1eabb3f6befb679d5161f8c10027a640 %}

Once the scheduler wakes up and the queue is not empty, it then does the following. First, it peeks
at the minimum task to see if it can be run. This is done by comparing the task's time T' to current
system time T. If T' < T, then the scheduler requests the associated runnable to be executed. Otherwise,
the scheduler goes to sleep for (T' - T) milliseconds. This is the maximum amount of time that can pass
until the scheduler is ready to run again. However, there's a catch: what if scheduler client inserts a 
new task whose schedule time T* is less than T'? If the scheduler is sleeping until T', it would not run
the new task at the earliest possible time. What we want is something that lets the scheduler sleep for
T' - T milliseconds, but to also be woken up if necessary to run any new tasks. To do this, we rely on
the condition variable's ```awaitNanos``` function. This will put the thread to sleep until the condition
is signalled or the specified time passes. Thus, we only need to signal the condition variable when
inserting new tasks into the queue to get the desired behavior. 

The entire ```run()``` function is shown below, along with the ```runAfter``` code that adds tasks to
the queue. The code should be fairly easy to read. 

{% gist 4faf4557b60e6d6b1de84eb5513d7f70 %}

## (3) Executing Tasks

This one's easy: we want the scheduler to maximize the amount of available threads that can be run
concurrently on a single system. And we also don't want the scheduler to deal with dispatching code
to the CPU (rightfully so in Java, too). So we just use a bounded ThreadPool whose size is equal to
the number of available cores on the current machine. The scheduler then submits tasks to the pool
to be executed and the pool runs them as soon as a thread is available. Simple enough.

## Start and Stop

Starting the scheduler is also pretty easy. Since we want it to run asynchronously, we create and start
a ```Thread``` that we can later shut down when the time comes. To control the execution of the thread,
we use an atomic boolean as a "killswitch." Basically, the scheduler thread runs while the killswitch
is false. When the ```Stop``` function is invoked by the client, the killswitch is flipped, the scheduler
is awoken, and it stops service. However, before doing so, it drains its queue into the pool and then asks
the pool to promptly shut down. I believe the ThreadPool behavior is to run its pending tasks to completion
and then gracefully exit. There is a forceful shutdown function in the pool API that I could have used, but
it didn't seem necessary.

# Putting it Together

So that's the majority of the scheduler. To test it out I created a simple "sleeper task" whose job is to,
well, sleep for a given amount of time. These are the runnables that are submitted to the scheduler to be executed.
The code is shown below.

{% gist bfc8a5b7df3436a8c1e61236bb2aa708 %}

I then created a driver for the scheduler that does the following:

1. Create the scheduler.
2. Create some sleeper tasks. (The number and sleep time of which are specified via the command line.)
3. Start the scheduler.
4. For each task, generate a random future runtime and submit it via the ```runAfter``` function.
5. Stop the scheduler and let the sleepers run to completion.

That's it. Nothing fancy. You can check out the driver code below and the complete code for the scheduler
[in this repo](https://github.com/chris-wood/scheduler).

{% gist c98002d063a98748d33f63408fde2622 %} 
