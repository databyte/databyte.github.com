---
layout: post
title: Cheap USB 2.0 Drives
---

With prices of 16GB USB 3.0 drives hovering around $40, I was looking
for a cheap $10 option. Basic throw away drives that I didn’t care to
lose, didn’t copy to often but were really small and easy to pack.

I compared the [SanDisk Cruzer Blade](http://www.amazon.com/gp/product/B002U1ZBG0/ref=as_li_ss_tl?ie=UTF8&tag=mybl03-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=B002U1ZBG0)
against the [Kingston DataTraveler 108](http://www.amazon.com/gp/product/B005755U3U/ref=as_li_ss_tl?ie=UTF8&tag=mybl03-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=B005755U3U)
- both 16GB USB 2.0 flash drives. Both were formatted FAT as most USB
drives should work cross-platform as a simple sneaker net.

The SanDisk wins with a cumulative score of 654.6 compared to Kingston’s
score of 393.43. Benchmark results 
[available in a gist](https://gist.github.com/2964017).

The SanDisk Cruzer Blade won in every single test except Sequential
Uncached Read using 4K blocks and Random Uncached Writes using 256K
blocks. The SanDisk achieved 4.19 MB/sec and 0.51 MB/sec, respectively.
The Kingston pulled in 4.46 MB/sec and 2.70 MB/sec, respectively.

The numbers for the 4k blocks are very close but the gap in the single
256K block is fairly substantial. However, given that FAT32 is in 4k
sectors - the clear winner is still the SanDisk.

