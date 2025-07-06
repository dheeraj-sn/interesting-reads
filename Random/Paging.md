# Paging: The Foundation of Virtual Memory

In our article, [A Backend Engineer's Guide to OS Memory Management](./OSMemoryManagement.md), we introduced the concept of virtual memory. Now, we'll explore the mechanism that makes it all possible: **paging**.

Paging is a memory management scheme that allows the OS to store a process's memory in non-contiguous physical memory. It's the foundation of modern virtual memory systems.

## Pages, Frames, and Page Tables

To understand paging, you need to know three key terms:

*   **Page:** A fixed-size block of virtual memory.
*   **Frame:** A fixed-size block of physical memory. The size of a frame is the same as the size of a page.
*   **Page Table:** A data structure that maps virtual pages to physical frames.

When a process is started, the OS creates a page table for it. This page table keeps track of where each page of the process's virtual address space is stored in physical memory.

## The Address Translation Process

When your program accesses a memory address, the CPU's Memory Management Unit (MMU) performs the following steps:

1.  The virtual address is divided into two parts: a **page number** and an **offset**.
2.  The MMU uses the page number as an index into the process's page table to find the corresponding **frame number**.
3.  The frame number is combined with the offset to form the **physical address**.
4.  The MMU accesses the physical address in RAM.

This entire process is transparent to your application. It sees a large, contiguous address space, while the OS and the MMU handle the complexity of mapping it to physical memory.

## The Translation Lookaside Buffer (TLB)

Looking up the frame number in the page table for every memory access can be slow. To speed things up, the MMU uses a special cache called the **Translation Lookaside Buffer (TLB)**.

The TLB stores the most recently used page-to-frame mappings. When a virtual address is translated, the MMU first checks the TLB. If the mapping is found in the TLB (a "TLB hit"), the physical address is formed immediately. If the mapping is not in the TLB (a "TLB miss"), the MMU looks it up in the page table and then stores the mapping in the TLB for future use.

## Page Faults: When Memory Isn't There

A page fault occurs when a program tries to access a page that is not currently in physical memory. This can happen for two reasons:

1.  The page has been "paged out" to the disk to make room for other pages.
2.  The program is trying to access a memory address that is not part of its address space (a segmentation fault).

When a page fault occurs, the OS takes over:

1.  It checks if the memory access is valid.
2.  If it is, the OS finds the page on the disk.
3.  It loads the page into a free frame in physical memory.
4.  It updates the page table to reflect the new mapping.
5.  It resumes the program from the instruction that caused the fault.

## Multi-Level Page Tables

For a 64-bit system with a large virtual address space, a single page table can be very large. To solve this problem, modern systems use **multi-level page tables**. In this scheme, the page table itself is paged.

## Conclusion

Paging is a fundamental concept in modern operating systems. It's what makes virtual memory possible, and it's what allows us to run more applications than we have physical memory for. As a backend engineer, understanding how paging works will give you a deeper appreciation for what's happening under the hood of your applications.

## Further Reading

*   [A Backend Engineer's Guide to OS Memory Management](./OSMemoryManagement.md)
*   [Stack vs. Heap: A Developer's Guide to Memory Allocation](./StackAndHeap.md)
