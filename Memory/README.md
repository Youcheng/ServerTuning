executable
----------
    There are two essential pieces of information that are stored in the executable file: code and data.
     
    **Code** is the compiled version of functions: each function in the source file will be translated into a chunk of machine code, 
    and stored in the executable file. Machine code is a set of instructions that will be executed by the CPU. 
    Each instruction is a basic operation like reading from, and writing to, 
    memory, adding and subtracting numbers, and so forth. 
    Although all CPUs have similar concepts, 
    the machine code is different for each CPU architecture. 
    For example, the x86-64 architecture has its own machine code 
    which is different from the code of the ARM architecture.
    
    

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
    
    