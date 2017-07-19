linear memory model
-------------------
From the point of view of a running program, memory can be understood as a long tape of bytes,  
and the program can read from or write to certain places on the tape.  
There are two concepts that we need to know:  
- *address*, an integer that specifies a place on the tape
- *data*, the byte stored in each place on the tape
![addressData](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/addressData.png)


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
    
    Data  is the compiled version of global variables and constants. 
    The information for each global variable in the source file will be stored in the executable file, 
    along with the initial value that was specified in the source file.
    
    When the user executes the executable file, the OS allocates memory for the program, 
    and loads the code and data into memory. 
    The subsystem that handles this workflow is part of the OS and is called the loader.
    Executable files can be copied to other computers, but since they are OS- and CPU-specific, 
    they need to match the computer’s environment in order to run appropriately.

virtual memory space for concurrent processeses
-----------------------------------------------
    Each process has a virtual memory space. 
    This is a memory address space that is isolated from other processes. 
    This means that a process can access its own virtual memory space, 
    but not the memory space of another process.
![virtualMemorySpace](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/virtualMemorySpace.png)


memory segments
---------------
    In order to effectively manage memory, 
    the loader defines certain areas of memory called segments, 
    which have different characteristics. 
    There are four important memory segments: code, static, stack, and heap.
    The areas between segments are not allocated for the running program. 
    Access to these areas is not allowed, and is called a segmentation fault. 
    In such cases, the OS will typically abort the program.
     
![memorySegments](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/memorySegments.png)


code segment
------------


static segment
--------------
    The static segment contains global variables which cannot be resized 
    and the memory cannot be freed while the program runs.

 
stack segment
-------------
    There is one stack frame per function call.
    A stack frame consists of Arguments, Local variables,  Return address.
    In reality, stack frames hold other information such as CPU states.
    The stack segment contains local variables that have fixed sizes, 
    and will be automatically freed after the function ends.
    The return address is used to indicate where to continue after the function is done.
![callStackAnimation](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/callStackAnimation.gif)


heap segment
------------
    Memory in the heap segment is managed through the standard library that is provided with the compiler. 
    The memory manager in the standard library allocates a pool of memory in advance. 
    This is a continuous area of memory which is split up into smaller chunks for each memory allocation.


physical memory and address
---------------------------
    When we boot a computer, the RAM installed on it will become accessible to the OS from a certain memory address. 
    The starting address of physical memory is assigned by the computer's chipset or SoC.
    When the OS boots, it allocates some memory for itself within this physical address space, 
    and keeps the rest reserved to give to processes
![physicalAddressMap](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/physicalAddressMap.png)


virtual and physical memory
---------------------------
    Physical addresses are accessible to the OS but not to processes. When processes access memory, 
    the CPU goes through a special virtual-to-physical memory address mapping which is created and managed by the OS. 
    The input to this mapping is a virtual memory address and the output is a physical memory address.
    The machine code instruction to read from the virtual memory is to have CPU find the find the corresponding physical 
    address from this amppling then CPU makes a read request to the RAM and RAM returns the data to CPU.
    Physical memory for each virtual memory segment can be mapped anywhere as long as the physical address is unused. 
    The order of the segments doesn't matter.
    
![virtualPhysicalMemoryMapping](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/virtualPhysicalMemoryMapping.png)


memory page
-----------
    A memory page is a chunk of memory that has a fixed size, with an address that is a multiple of the size. 
    The size and alignment of memory pages depend on the CPU architecture.
    All memory segments are allocated in memory pages.
    Often the amount of memory required is not an exact multiple of the size of a memory page, 
    so there is some memory is allocated but not used.
    
    [crtrader@CRT-017 ~]$ getconf PAGESIZE
    4096

    Memory pages are the units of virtual memory management. 
    When the OS creates the virtual-to-physical memory mapping, it maps one virtual memory page to one physical memory page. 
    The name of this mapping is the page table, and each record in the mapping is called a page table entry. 
    The OS manages the page tables, and there is one page table per process.
    
    There are multiple page table entries in a page table. 
    Each page table entry maps one virtual memory page to one physical memory page. 
    With this page table, the process can have an isolated memory space, which has continuous virtual addresses, 
    but the physical memory can be dispersed across the RAM module.
    
    We can see that the page table is acting as the interface between the OS and the MMU(a subsystem in the CPU called memory management unit). 
    The OS decides the virtual addresses and sets up the page table. 
    When the CPU accesses a virtual memory address, the MMU looks up the page table to find the corresponding physical address.
    
![virtualPhysicalPageMapping](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/virtualPhysicalPageMapping.png)    



process
-------
    A process is a unit of a running program which is managed by the OS. 
    Each time an executable file is executed, the OS creates a process, 
    allocates memory, and lets the process run.
    Each process has its own virtual memory, and the OS manages the mapping for each process 
    so that they don't clash with each other.
    The loader has the executable file to load. 
    It creates a process for the program and starts setting up the virtual memory by creating a page table. 
     
    
context switch
--------------
    The OS gives each process a certain amount of time to run, 
    then pauses it and switches to another process.
    
    
scheduling algorithm
--------------------

    
    