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
![readingFileUsingMemoryMapping](https://github.com/Youcheng/ServerTuning/blob/master/TechniquesForPeformance/pictures/readingFileUsingMemoryMapping.png)  
     
      
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
![unmappedPageTable](https://github.com/Youcheng/ServerTuning/blob/master/TechniquesForPeformance/pictures/unmappedPageTable.png)  


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
![demandLoad](https://github.com/Youcheng/ServerTuning/blob/master/TechniquesForPeformance/pictures/demandLoad.png) 
