# CreateSimpleLinuxLoadableKernelModule
Create simple Linux loadable kernel module which will be able to run on ARMv7 device
----------

# What is a Kernel Module? #
----------

A loadable kernel module (LKM) is a mechanism for adding code to, or removing code from, the Linux kernel at run time.

They are ideal for device drivers, enabling the kernel to communicate with the hardware without it having to know how the hardware works. The alternative to LKMs would be to build the code for each and every driver into the Linux kernel.

# Why Write a Kernel Module? #

----------

When interfacing to electronics circuits under embedded Linux you are exposed to sysfs and the use of low-level operations for interfacing to electronics circuits. This approach can appear to be ineÜcient (especially if you have experience of traditional embedded systems); however, these Úle entries are memory mapped and the performance is suficient for many applications. I have demonstrated in my book that it is possible to achieve response times of about one third of a millisecond, with negligible CPU overhead, from within Linux user space by using pthreads, callback functions
and sys/poll.h.# The Module Code #
----------

The run-time life cycle of a typical computer program is reasonably straightforward. A loader allocates memory for the program, then loads the program and any required shared libraries. Instruction execution begins at some entry point (typically the main() point in C/C++ programs), statements are executed, exceptions are thrown, dynamic memory is allocated and deallocated, and the program eventually runs to completion. On program exit, the operating system identiÚes any memory leaks and frees lost memory to the pool.A kernel module is not an application — for a start there is no main() function! Some of the key diferences are that kernel modules:

- do not execute sequentially — a kernel module registers itself to handle requests using its initialization function, which runs and then terminates. The type of requests that it can handle are deÚned within the module code. This is quite similar to the event-driven programming model that is commonly utilized in graphical-user interface (GUI) applications.

- do not have automatic cleanup — any resources that are allocated to the module must be manually released
when the module is unloaded, or they may be unavailable until a system reboots.

- do not have printf() functions — kernel code cannot access libraries of code that is written for the Linux user space. The kernel module lives and runs in kernel space, which has its own memory address space. The interface between kernel space and user space is clearly deÚned and controlled. We do however have a printk() function that can output information, which can be viewed from within user space.

- can be interrupted — one conceptually dificult aspect of kernel modules is that they can be used by several
diferent programs/processes at the same time. We have to carefully construct our modules so that they have a
consistent and valid behavior when they are interrupted. The BeagleBone has a single-core processor (for the moment) but we still have to consider the impact of multiple processes accessing the module simultaneously.

- have a higher level of execution privilege — typically, more CPU cycles are allocated to kernel modules than to user-space programs. This sounds like an advantage, however, you have to be very careful that your module does not adversely afect the overall performance of your system.

- do not have floating-point support — it is kernel code that uses traps to transition from integer to Ûoating-point mode for your user space applications. However, it is very diÜcult to perform these traps in kernel space. The alternative is to manually save and restore Ûoating point operations — a task that is best avoided and left to your user-space code.
----------
Listing 1: The Hello World Linux Loadable Kernel Module (LKM) Code`#include <linux/init.h>`

`#include <linux/module.h>`

`#include <linux/kernel.h>`

`MODULE_LICENSE("GPL");`

`MODULE_AUTHOR("Uladzimir Marushka");`

`MODULE_DESCRIPTION("A simple Linux driver for the BBB.");`

`MODULE_VERSION("0.1");`

`static char *name = "world";`

`module_param(name, charp, S_IRUGO);`

`MODULE_PARM_DESC(name, "The name to display in /var/log/kern.log");`


`static int __init helloBBB_init(void){`

`printk(KERN_INFO "EBB: Hello %s from the BBB LKM!\n",name);`

`return 0;`

`}`

`static void __exit helloBBB_exit(void){`

`printk(KERN_INFO "EBB: Goodbye %s from the BBB LKM!\n",name);`

`}`

`module_init(helloBBB_init);`
`module_exit(helloBBB_exit);`
----------
# Prepare the System for Building LKMs #
----------
The system must be prepared to build kernel code, and to do this you must have the Linux headers installed on your device. On a typical Linux desktop machine you can use your package manager to locate the correct package to install.For example, under 64-bit Ubuntu you can use:`~$ sudo apt-get update``~$ apt-cache search linux-headers-$(uname -r)``~$ sudo apt-get install linux-headers-3.16.0-4-amd64``~$ cd /usr/src/linux-headers-3.16.0-4-amd64/``~$ /usr/src/linux-headers-3.16.0-4-amd64$ ls``~$ arch include Makefile Module.symvers scripts`# Building the Module Code #A Make file is required to build the kernel module — in fact, it is a special kbuild Make file. The kbuild Makefile required to build the kernel module in this article can be viewed in Listing 2
----------
Listing 2: The Makefile Required to Build the Hello World LKM`obj-m+=hello.o``all:``make -C /lib/modules/$(shell uname -r)/build/ M=$( PWD) modules``clean:``make -C /lib/modules/$(shell uname -r)/build/ M=$( PWD) clean`# Conclusions #
----------
Hopefully you have built your first loadable kernel module (LKM). Despite the simplicity of the functionality of this module there was a lot of material to cover — by the end of this article: you should have a broad idea of how loadable kernel modules work; you should have your system configured to build, load and unload such modules; and, you should be able to define custom parameters for your LKMs.