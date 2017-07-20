kernel tuning
=============

[LinuxKernelParameters](https://www.kernel.org/doc/Documentation/admin-guide/kernel-parameters.txt)


isolate cores from being used by userspace threads
--------------------------------------------------
    Isolate CPU cores from userspace threads unless specifically configured to do so.
    Kernel threads continue to run on these isolated cores.
    It could be set up from /proc/cmdline, like isolcpus=2-11,14-23

move kernel threads away from isolated cores
--------------------------------------------
    # tuna --cpus=<Isolated cores> --no_uthreads --isolate
        --cpus=CPU-LIST
        --no_uthreads, Operations will not affect user threads.
        --isolate, Move all threads away from CPU-LIST
 
prevent kernel timer tick from running on isolated cores
--------------------------------------------------------
    It could be set up from /proc/cmdline, like nohz_full=2-11,14-23.
    The cpu list will be forced outside the range to maintain the timekeeping.
    The CPUs in this range must also be included in the rcu_nocbs=2-11,14-23. 

[Redhat Tuna](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_MRG/1.3/html-single/Tuna_User_Guide/index.html) - Tuna is a tool that can be used to adjust scheduler tunables


disable realtime throttling
---------------------------
    Realtime throttling is a linux kernel feature that throttles realtime threads (SCHED_FIFO) to 95% of cpu time.
    We could disable it to give the accelerated threads full use of the cores.
    # echo -1 > /proc/sys/kernel/sched_rt_runtime_us
[SCHED_FIFO and realtime throttling](https://lwn.net/Articles/296419/) - A good explanation