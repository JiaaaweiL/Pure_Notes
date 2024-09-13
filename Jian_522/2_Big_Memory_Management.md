# Big Memory Management
### "Efficient Virtual Memory For Big Memory Servers"  

Key Point: How could we change the virtual memory system to support a big memory server?
  The Big Memory Workload is Important
    - graph analysis, databases, Memcached(?????)
   
  Analysis:
    - TLB miss burns up to 51% of execution cycles
    - Paging is not needed for almost all of their memory  
Why? Because they do not get benefit from the paging. Large companies like amazon, mata, users data in memory, and use disk only for backup.
Because they do not put data on disk. No swapping, no paging. So paging is not important.

**Their approach:** Using direct Segment:
- paged virtual memory where needed
- segmentation where possible

**Result:** Direct segment often eliminate 99% DTLB misses.


### Todays problem: Memory scale a lot. But TLB do not.    
![image](https://github.com/user-attachments/assets/d6c6a506-988c-4437-b54e-0ea1b81c39b7)  
Today: 128 TLB!  
**SO： as long as the access locality of the server workload decrease, the TLB is less effective.**    

5 years ago, lots of research about TLBs. Now most of them are about AI :(  

## How OS support for Huge page management
**Two common page sizes in modern processors: 2MB/1GB**   
- Ensure the corresponding flags in CPU info is configured.
    - PSE/PDPE
- Enable huge page when booting your system
    - CONFIG_HUGETLB_PAGE=y

**Challenges of using huge page**
1. Increase Tail Latency(searching, compaction, unpredictable performance)  
2. Fragmentation issue
3. Unfairness(race condition on regular page, huge page)
