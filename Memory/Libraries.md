what is a library?
------------------
collections of reusable software resources

why do we need a library?
------------------------
We want to reuse software that has already been written instead of reinventing the wheel each time. 

how is a library used?
--------------------------
    Libraries may be provided as source files or as precompiled binary files.
    When we want to use a library that is provided as a precompiled binary, 
    we need to perform a step called linking. 
    This is a step to connect compiled chunks of programs to make them work together.

linking
-------
- static linking
- dynamic linking


static linking
--------------
        Static linking happens at compile time.
        At step (1), the compiler reads the source file (A), 
        generates compiled code and data,
        and stores it in the compiled binary file (B).
        We prepare the precompiled library file (C)At step (2),
        the linker takes the two binary files (B) and (C), 
        links them together, and outputs the executable file (D).
        After the executable file is generated, it can be loaded into memory as a process. 
        This part happens when the program is executed, and is called the run time. At step (3), 
        the loader allocates the code and static segments and loads the information from the executable file. 
        This load operation is performed via memory mapping.

![staticLinking](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/staticLinking.png)


dynamic linking
---------------
        The idea here is to perform the linking work when programs are loaded, and share the library code in the page cache.
        Compile time:
        At step (1), the compiler compiles the source file (A) and generates the executable file (B)
        The precompiled library file (C) is present but not linked yetRuntime when loading the program:
        At step (2), the loader will allocate code and static segments for the executable file (B)
        and for the precompiled library file (C), and load each respectivelyAt step (3), 
        the loader will link the segments in the process

![dynamicLinking](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/dynamicLinking.png)

what happens when multiple processes are using the same dynamic libraries?
--------------------------------------------------------------------------
        When we use libraries with dynamic linking, there can be multiple code and static segments in a process. 
        The segments of the libraries can be shared with other processes through page caching, 
        and this library structure is called shared libraries (or shared objects, dynamic shared objects, dynamic-link libraries, etc.)
        
        The code segment of the shared library is loaded via memory mapping and the permission is set to read only. 
        It is loaded into the page cache at most once, and shared across the processes that use the library. 
        This significantly reduces memory consumption and program load time. 
        (Note: some operating systems are not equipped with this feature).
        
        The static segment of the shared library is also loaded via memory mapping, but the permission is set to read-write.
        At program load time, the pages of the static segment will be shared, and when a write access occurs, 
        the page will become unshared due to the copy-on-write feature (the light red part in the figure above).
        

![dynamicLibShare](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/dynamicLibShare.png)
