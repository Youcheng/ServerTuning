page cache
==========
    In order to keep the recently accessed files in memory,   
    the OS uses the page cache, which is an in-memory copy of files.
    Since the amount of memory that can be used for the page cache is finite, 
    the OS may have to drop an existing file before loading a new file into the page cache.  
    Since the original files are stored on disk, 
    it is safe to drop files from the page cache whenever the OS needs to free up memory for other purposes.     

use page cache to read files
----------------------------
When a file is accessed, the page caching algorithm performs the following steps:
1. check if the file exists in the page cache
2. if absent, load the file into the page cache
3. return the data from the page cache


use page cache to write to files
--------------------------------
Since we can write to files, we can also write to the page cache.   
It is common to write small chunks of data frequently.   
For example, when we write logs to a file, 
each log record is small and we may write many records in a short period of time. 

The workflow for write requests would look like the following:
1. write the data to the page in the page cache
2. mark the page as “dirty” (this means “need to be flushed to disk”)
3. flush the page to disk after a certain period of time
4. mark the page as “clean” (this means “no need to flush to disk”)    
    
Instead of flushing to disk for each write instruction,   
the data is buffered in the dirty page.   
There can be multiple write instructions while the page is dirty,   
which reduces the number of times a write to disk is performed,   
and increases the overall performance.  
   
    
memory mapping
==============
how to read file to process virtual memory without memory mapping
-----------------------------------------------------------------
- Disk-to-RAM-copy  
  copy data from hard disk to page cache
- RAM-to-RAM-copy  
  copy data from page cache in the physical memory 
  to the physical memory which is mapped to the process's virtual memory, like stack segment.
      
why do we need memory mapping
-----------------------------      
    Both page cache and process's virtual memory are in the RAM which seems redundant.
    Since the data is already in RAM(page cache), it would be great if the process could
    use it directly without copying it from the page cache to the process's memory again.
    This is done by memory mapping.
          
abstract code to use memory mapping
-----------------------------------
````
void data_processing_2() {
    char *head;
    char *rec;
    head = map_file(file_name);
    for (int i = 0; i < 1000; i++) {
        rec = head + 200 * i;
        /* use the data somehow */
    }
    unmap_file(head);
}
````

how does memory mapping work
----------------------------
    Instead of opening the file, we map it to memory. 
    This creates a virtual-to-physical memory mapping. 
    The process can access the data in the page cache through its virtual memory. 
    The beginning address of the mapping is stored in the head variable. 
    Then, the pointer variable rec is set to the address of each record in the loop,
    and the process uses it to refer to the record data.
   
workflow for reading files using memory mapping
-----------------------------------------------    
    a) The OS loads the file from disk into the page cache if absent.  
       If the file is already present in the page cache, the OS skips the load from disk, which results in a significant performance gain.    
    b) The OS creates a virtual-to-physical memory mapping.  
    c) The process uses the record data referred to by the rec variable directly.      
![readingFileUsingMemoryMapping](https://github.com/Youcheng/ServerTuning/blob/master/OSTechniquesForPeformance/pictures/readingFileUsingMemoryMapping.png)  
     
      
demand load
==============   
Partial loading and skips unnecessary pages from being loaded.   

how does demand load work
------------------------
    When the memory mapping system call is invoked, 
    the OS allocates the pages in the process’ virtual memory space. 
    From the programmer’s point of view, the OS creates a virtual-to-physical memory mapping,
    but the OS actually defers that work. 
    Instead of creating the virtual-to-physical memory mappings, 
    the OS leaves the page table entry marked as “unmapped”, 
    and continues executing the process.
![unmappedPageTable](https://github.com/Youcheng/ServerTuning/blob/master/OSTechniquesForPeformance/pictures/unmappedPageTable.png)  


    The process, however, assumes that the memory mapping is set up, 
    and will access the memory of the mapping.
    
    Let’s say it accesses an address in the first page.
    In regular cases where virtual-to-physical memory mappings exist, 
    the MMU (a subsystem of the CPU) can look up the physical memory page that 
    corresponds to a virtual memory page. However, in this case, 
    the virtual memory page exists but the page table entry does not 
    have a corresponding physical memory page. 
    
    Therefore, the MMU will fail when it tries to look up the physical memory page, 
    and raise a special exception that is called a page fault. 
    The MMU will send this exception to the OS to handle the failure. 
    
    The OS knows that the page fault is due to the physical page being absent, 
    and it also knows that the original file is on the disk. 
    
    The OS finds an unused physical memory page and loads the data into it from disk, 
    then updates the page table entry by setting up the virtual-to-physical memory mapping. 
    Typically, the OS will use DMA (direct memory access) modules for the data transfer.
    
    The process gets paused when it accesses an address in the first page. 
    After the MMU and OS handle the missing physical memory page, 
    the process gets resumed and runs the memory access instruction again. 
    The process does not need to be aware of all of the OS work that happened. 
    It can simply expect that the memory mapping is available for use at any time.
![demandLoad](https://github.com/Youcheng/ServerTuning/blob/master/OSTechniquesForPeformance/pictures/demandLoad.png) 



page sharing
============
With the page cache, we can keep files that were recently accessed in memory to increase performance.   
The same files may be accessed by multiple processes simultaneously.
 
    The two processes have their own virtual memory spaces, 
    and each has a mapping to the physical memory address in the page cache. 
    The mappings are both for the same file.
![pageSharingByProcesses](https://github.com/Youcheng/ServerTuning/blob/master/OSTechniquesForPeformance/pictures/pageSharingByProcesses.png) 


    When creating memory mappings, we can set the permission of the mapping.
    
    The permission can be set to read only. 
    When the process attempts write access to read only pages, 
    the MMU raises a special exception and informs the OS to handle the failure. 
    Typically, the OS aborts the process.
    
    The permission can also be set to read and write. When the process attempts write access, the memory is updated. 
    There are additional flags that we can specify if the updates to memory should be “private” or “shared”. 
    When we specify the “shared” flag, the updates to memory are carried through to the underlying file on disk, 
    and are available to other processes that map the same file. 
    When we specify the “private” flag, the updates to the data in memory are made only in the process’ virtual memory 
    and are not reflected in the underlying file on disk, nor are they visible to other processes.
    
how does write with private flag work
-------------------------------------
    In order to achieve this result, the OS uses an algorithm called “copy-on-write”. 
    Let's assume that there are two processes that map the same file with the above settings, 
    process A and process B. The memory looks like the following diagram. 
![copyOnWriteBeforeWrite](https://github.com/Youcheng/ServerTuning/blob/master/OSTechniquesForPeformance/pictures/copyOnWriteBeforeWrite.png) 

    Although we want to have read and write access, 
    the OS actually creates the memory mappings with read only permissions at first. 
    The two processes can read from the memory through their own virtual memory space. 
    What happens when process B attempts to write to a memory address for the memory mapping at this point?
    The MMU informs the OS that there was a page fault. 
    The OS knows that this is due to a write access to a read only page. 
    The OS first checks whether this was due to an actual illegal write instruction, 
    or whether the process had, in fact, requested write permissions for this memory. 
    In the latter case, the OS executes the copy-on-write algorithm. 
    While the process is paused, the OS: 
    (1) allocates another memory page,
    (2) copies the content of the original page that was accessed,
    (3) remaps the virtual-to-physical memory mapping for the page that was accessed
    The new memory page (light red) has read and write permissions in the page table entry. 
    The process that was paused at the write instruction is resumed, and the write access succeeds. 
    From the process’ point of view, it looks like the memory was always writable!
![copyOnWriteAfterWrite](https://github.com/Youcheng/ServerTuning/blob/master/OSTechniquesForPeformance/pictures/copyOnWriteAfterWrite.png) 
    