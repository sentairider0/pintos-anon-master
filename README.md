HANOI UNIVERSITY OF SCIENCE AND TECHNOLOGY
SCHOOL OF ELECTRICAL & ELECTRONIC ENGINEERING


 

ET4291E – Operating System
ASSIGNMENT 


Instructor :	Assoc. Prof. Pham Van Tien
Course :	ET4291E
Class :	144074
Name :	NGUYEN ANH DUY	20210274





HÀ NỘI – 2023 
Table of Contents
1. Introduction	3
1.1. Pintos	3
1.2. Threads	3
1.3. User Program	3
2. Subproject 1: Advanced scheduler (Threads)	4
2.1. Background	4
2.2. Implementation	4
2.3. Test run result: 16/27, unable to pass Priority Scheduler	6
3. Subproject 2: Argument Passing (User Program)	7
3.1. Background	7
3.2. Implementation	7
3.3. Test run result by echo x y z	9
4. Repository Link: https://github.com/sentairider0/pintos-anon-master	10











 
1. Introduction
1.1. Pintos
Pintos is a simple operating system framework for the 80x86 architecture. It supports kernel threads, loading and running user programs, and a file system, but it implements all of these in a very simple way. 
Important folders of pintos inside pintos/src are:
•	“threads”: source code for the base kernel
•	“userprog”: source code for the user program loader
•	“vm”: an almost empty directory for implementation of virtual memory
•	“filesys”: source code for a basic file system
•	“devices”: source code for I/O device interfacing: keyboard, timer, disk, etc.
•	“lib”: an implementation of a subset of the standard C library. The code in this directory is compiled into both the Pintos kernel and, starting from project 2, user programs that run under it. In both kernel code and user programs, headers in this directory can be included using the #include <...> notation.
•	“tests”: tests for each project
•	“examples”: example user programs for use starting with project 2.
1.2. Threads
	In this assignment, pintos gives a minimally functional thread system. Our job is to extend the functionality of this system to gain a better understanding of synchronization problems.
I will be working primarily in the threads directory for this assignment, with some work in the devices directory on the side. Compilation is done in the threads directory.
Synchronization: Proper synchronization is an important part of the solutions to these problems. Any synchronization problem can be easily solved by turning interrupts off: while interrupts are off, there is no concurrency, so there's no possibility for race conditions. Pintos uses semaphores, locks, and condition variables to solve the bulk of synchronization problems.
Pintos already implements thread creation and thread completion, a simple scheduler to switch between threads, and synchronization primitives (semaphores, locks, condition variables, and optimization barriers).
1.3. User Program
The base code already supports loading and running user programs, but no I/O or interactivity is possible. In this subproject, I will enable programs to interact with the OS.
I will be working out of the userprog directory for this assignment, but I will also be interacting with almost every other part of Pintos
Virtual memory in Pintos is divided into two regions: user virtual memory and kernel virtual memory. A user program can only access its own user virtual memory. An attempt to access kernel virtual memory causes a page fault, handled by page_fault() in userprog/exception.c, and the process will be terminated. Kernel threads can access both kernel virtual memory and, if a user process is running, the user virtual memory of the running process. However, even in the kernel, an attempt to access memory at an unmapped user virtual address will cause a page fault. Every user program will page fault immediately until argument passing is implemented.
2. Subproject 1: Advanced scheduler (Threads)
2.1. Background
According to the description of Pintos manual, the proposal of multi-level feedback scheduling is another alternative to priority donation. Imagine this situation: During the operation of the operating system, different threads have different working characteristics. Some threads have a lot of I/O operations, which require a fast system response time, but they do not need to take up a lot of time. Long CPU times, and the work of some threads may not require a lot of I/O waiting, but this means they require longer CPU time. So in this case, we cannot treat the priorities of all threads equally (we do not consider the priority donation method here), so the Pintos manual gives a method to quantify priority, using the nice value and recent_CPU Two parameters are used to characterize the priority of the thread. Pintos has given the corresponding formula, which is as follows:
1) priority_ = PRI_MAX - (recent_cpu / 4) - (nice * 2)._
recent_cpu means the recent average CPU usage time of the thread, which can be iterated. The description of recent_cpu is as follows:
2) recent_cpu_ = (2*load_avg)/(2*load_avg + 1) * recent_cpu __ + __ nice
load_avg is often called the system load average and is used to estimate the average running time of a thread in the past time. It can be iterated, and the iteration formula is as follows:
3) load_avg_ = (59/60)*load_avg + (1/60)*ready_threads
From the above formula, it can be seen that the more recent_cpu iterations, the value is Getting bigger and bigger. Therefore, it can be seen from the first formula that its priority will be lower and lower, which is also consistent with the content of the class.
In addition, Pintos has provided the header file fixed_point.h, because Pintos does not support floating point operations.

2.2. Implementation
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


2.3. Test run result: 16/27, unable to pass Priority Scheduler
 
3. Subproject 2: Argument Passing (User Program)
3.1. Background
There is already a function process_execute in the process.c file of the Pintos code. This function is used to create a new user-level process. But currently Pintos does not support parameter instructions, so I need to implement parameter transfer. Similar to process_execute("ls -ahl") when two parameters are passed in, the markers used in the program are argc and argv.
Another thing to note is that because parameter passing in Pintos has not been implemented yet, and the test cases provided by Pintos have their own names, currently executing the test Pintos will directly crash.
So the current purpose is to realize parameter passing during process execution.
3.2. Implementation	
•	Implement parse_filename function to extract the first argument from source string to destination string.
•	In process_execute function, parse file_name using parse_filename function and forward the first token as name of new process to thread_create function.
•	In start_process function, parse file_name and save the tokens on user stack of new process by calling construct_esp function
•	Implement construct_esp function to set up an user stack for processing the arguments
3.3. Test run result by echo x y z
Invoke make from userprog folder to create build directory 
Invoke make from examples folder to create executable file of echo.c
Navigate to the build directory of userprog, then run these commands to create a disk with a file system partition, format the file system, copy the echo program into the new disk:
		pintos-mkdisk filesys.dsk --filesys-size=2
		pintos -f -q
		pintos -p ../../examples/echo -a echo -- -q
Run pintos -q run 'echo x y z' to execute the code inside echo.c
 
4. Repository Link: https://github.com/sentairider0/pintos-anon-master 
