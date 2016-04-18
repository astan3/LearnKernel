## 1. Introduction
### 1.1 About Linux
Linux is a Unix-like operating systems.  
Some important characteristics of the linux kernel are:  
- Is a monolitic kernel  
  So, the entire operating system is running in kernel space, in privileged mode.  
  By contrast, in a microkernel contains the minimum amount of software needed to implement in operating system (such as memory management, process management, IPC, etc).  
  The microkernel runs in privileged mode. Files systems, drivers, networking stack are tipically removed from the microkernel and run in user space.  
- Supports modules  
  Linux's support for modules is very good, being able to automactically load and unload modules on demand.  
- Have kernel threads  
  Linux have support for kernel threads.  
  It uses them, for example, to execute some kernel functions periodically, suck as flushing disk caches, swapping out unused memory pages, etc.  
  Kernel threads are executing exclusively in kernel space.  
- Provides multithreading support for applications.  
  There is no strong distinction between user threads and processes in the kernel.  
  Both are represented by the same abstraction (*task_struct*) and are scheduled similary by the scheduler.  
- Is a preemptive kernel  
  This means the execution paths in kernel mode can be interleaved at the scheduler decision.  
- Supports SMP (symmetric multi-processing) mode  
  In that case the kernel can use multiple processors and each processor can handle any kernel task.  
  All processors are viewed in the same way, the kernel makes no distinction between them.  
- Supports a wide range of hardware architectures, see [here](https://github.com/torvalds/linux/tree/master/arch)  

### 1.2 Basic operating system concepts
The main purpose of an operating system is to provide an environment in which the use application can run.  
The operating system abstract the hardware resources and hide some low level details regarding computer hardware organization from the user applications.  
In order for the operating system to provide such and abstraction, it must interact directly with the hardware.  
The kernel is the lowest level part of an operating systems. We can see the kernel as the heart of the operating system.  
When an application needs to use a hardware resource, it must issue a request to the kernel, which will interact with that resource in the application behalf.  
Modern operating systems also provide a level of _protection_ to the application programs. For example, an application cannot access directly the physical memory or the memory reserved for the operating system itself.  
In order to achieve this, modern hardware provides at least two different _execution modes_ for the CPU: a non-privileged mode for user programs (called _User Mode_) and a privileged mode for the operating sytem kernel (called _Kernel Mode_).  

#### 1.2.1 Processes
A fundamental abstraction of an operating system is the _process_. A process is the execution context of a running program.  
Each process has its own _address space_: the set of addresses that the process can reference.  
Several processes can be active concurrently and contend for the system resources (such as CPU).  
On _uniprocessor_ system, only one process can use the CPU at a certain moment, so only one execution flow can progress at a time.  
But many modern hardware architectures are multiprocessor architectures and, in that case, the kernel must use the hardware efficiently and allow multiple processes executing at the same time (a maximum number equal to the processor cores).  
An operating system component called the _scheduler_ chooses which process gets executed at a certain time.  
Linux is a __preemptable__ operating systems, meaning that the operating systems tracks how long a process can hold a CPU and periodically activates the scheduler which decides when the process needs to relinquish the CPU.  
A process does not need to know about the existance of another process. From the process point a view, it is like is the only process on the machine and it has exclusive access to the resources abstracted by the operating system.  
Of course, this is only an illusion, but it has the big merrit of greatly simplifying programming of the user applications.  

We have seen that, when using Linux, a CPU can run in either Kernel Mode (privileged mode) or User Mode (non-privileged mode).  
Some CPUs (such a x86 CPUs) can have more than two execution modes. For example, x86 CPUs have four execution modes call rings (ring 0 to ring 3).  
But, even in this case, Linux uses only two execution modes: Kernel Mode and User Mode.  
We have also seen that each time a process needs to have access to the kernel services and needs to make a request to the kernel.  
This happens by using _system calls_.  
Each system call sets up some parameters to identify the request and execute a hardware dependent CPU instruction to switch the CPU execution mode from User Mode to Kernel Mode.  
Similary, a CPU instruction is used to return from the Kernel Space to User Space.   
So, during the lifetime of an user process, it can execute both in user space and in kernel space.  
For example, here are some examples in which the execution can switch from User Mode to Kernel Mode:  
- The process issues a system call (for example because it wants to read from a file). This effectively switches the execution from User Mode to Kernel Mode.  
  The system call will be handled, in Kernel Mode, by a  _system call handler_. The system call handler executes the system call than returns to User Space.  
- A timer interrupt activates the kernel scheduler. The scheduler code executes in Kernel Space.  
- A hardware device _raises an interrupt_ (for example because in I/O operation has finished). The execution switches to Kernel Mode and the _interrupt handler_ gets executed.  
- The CPU executing the process instructions _signals an exception_ (for example, because an invalid instruction was attempted to be executed).  
  The execution switches to Kernel Mode and the exception handler is called.  
- A kernel thread is executed. Kernel threads are executed exclusively in Kernel Space.  

Here is an example showing how processes switches between User Space and Kernel Space:  

![User Space - Kernel Space transitions](img/User_Space_Kernel_Space_Transitions.png)

In this example:
- Process A switches execution from User Space to Kernel Space via a system call.  
- The system call gets executed in Kernel Space, than the execution switches back to User Space.  
- At some point, the scheduler gets activated via a timer interrupt, the execution switching again to Kernel Mode.  
- The scheduler decides to stop the execution of process A and to start the execution of process B.  
- The process B continues its execution in User Space until a hardware device raises an interrupt.
  At that point, the execution switches to Kernel Mode in order for the interrupt handler to be executed.  

As we have seen, the scheduler might decide that is time to stop the execution of a process and to start or continue the execution of another one.  


