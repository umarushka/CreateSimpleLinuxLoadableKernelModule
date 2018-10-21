# CreateSimpleLinuxLoadableKernelModule
Create simple Linux loadable kernel module which will be able to run on ARMv7 device
----------

# What is a Kernel Module? #
----------

A loadable kernel module (LKM) is a mechanism for adding code to, or removing code from, the Linux kernel at run time.
They are ideal for device drivers, enabling the kernel to communicate with the hardware without it having to know how
the hardware works. The alternative to LKMs would be to build the code for each and every driver into the Linux kernel.

# Why Write a Kernel Module? #

----------

When interfacing to electronics circuits under embedded Linux you are exposed to sysfs and the use of low-level Úle
operations for interfacing to electronics circuits. This approach can appear to be ineÜcient (especially if you have
experience of traditional embedded systems); however, these Úle entries are memory mapped and the performance is
suÜcient for many applications. I have demonstrated in my book that it is possible to achieve response times of about one
third of a millisecond, with negligible CPU overhead, from within Linux user space by using pthreads, callback functions
and sys/poll.h.# The Module Code #
----------

The run-time life cycle of a typical computer program is reasonably straightforward. A loader allocates memory for the
program, then loads the program and any required shared libraries. Instruction execution begins at some entry point
(typically the main() point in C/C++ programs), statements are executed, exceptions are thrown, dynamic memory is
allocated and deallocated, and the program eventually runs to completion. On program exit, the operating system
identiÚes any memory leaks and frees lost memory to the pool.A kernel module is not an application — for a start there is no main() function! Some of the key diÙerences are that kernel modules:


- do not execute sequentially— a kernel module registers itself to handle requests using its initialization function,
which runs and then terminates. The type of requests that it can handle are deÚned within the module code. This is
quite similar to the event-driven programming model that is commonly utilized in graphical-user interface (GUI)
applications.
- do not have automatic cleanup — any resources that are allocated to the module must be manually released
when the module is unloaded, or they may be unavailable until a system reboots.
- do not have printf() functions — kernel code cannot access libraries of code that is written for the Linux user
space. The kernel module lives and runs in kernel space, which has its own memory address space. The interface
between kernel space and user space is clearly deÚned and controlled. We do however have a printk() function
that can output information, which can be viewed from within user space.
- can be interrupted — one conceptually diÜcult aspect of kernel modules is that they can be used by several
diÙerent programs/processes at the same time. We have to carefully construct our modules so that they have a
consistent and valid behavior when they are interrupted. The BeagleBone has a single-core processor (for the
moment) but we still have to consider the impact of multiple processes accessing the module simultaneously.
- have a higher level of execution privilege — typically, more CPU cycles are allocated to kernel modules than to
user-space programs. This sounds like an advantage, however, you have to be very careful that your module does
not adversely aÙect the overall performance of your system.
- do not have Ûoating-point support — it is kernel code that uses traps to transition from integer to Ûoating-point
mode for your user space applications. However, it is very diÜcult to perform these traps in kernel space. The
alternative is to manually save and restore Ûoating point operations — a task that is best avoided and left to your
user-space code.``#include <linux/init.h>``

``#include <linux/module.h>``

``#include <linux/kernel.h>``

``static int __init helloBBB_init(void){``

``printk(KERN_INFO "EBB: Hello %s from the BBB LKM!\n",name);``

``return 0;``

``}``

``static void __exit helloBBB_exit(void){``

``printk(KERN_INFO "EBB: Goodbye %s from the BBB LKM!\n",name);``

``}``

``module_init(helloBBB_init);``
``module_exit(helloBBB_exit);``