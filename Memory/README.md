process
-------
    A process is a unit of a running program which is managed by the OS. 
    Each time an executable file is executed, the OS creates a process, 
    allocates memory, and lets the process run.
    
context switch
--------------
    The OS gives each process a certain amount of time to run, 
    then pauses it and switches to another process.
    
    
scheduling algorithm
--------------------

virtual memory space
--------------------
Each process has a virtual memory space. 
This is a memory address space that is isolated from other processes. 
This means that a process can access its own virtual memory space, 
but not the memory space of another process.

![virtualMemorySpace](https://github.com/Youcheng/ServerTuning/blob/master/Memory/virtualMemorySpace.png)
    
    