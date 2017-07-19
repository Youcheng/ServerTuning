executable
----------
    There are two essential pieces of information that are stored in the executable file: code and data.
     
    Code is the compiled version of functions: each function in the source file will be translated into a chunk of machine code, 
    and stored in the executable file. Machine code is a set of instructions that will be executed by the CPU. 
    Each instruction is a basic operation like reading from, and writing to, 
    memory, adding and subtracting numbers, and so forth. 
    Although all CPUs have similar concepts, 
    the machine code is different for each CPU architecture. 
    For example, the x86-64 architecture has its own machine code 
    which is different from the code of the ARM architecture.
    
    Data
    
    When the user executes the executable file, the OS allocates memory for the program, 
    and loads the code and data into memory. 
    The subsystem that handles this workflow is part of the OS and is called the loader.
    Executable files can be copied to other computers, but since they are OS- and CPU-specific, 
    they need to match the computer’s environment in order to run appropriately.

memory segments
---------------
    In order to effectively manage memory, 
    the loader defines certain areas of memory called segments, 
    which have different characteristics. 
    There are four important memory segments: code, static, stack, and heap.
    The areas between segments are not allocated for the running program. 
    Access to these areas is not allowed, and is called a segmentation fault. 
    In such cases, the OS will typically abort the program.
     
![memorySegments](https://github.com/Youcheng/ServerTuning/blob/master/Memory/memorySegments.png)


stack segment
-------------
    There is one stack frame per function call.
    A stack frame consists of Arguments, Local variables,  Return address.
    In reality, stack frames hold other information such as CPU states.
    Since the memory layout of the stack frame is fixed, 
    the size of variables cannot grow or shrink.
    The return address is used to indicate where to continue after the function is done.
![callStackAnimation](https://github.com/Youcheng/ServerTuning/blob/master/Memory/callStackAnimation.gif)



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
    
    