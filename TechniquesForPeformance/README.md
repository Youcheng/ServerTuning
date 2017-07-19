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
    
memory mapping
==============
    