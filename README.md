# 4190.307 Operating Systems (Spring 2020)
# Project #2: System calls
### Due: 11:59PM (Wednesday), April 1

## Introduction

System call is a mechanism that allows user applications to ask a variety of services to the operating system kernel. The goal of this project is to understand how those system calls are implemented in ``xv6-riscv``.

## Problem specification

In Unix-like operating systems, each process has a process group ID (or ``PGID``). A process group denotes a collection of one or more processes. Usually, a process group is used to control the distribution of a signal; when a signal is directed to a process group, the signal is delivered to all the processes belonging to the process group.

Your task is to implement ``setpgid()`` and ``getpgid()`` systems calls in ``xv6`` that are used to set or get the process group ID of the given process. Detailed project requirements are described below.

### 1. setpgid() system call

__SYNOPSIS__
```
    int setpgid(int pid, int pgid);
```

__DESCRIPTION__

``setpgid()`` sets the PGID of the process specified by ``pid`` to ``pgid``. 

* If ``pid`` is zero, then the process ID of the calling process is used. 
* If ``pgid`` is zero, then the PGID of the process specified by ``pid`` is made the same as its process ID. 
* ``pid`` and ``pgid`` should be non-negative integers. 
* During ``fork()``, the PGID of the child process is inherited from that of the parent process by default.
* The PGID of the ``init`` process (PID 1) is set to 1.

__RETURN VALUE__

* On success, ``setpgid()`` returns zero. 
* On error (e.g., process not found, invalid value of ``pid`` or ``pgid``, etc.), -1 is returned.

### 2. getpgid() system call

__SYNOPSIS__
```
    int getpgid(int pid);
```

__DESCRIPTION__

``getpgid()`` returns the PGID of the process specified by ``pid``.

* If ``pid`` is zero, the process ID of the calling process is used.
* ``pid`` should be a non-negative integer.


__RETURN VALUE__

* On success, ``getpgid()`` returns zero. 
* On error (e.g., process not found, invalid value of ``pid``, etc.), -1 is returned.


### 3. Displaying PGID in process list

If you press ``ctrl-p`` (or ``^p``) in the shell prompt, ``xv6`` prints the list of processes as shown below. The first number represents the PID of the process, the next one its state, and the final column shows the name of the process. 

```
$ make qemu
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 3G -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$ ^p
1 sleep  init
2 sleep  sh
```

You need to modify the relevant ``xv6`` source code so that ``xv6`` also prints the PGID of each process next to the PID as shown below.

```
$ make qemu
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 3G -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$ ^p
1 1 sleep  init
2 1 sleep  sh
```

In the above example, you can see that the PGID (second column) of the ``sh`` process is also set to 1 as its PGID is inherited from its parent, i.e., the ``init`` process. 



## Skeleton code

We have made some changes to the original ``xv6`` source code and created a new repository for upcoming projects. You need to download our customized ``xv6`` source code again. The skeleton code for this project (PA2) is available as a branch named ``pa2``. Therefore, you should work on the ``pa2`` branch as follows:

```
$ git clone https://github.com/snu-csl/xv6-riscv-snu
$ git checkout pa2
```

The ``pa2`` branch has user-level utilities called ``setpgid`` and ``getpgid`` which simply call ``setpgid()`` and ``getpgid()`` system calls, respectively, with the given arguments. The source code of these utilities are available in ``./user/setpgid.c`` and ``./user/getpgid.c``. Using these utilities, you can change or read the PGID of the existing process.

Any kernel-level implementation for ``setpgid()`` or ``getpgid()`` is missing. So, if you run ``setpgid`` or ``getpgid`` utility in the shell, you will get an error as shown below.

```
$ make qemu
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 3G -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$ setpgid 1 100
3 setpgid: unknown sys call 22
setpgid: error
$ getpgid 1
4 getpgid: unknown sys call 23
getpgid: error
$
```

The following example shows what you should get when you implement ``setpgid()`` an ``getpgid()`` correctly.


```
$ make qemu
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 3G -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 1 starting
hart 2 starting
init: starting sh
$ ^p
1 1 sleep  init
2 1 sleep  sh

$ setpgid 1 100
$ ^p
1 100 sleep  init
2 1 sleep  sh

$ getpgid 1
pgid: 100
$ setpgid 2 20
$ ^p
1 100 sleep  init
2 20 sleep  sh

$ getpgid 2
pgid: 20
$ setpgid 2 0
$ ^p
1 100 sleep  init
2 2 sleep  sh

$ setpgid 0 0
$ ^p
1 100 sleep  init
2 2 sleep  sh

$ setpgid 0 -1
setpgid: error
```


## Restrictions

* You only need to change the files in the ``./kernel`` directory. Any other changes will be ignored during grading.

* The system call numbers for ``setpgid()`` and ``getpgid()`` are already assigned to 22 and 23, respectively, in the ``kernel/syscall.h`` file. Do not change these numbers.


## Hand in instructions

* For project submission and automatic grading, we use a dedicated server at https://sys.snu.ac.kr. Please register an account to the submission server by visiting https://sys.snu.ac.kr.
  * You must enter your real name & student ID
  * Wait for an approval from the admin
* By default, the ``sys.snu.ac.kr`` server is only accessible from the SNU campus network. If you want to access the server outside of the SNU campus, please send a mail to the TA.
* To submit your code, please run ``make submit`` in the ``xv6-riscv-snu`` directory. It will create a file named ``xv6-PA2-STUDENTID.tar.gz`` file in the parent directory. Please upload the file to the server.


## Logistics

* You will work on this project alone.
* Only the upload submitted before the deadline will receive the full credit. 25% of the credit will be deducted for every single day delay.
* __You can use up to 5 _slip days_ during this semester__. If your submission is delayed by 1 day and if you decided to use 1 slip day, there will be no penalty. In this case, you should explicitly declare the number of slip days you want to use with your submission. Saving the slip days for later projects is highly recommended!
* Any attempt to copy others' work will result in heavy penalty (for both the copier and the originator). Don't take a risk.

Have fun!

[Jin-Soo Kim](mailto:jinsoo.kim_AT_snu.ac.kr)  
[Systems Software and Architecture Laboratory](http://csl.snu.ac.kr)  
[Dept. of Computer Science and Engineering](http://cse.snu.ac.kr)  
[Seoul National University](http://www.snu.ac.kr)
