//NGUYEN ANH DUY 20210274 SUBSPROJECT OS
---------------------------------------
REPOSITORY LINK: https://github.com/sentairider0/pintos-anon-master
---------------------------------------
// PROJECT 1 ADVANCED SCHEDULER: FAIL 20/27 //

I've tried a number of algorithms and newly added code lines for my files (synch.c, thread.c, thread.h) but unfortunately, the results return all the same. I've only passed tests/threads/alarm-single; tests/threads/alarm-multiple; tests/threads/alarm-simultaneous; tests/threads/alarm-zero; 
tests/threads/alarm-negative; tests/threads/mlfqs-fair-2; and tests/threads/mlfqs-fair-20. Thus, failed to complete Advanced Scheduler.

Here's the best code explanation I've found that I kind of understand. Their way of implementing Advanced Scheduler is marked @1-3 in the files. Source code: https://github.com/ChenyfWula/Pintos/tree/main

* synch.c *

/*line 238-243: to modify mlfqs-lock_release*/
  if(thread_mlfqs == true){
    lock->holder = NULL;
    sema_up (&lock->semaphore);
    return;
  }

* thread.c *

/*line 64-66: to modify global load_avg
LOAD_AVG which is a fixed_point number which we use fx_p to reperesent in PINTOS*/
fx_p load_avg;

/*line 119-121: to modify init load_avg
LOAD_AVG is a FP with initial value 0*/
load_avg = CONVERT2FP(0);

/*line 345-351: to modify mlfqs-thread_set_priority*/
  if (thread_mlfqs == true)
  /* thread_set_priority not called in mlfqs */
    thread_current ()->priority = new_priority;
    thread_yield();
    return;
}

/*line 368-373: to modify thread_get_nice*/
/* Returns the current thread's nice value. */
int thread_get_nice(void)
{
  return thread_current()->nice;
}

/*line 375-380: to modify thread_get_load_avg*/
/*Returns 100 times the current system load average, rounded to the nearest integer.*/
int thread_get_load_avg(void)
{
  return CONVERT2INT_ROUND(FP_MUL_INT(load_avg, 100));
}

/*line 382-387: to modify thread_get_recent_cpu
Returns 100 times the current thread's recent_cpu value, rounded to the nearest integer.*/
int thread_get_recent_cpu(void)
{
  return CONVERT2INT_ROUND(FP_MUL_INT(thread_current()->recent_cpu, 100));
}

/*line 481-492: to modify init*/
  t->nice = 0;
  t->recent_cpu = CONVERT2FP(0);
  list_init(&t->locks_holded);

  old_level = intr_disable(); 

  /* no need to order all_list, I think.  */
  list_push_back(&all_list, &t->allelem);

  intr_set_level(old_level); 
}

/*line 607-684: additional part, I don't know how to explain*/

* thread.h *

/*line 105-107: to modify nice in struct thread*/
    int nice;
    fx_p recent_cpu;

------------------------------

//* PROJECT 2 ARGUMENT PASSING: TEST FAILED MISERABLY*//

Unfortunately, I don't understand how to test or compile the complete source code for project User Program, as the video tutorial 4 you gave us, I believe it only works for Threads project. Still, I tried to test make check with kernel.bin, kernel.o and loader.bin from inagural folders to userprog but, it couldn't run in the first place.

The source code I uploaded is original. From the same authors as in project 1. Their changes are marked by @2-2, @2-3 and @2-4

------------------------------

//*ALGORITHMS IN THE PROJECTS*// Source code: https://github.com/Zhouyuan-Chen/Tutorial-for-OS-Pintos-CS162
The authors explained in Chinese so I took time to translate the script back to English.

* ADVANCED SCHDULER *
According to the description of Pintos manual, the proposal of multi-level feedback scheduling is another alternative to priority donation. Imagine this situation: During the operation of the operating system, different threads have different working characteristics. Some threads have a lot of I/O operations, which require a fast system response time, but they do not need to take up a lot of time. Long CPU times, and the work of some threads may not require a lot of I/O waiting, but this means they require longer CPU time. So in this case, we cannot treat the priorities of all threads equally (we do not consider the priority donation method here), so the Pintos manual gives a method to quantify priority, using the nice value and recent_CPU Two parameters are used to characterize the priority of the thread. Pintos has given the corresponding formula, which is as follows:
1) priority_ = PRI_MAX - (recent_cpu / 4) - (nice * 2)._

recent_cpu means the recent average CPU usage time of the thread, which can be iterated. The description of recent_cpu is as follows:
2) recent_cpu_ = (2*load_avg)/(2*load_avg + 1) * recent_cpu __ + __ nice

load_avg is often called the system load average and is used to estimate the average running time of a thread in the past time. It can be iterated, and the iteration formula is as follows:
3) load_avg_ = (59/60)*load_avg + (1/60)*ready_threads

From the above formula, it can be seen that the more recent_cpu iterations, the value is Getting bigger and bigger. Therefore, it can be seen from the first formula that its priority will be lower and lower, which is also consistent with the content of the class.

In addition, Pintos has provided the header file fixed_point.h, because Pintos does not support floating point operations.

* ARGUMENT PASSING *
There is already a function process_execute in the process.c file of the Pintos code. This function is used to create a new user-level process. But currently Pintos does not support parameter instructions, so I need to implement parameter transfer. Similar to process_execute("ls -ahl") when two parameters are passed in, the markers used in the program are argc and argv.

Another thing to note is that because parameter passing in Pintos has not been implemented yet, and the test cases provided by Pintos have their own names, currently executing the test Pintos will directly crash.

So the current purpose is to realize parameter passing during process execution.

//
