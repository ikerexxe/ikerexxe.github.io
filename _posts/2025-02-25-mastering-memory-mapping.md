---
layout:     post
title:      "Mastering memory mapping: from virtual to physical addresses"
date:       2025-02-25 10:00:00 +0100
categories: computer-architecture
background: '/assets/figures/2025-02-25-mastering-memory-mapping.png'
---

I'm currently reading the [Computer Architecture: A quantitative approach](https://www.goodreads.com/book/show/70135.Computer_Architecture) book and after so many diagrams that are connected, but that never appear together I decided to summarize everything in a design. It's not intended to be complete, as there are many ways of solving this topic, but from a hypothetical perspective it gives a good perspective on how memory mapping works in current computer architecture.

# Introduction

This post examines the memory mapping flow within a hypothetical 64-bit computer architecture. The system employs a three-level cache hierarchy, starting with a Translation Lookaside Buffer (TLB) for virtual-to-physical address translation.  Subsequent levels include a 32KB L1 cache (organized into 256 sets with 2-way associativity and 512-bit block size) and a 4MB L2 cache (comprising 16,384 sets with 4-way associativity and 512-bit block size).  For the purpose of illustrating the mapping process, we'll present a simplified model that combines instruction and data caches.  We will not discuss block replacement algorithms or write-back mechanisms in this post.

# How memory mapping works?

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-02-25-mastering-memory-mapping.png){: .img-fluid}
| *Visual explanation of the memory mapping process* |
</div>

Numbers in red explain how the diagram should be read and each one explains the following:

1. The virtual address is divided into two parts: virtual page number and page offset.
2. The virtual page number is divided in two parts: TLB tag compare address and TLB index.
3. The TLB uses the TLB index to look up the corresponding entry in the tables.
4. The TLB tag is compared with the tag stored in the TLB entry.
5. TLB hit.
6. The L1 cache index is used to select the set in the cache.
7. The L1 cache tag is compared with the tag stored in the L1 cache entry.
8. L1 cache hit.
9. The block offset selects the offset from L1 data and this is sent to the CPU.
10. L1 cache miss.
11. Let's take back the physical address and divide it in: L2 tag compare address, L2 cache index and block offset.
12. The L2 cache index is used to select the set in the cache.
13. The L2 cache tag is compared with the tag stored in the L2 cache entry.
14. L2 cache hit.
15. The block offset selects the offset from L2 data and this is sent to the CPU.
16. Let's come back to the TLB cache miss.
17. The virtual address is divided in: canonical form, page-map L4 entry, page-directory-pointer entry, page directory entry, page table entry and page offset.
18. The CR3 register entry is summed to the page-map L4 to obtain the location of the page-directory-pointer table.
19. The page-directory-pointer table is summed to the page-directory-pointer entry to obtain the location of the page directory table.
20. The page directory table is summed to the page directory entry to obtain the location of the page table entry.
21. The page table is summed to the page table entry to obtain the physical address.
22. The physical address is the physical page frame number.
23. The physical page frame number and the page offset indicate the exact location in main memory.

# Conclusion

I enjoyed understanding and explaining the memory mapping process in this hypothetical architecture. I hope you found it helpful and that it demystified some of the inner workings of computer memory. Thanks for reading!
