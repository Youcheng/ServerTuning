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

what happens when multiple processes are using the same dynamic library?
--------------------------------------------------------------------------
        When we use libraries with dynamic linking, there can be multiple code and static segments in a process. 
        The segments of the libraries can be shared with other processes through page cache, 
        and this library structure is called shared libraries (or shared objects, dynamic shared objects, dynamic-link libraries, etc.)

        The code segment of the shared library is loaded via memory mapping and the permission is set to read only. 
        It is loaded into the page cache at most once, and shared across the processes that use the library. 
        This significantly reduces memory consumption and program load time. 
        (Note: some operating systems are not equipped with this feature).

        The static segment of the shared library is also loaded via memory mapping, but the permission is set to read-write.
        At program load time, the pages of the static segment will be shared, and when a write access occurs, 
        the page will become unshared due to the copy-on-write feature (the light red part in the figure above).
        
[page cache and memory mapping](https://github.com/Youcheng/ServerTuning/blob/master/OSTechniquesForPeformance/README.md)

![dynamicLibSharing](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/dynamicLibSharing.png)


relocation
----------
```
extern int g;

int func(int x) {
    return g + x;
}

```
The first line of the code above declares that a global variable g is defined somewhere else. 
The global variable may be defined in another .c source file or in a shared library.
Assume that each bullet point is a machine code instruction, and that R1, R2, ... are processor registers, 
which can be used to hold temporary data.

1. get the address of x and store it in R1   
  There is a special processor register that is called the stack register which stores the top address of the call stack. 
  The compiler knows the size of all of the items that go into the stack frames. 
  The addresses of all of the items in the call stack can be referred to by an offset from the stack register.0
  Therefore, the compiler can generate machine code that accesses the function argument x.
  ![staticOffset](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/stackOffset.png)

2. get the value from that address and store it in R2
3. get the address of g and store it in R3
4. get the value from that address and store it in R4
5. add R2 and R4 and store the result in R5
6. return R5

