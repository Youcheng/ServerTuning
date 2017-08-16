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
The compiler leaves relocations for variables that have addresses that are unknown at compile time, 
and the loader resolves them at load time using the symbol table.

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
```
  There is a special processor register that is called the stack register which stores the top address of the call stack. 
  The compiler knows the size of all of the items that go into the stack frames. 
  The addresses of all of the items in the call stack can be referred to by an offset from the stack register.0
  Therefore, the compiler can generate machine code that accesses the function argument x.
```  
  ![staticOffset](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/stackOffset.png)

2. get the value from that address and store it in R2
3. get the address of g and store it in R3
4. get the value from that address and store it in R4
```
    Unlike the function argument x, the compiler cannot resolve the address of g at compile time, 
    and the machine code instruction to get the address of g cannot be completed at compile time.
    
    Instead the compiler leaves a “relocation”, which is an instruction for the loader to patch memory after loading. 
    A relocation consists of a global variable name and the location to be patched, 
    and is stored in compiled binary and executable files. In english, relocation means 
    finding the address of g and update the third instruction of func with it.
    
    In order for the loader to find the addresses of the global variables at runtime, 
    the compiler also stores the name and offset of all internal global variables in compiled binary and executable files:
    variable name: g; location: offset 24 bytes from the beginning of the static segment.
    
    The compiler stores all relocations in the relocation table in the executable file, 
    and lets the loader handle them when the addresses of the static segments have been fixed in the running processes.

    The loader will use symbol tables and relocation tables to patch the process memory after addresses are fixed.
 ```   
5. add R2 and R4 and store the result in R5
6. return R5

What is Position Independent Code?
----------------------------------
The loader updated the code segment due to a relocation. 
However, we want the code segment to be read-only so that we can take advantage of sharing pages across processes. 
That is why we need this PIC techinique. What we need is 1) code segments need to be read-only 2) static segments can be read-write.

Compilers have a special option to generate machine code that does not have any relocations for code segments. 
This option is called position independent code (PIC). 
For example, the option flag is -fPIC for the GNU compiler gcc, and -fpic for the the Intel compiler.
The idea of PIC is to generate code that makes indirect access to external variables. 
Without direct access there is no need for relocations for the code segment.

How Positions Independent Code works?
-------------------------------------
```
In order to generate code that indirectly accesses variables, 
the compiler introduces a special table called the global offset table (GOT) for each compiled binary file. 
The executable file has a GOT, and each shared object has one as well. 
The GOT is placed in the static segment along with the global variables. 
There is one entry per external global variable in the GOT.

When the PIC option is specified, the compiler generates position independent code using the GOT, 
which can be accessed without relocations. 
The GOT entries contain the addresses of external variables and are subject to relocations. 
Since the GOT is in the static segment, updating it for relocations meets our requirements: 
the code segments of shared libraries remain read-only, 
and can be effectively shared across processes through the page cache.
```
![PIC](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/PIC.png)
  
```
extern int g;
extern int h;

int func(int x) {
    return g + x;
}

int calc(int x, int y) {
    int z;
    z = 2 * x + y;
    return g + h + z;
}


The addresses X and Y are determined at runtime by the loader, 
which means that the compiler cannot resolve them at compile time. 
However, since the compiler generates the code and data, it knows the offset Z, 
which is the offset from the code to the GOT entry.

The compiler generates machine code that accesses the external variable g in the following way:
- get the address of the GOT entry for g and store it in R1 … (1)
- get the value at that address and store it in R2; this is the address of g … (2)
- get the value of g from that address and store it in R3

CPUs have an instruction register (sometimes called “program counter”, “instruction pointer”, etc.) 
that holds the address of the machine code instruction that the CPU is currently running, 
and therefore the compiler can generate code with an offset from the current instruction address. 
Assuming that the instruction register is IR, the machine code instruction for (1) will look something like this:
add IR and Z and store the result in R1. 
Regardless of the address assigned to X at runtime, 
the process can access the GOT by using the offset from the current instruction.

The code for (1) can be resolved with the offset from the instruction register. 
Next is the code for (2). The GOT is in the static segment, 

and it contains the address of the external variables.

For the variable g, the compiler will leave a relocation for the loader 
to put the address of `g` into the GOT entry  in order to resolve the address.
```

prodedure linkage table
-----------------------
```
Similar to using the GOT for external global variables, 
the compiler generates code that uses a procedure linkage table (PLT) for external function calls. 
PLTs are placed into the static segments, and each entry in a PLT is a JUMP instruction.
By default, all JUMP instructions will jump to a special function that we’ll call the lazy_bind function. 
```

```
For example, given a source file like the following:
double sqrt();
double pow();

double calc(void) {
    double x;
    x = sqrt(2.0);  // [*1]
    if (x > 10.0) {
        x = pow(2.0, 10.0);
    }
    return x;
}

The functions sqrt and pow are defined in a shared library. 
After compiling the source file above, the executable file has a PLT that looks like the following:
PLT #1: JUMP <lazy_bind>
PLT #2: JUMP <lazy_bind>

The lazy_bind function is a relocation resolution function that finds the address of the target function,  
updates the PLT entry with the found address, and then jumps to it.

Once the PLT entry has been updated, the flow is different for the second time that sqrt is called. 
Let’s say the calc function is called again and it calls sqrt. The CPU’s instruction register jump to
the 1st PLT entry, then the `sqrt` function 
```
![ProdecureLinkageTable0](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/ProdecureLinkageTable0.png)
![ProdecureLinkageTable](https://github.com/Youcheng/ServerTuning/blob/master/Memory/pictures/ProdecureLinkageTable.png)
