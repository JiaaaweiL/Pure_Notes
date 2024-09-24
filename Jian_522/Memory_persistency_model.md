### If we use the DRAM, should we care about ordering? 
Suppose we don't have persistent memory, we care about the performance by using cache coherence protocol/Memory barriers. We do not care too much about ordering.  
But for NVM, we need to care about the ordering. Because we need to recover it when the system fails. When the crash happens, we need to make sure the data stored on the persistent memory should be consistent with the program order.  
EG: file system on disk. The file system should guarantee the way they put the data into the disk matches the operation you done. 
### two main point 
- persistent memory ordering is not visible to executing code
- Only visible following the crash
### EXAMPLE: CPU Cache may reorder the write to NVM, which breaks crash consistent update protocol.
- do transaction
- cache flush periodically. (INTEL CLFLASH command)
- write through cache(changing cache policy, but bad performance)

### Durable memory transaction: Compiler instrument's atomic locks just go to do transaction
ACID: Atomic Consistency Isolation durability   
![image](https://github.com/user-attachments/assets/a4a2e216-603f-4c36-a980-720d00a2b7d2)


### Epoch Barrier
- Definition of Epoch
  - A sequence of writes to NVM from the same thread
  - in-flight epoch: contains dirty data not yet reflected to NVM
  - in-flight epoch get commit( just like ooo )
  
**Implementation** 
- Per processor epoch ID tags writes
- The programmer should be familiar with the hardware, programmability is bad

**BPFS EPOCH MODEL**
- Per-thread transactions: Ordering between transactions within a thread is guaranteed.  
- Between-thread: must access persistent data from the previous epoch to enforce ordering（ensure the order between epochs)

### Intel Hardware Support
- CLFLUSH unordered version of CLFUSH, provides efficient cache flushing
- CLWB write back modified data of a cacheline similar to CLFUSHOPT
- PCOMMIT commit write data queued in the memory subsystem to NVM
