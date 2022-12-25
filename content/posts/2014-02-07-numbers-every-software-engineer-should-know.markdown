---
categories: performance snippet
comments: true
date: '2014-02-07T00:00:00Z'
title: Numbers Every Software Engineer Should Know
---

This article is a collection of runtime and memory usage numbers that
might be useful to keep in mind when building performance critical software.

#### Big-O Absolute Numbers

This chart shows how well or poorly certain algorithm complexities scale
when `n` grows.

    Function   | n=8 (2^3) | n=64 (2^6)  | n=1024 (2^10) | n=2^32 (~ 4 billion)
    -----------|-----------|-------------|---------------|---------------------
    log(n)     | 3         | 6           | 10            | 32
    n          | 8         | 64          | 1024          | 4294967296
    n*log(n)   | 24        | 384         | 10240         | 1.374...E11
    n^2        | 64        | 4096        | 1048576       | 1.844...E19
    n^2*log(n) | 192       | 24576       | 1.048...E7    | -
    n^3        | 512       | 262144      | -             | -
    2^n        | 256       | 1.844...E19 | -             | -
    n!         | 40320     | 1.268...E89 | -             | -

#### Runtime Performance Numbers

##### Big-O Runtime Performance Numbers

This chart shows how well or poorly certain algorithm complexities scale when `n` grows,
in the context of CPU runtime. Exponentials and factorial algorithms clearly
do not scale to values of `n` of any significant size.

    Function   | n=8 (2^3) | n=64 (2^6)       | n=1024 (2^10) | n=2^32 (~ 4 billion)
    -----------|-----------|------------------|---------------|---------------------
    log(n)     | 3   ns    | 6   ns           | 10 ns         | 32  ns
    n          | 8   ns    | 64  ns           | 1  µs         | 4.2 seconds
    n*log(n)   | 24  ns    | 384 ns           | 10 µs         | 137 seconds
    n^2        | 64  ns    | 4   µs           | 1  ms         | 584 years
    n^2*log(n) | 192 ns    | 24  µs           | 10 ms         | forever
    n^3        | 512 ns    | 262 µs           | 1  second     | forever
    2^n        | 256 ns    | 584 years        | forever       | forever
    n!         | 40  µs    | forever          | forever       | forever

    ns      = Nanoseconds, about the time it takes for a CPU instruction cycle to run
    µs      = 1,000 ns
    ms      = 1,000 µs
    forever = way way longer than the lifetime of the universe of 13.798 billion years

##### Latency Numbers

This chart shows the round trip latency for certain computing actions.

     Action                             | Time     | Comparisons
    ------------------------------------|----------|-----------------------------
     L1 cache reference                 | 0.5   ns |
     Branch mispredict                  | 5     ns |
     L2 cache reference                 | 7     ns | 14x L1 cache
     Mutex lock/unlock                  | 25    ns |
     Main memory reference              | 100   ns | 20x L2 cache, 200x L1 cache
     Compress 1K bytes with Zippy       | 3,000 ns |
     Send 1K bytes over 1 Gbps network  | 10    µs |
     Read 4K randomly from SSD*         | 150   µs |
     Read 1 MB sequentially from memory | 250   µs |
     Round trip within same datacenter  | 500   µs |
     Read 1 MB sequentially from SSD*   | 1     ms | 4X memory
     Disk seek                          | 10    ms | 20x datacenter roundtrip
     Read 1 MB sequentially from disk   | 20    ms | 80x memory, 20X SSD
     Send packet CA->Netherlands->CA    | 150   ms |

    + By Jeff Dean: http://research.google.com/people/jeff/
    + Originally by Peter Norvig: http://norvig.com/21-days.html#answers
    + Retrieved from Jonas Bonér: https://gist.github.com/jboner/2841832

#### Memory and Storage Numbers

##### Big-O Storage size

This chart shows how well or poorly certain algorithm complexities scale when `n` grows,
in the context of memory and storage.

    Function   | n=8 (2^3) | n=64 (2^6)    | n=1024 (2^10) | n=2^32 (~ 4 billion)
    -----------|-----------|---------------|---------------|---------------------
    log(n)     | 3   B     | 6   B         | 10 B          | 32 B
    n          | 8   B     | 64  B         | 1  KB         | 4 GB
    n*log(n)   | 24  B     | 384 B         | 10 KB         | 128 GB
    n^2        | 64  B     | 40  KB        | 1  MB         | 16 exabytes
    n^2*log(n) | 192 B     | 24  KB        | 10 MB         | ...
    n^3        | 512 B     | 256 KB        | 1  GB         | ...
    2^n        | 256 B     | 16  exaBytes  | ...           | ...
    n!         | 40  KB    | ...           | ...           | ...

    B       = 1 Byte, or 8 Bits
    KB      = kilobyte (1024 B)
    MB      = megabyte (1024 KB)
    GB      = gigabyte (1024 MB)
    TB      = terabyte (1024 GB)
    TB      = petabyte (1024 TB)
    exabyte = 1024 petabytes, or 1 million computers with 1 terabyte HDs
    ...     = there isn't enough room on earth to store the required computers.

##### Powers of Two Table

This chart shows how much memory is required to store bits in powers of two. For example, it takes
4GB of memory to store 2^32 bits in memory, and would require 16GB of memory to store
an array of 32-bit integers. This is useful for knowing when a hash table is appropriate for a problem.

    Power of 2 | Exact Value       | Storage size
    -----------|-------------------|-------------
    7          | 128               | 128 B
    8          | 256               | 256 B
    10         | 1024              | 1   KB
    16         | 65,536            | 64  KB
    20         | 1,048,576         | 1   MB
    30         | 1,073,741,824     | 1   GB
    32         | 4,294,967,296     | 4   GB
    40         | 1,099,511,627,776 | 1   TB
    64         | 1.844...E19       | 16  exabytes

    + Modified from _Cracking the Coding Interview_ By Gayle L. McDowell (p. 47)
