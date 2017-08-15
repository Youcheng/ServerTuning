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
