#Application based Low-Power Optimization with Victim Caching in Embedded Processors

Added the implementation of a victim cache. Have also provided the additions to measure the power and performance metrics of the processor, when a victim cache is used. The victim cache is implemented as a fully-associative cache, inserted in between the level-1 and the level-2 data caches (or a unified level-2 cache). 

Simulation using sim-outorder can be performed with a victim cache by providing the additional command line argument: 
**`-cache:vc <name>:<nsets>:<bsize>:<assoc>:<repl>`**

This victim cache configuration is formatted similar to the general cache configuration format used in SimpleScalar.
Here, `<name>`  is the unique cache name, `<nsets>` is the number of sets in the cache (nsets should be 1 for a fully-associative cache), `<bsize>` is the block size, `<assoc>` is the associativity of the cache (for fully-associative cache, assoc will be the total number of blocks), and `<repl>`  is the replacement policy (repl can take the values ‘l’, ‘f’ or ‘r’, where l stands for LRU, f stands for FIFO, and r stands for random replacement). 

Example- The additional command line argument to be provided when executing sim-outorder, to create a victim cache with 4 blocks, each of size 32 bytes is “-cache:vc vc:1:32:4:l”. 
 
A separate function called `dl1_cache_access()` has been implemented to deal with accesses of the level-1 data cache dl1. Apart from the read and the write operations, a new operation called *WriteCleanEvictedBlkToVC* is also used here.
In the event of a dl1 cache miss, the block to be evicted from dl1 is identified based on the replacement policy prescribed for the level-1 data cache. The valid bit and the dirty bit of the block to be evicted, is then checked. If this block to be replaced is valid and dirty, a writeback is required to the level-2 data cache dl2 (or the unified level-2 cache ul2). Also, this evicted block should be inserted into the victim cache as well. Replacement is done in the victim cache, based on the replacement policy prescribed for the victim cache, provided as a command line argument when executing the simulator. If the block to be replaced is valid but not dirty, a writeback to dl2 (or ul2) is not required. However, this clean block still needs to be inserted into the victim cache. The WriteCleanEvictedBlkToVC operation is used in this case. An updated dl1 miss handler called `dl1_access_fn()` takes care of the above Read, Write and WriteCleanEvictedBlkToVC operations. The dl1_access_fn() returns the latency of the corresponding operation.
Now, the victim cache is searched to find the block that was not found in dl1, which resulted in the dl1 cache miss. If a hit in the victim cache is observed, the obtained block is then flushed from the victim cache and written in to dl1. If a miss in the victim cache is observed, the lower levels of memory are searched.
 
To calculate the power consumption of the victim cache, counters are initialized based on the victim cache configuration, provided as command-line argument to sim-outorder. Power estimates are then scaled dynamically based on victim cache accesses.
