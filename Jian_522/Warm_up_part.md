### Warm up: Memory v.s storage
- Access granularity  
  Memory: byte-addressable  
  Storage: page/block granularity
- Execution workflow  
  Memory can stall the instruction executions (e.g., load, store)  
  Storage may not (e.g., asynchronous I/O)
- Durability  
  volatile or not  
- Software support  
  Memory: virtual memory manager  
  Storage: file system

### Address translation process
  ![image](https://github.com/user-attachments/assets/3aed26e0-7612-4309-93e8-903c10c2501f)

# Introduction of Virtual Memory:
## Virtual Memory Management:
  1. Segmentation
  2. Paging
- Virtual memory requires HW+SW support

## Segmentation  
Not as popular today.  
Divided the address into segments, Physical Address = Base + Address.  
- Pros: 
Isolation/Protection  
Translation is simple    
- Cons:  
Complicated management, different segment sizes, the programmer has to manage the segment base/limit  
Fragmentation, wastes the spaces.    
Only a few segments are addressable at the same time(base/limit are placed in reg, and the #reg are limited)

## Paging   
Page are fixed-sized chunks of the address space.   
Using page table to store the VA to PA mapping. 
- Problem#1: Too large mapping    
Using hierarchical page table (only do the allocation when we need it)   
- Problem#2: Large latency  
Using translation lookaside buffer to cache the page table.

## Translation
- Flat Page table
one-to-one mapping,  20 bit for index(VPN to PPN) and 12bit for offset(4KB), identifying which byte we want to access

- Two level page table
![image](https://github.com/user-attachments/assets/a10c8ad9-7f88-4fd2-b75a-05f9db01de98)
CR3 Register points to the page table base address. When OS do context switch, it should flush the CR3 register(CR3 store the physical address)  

**The page table is per-process!  EVERY  process has its own page table. (OS) response to malloc the page table**    
Why multi-level page table save memory space? We only allocate PTE when we using it(we need to malloc the whole page table if we use direct map)  


## Protection
Four privilege level in X86(Ring 0 to Ring3)    
- Ring0: Highest privilege(operating system)  
- Ring3: Lowest privilege(user application)  

CPL: current privilege level determined by:
- The address of the instruction
- The DPL(descriptor privilege level) of the code segment

We put the FLAGS on the PDE(page directory entry) and PTE(page table entry) to do the protection.  PDE will have a larger granularity than the PTE.    
- Paging provides protection by PDE+PTE’s flags  
- Segmentation also provide protection by segment descriptor   

## TLB Management
TLB: A hardware structure where PTEs are cached.  
Context switches: 
  1) flush TLB(80836)  
  2) Associate TLB entries with processes.（modern X86, MIPS)    

**Hardware managed TLB miss (X86) OR  Software managed TLB miss (MIPS).**  
Hardware managed TLB
- Pro: 
  No exceptions, Instruction just stalls  
  Independent instructions may continue  
  Small footprint(no extra instructions/data)  
- Con:
  Page directory/table organizations etched in stone   

Software-Managed TLB  

- Pro:   
  The OS can design the page directory / table    
  More advanced TLB replacement policy  
- Con:
  Flushes Pipeline  
  Performance overhead.   

## Page Fault
1. Do not have the permission to access to the memory
2. The data is not in the memory

**Why map to the disk?**  
- 1. Demand Paging  
When a large file in disk needs to be read, not all of it is loaded into memory at once    
  Instead, page-sized chunks are loaded on-demand  
  If most of the file is never actually read …  
  Saves time (remember, disk is extremely slow)  
  Saves memory space  

- 2. Swapping  
There is no space in the memory. So there are some physical pages are swapped out to the disk, and free up those physical pages.   
When you access a physical page that has been swapped out, only then is it brought back into physical memory  
  This may cause another physical page to be swapped out
  If this “ping-ponging” occurs frequently, it is called thrashing

## Page size
Common are 4KB, large page size are 1GB.   
- Pro:  
  1. Lagre page size have fewer PTEs required -> Save memory spaces  
  2. Few TLB miss -> Improve performance  
- Con:  
  1. Cannot have fine-grained permission.  
  2. Large transfer from/to disk, waste BW.  
  3. Internal/External Fragmentation(Some space will be wasted. Because of the granularity?)  **?????**  
  **USE IT when program has very good locality**

