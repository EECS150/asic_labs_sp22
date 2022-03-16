# EECS 151/251A ASIC Project Specification: Checkpoint 3
<p align="center">
Prof. Sophia Shao
</p>
<p align="center">
TAs (ASIC): Dima Nikiforov
</p>
<p align="center">
Department of Electrical Engineering and Computer Science
</p>
<p align="center">
College of Engineering, University of California, Berkeley
</p>

---

## Cache
A processor operates on data in memory. Memory can hold billions of bits, which can either be instructions or data. In a VLSI design, it is a very bad idea to store this many bits close to the processor. The
chip area required would be huge - consider how many DRAM chips your PC has, and that DRAM cells
are much smaller than SRAM cells (which can actually be implemented in the same CMOS process).
Moreover, the entire processor would have to slow down to accommodate delays in the large memory
array. Instead, caches are used to create the illusion of a large memory with low latency.

Your task is to implement a (relatively) simple cache for your RISC-V processor, based on some
predefined SRAM macros (memory arrays) and the interface specified below.

### 1 Cache overview
When you request data at a given address, the cache will see if it is stored locally. If it is (cache hit), it
is returned immediately. Otherwise if it is not found (cache miss), the cache fetches the bits from the
main memory.
Caches store data in “ways.” A way is a logical element which contains valid bits, tag bits, and data.
The simplest type of cache is direct-mapped (a 1-way cache). A cache stores data in larger units (lines)
than single words. In each way, a given address may only occupy a single location, determined by the
lowest bits of the cache line address. The remaining address bits are called the “tag” and are stored so
that we can check if a given cache line belongs to a given address. The valid bit indicates which lines
contain valid data.
Multi-way caches allow more flexibility in what data is stored in the cache, since there are multiple
locations for a line to occupy (the number of ways). For this reason, a ”replacement policy” is needed.
This is used to decide which way’s data to evict when fetching new data. For this project you may use
any policy you wish, but pseudo-random is recommended.

### 2 Guidelines and requirements
You have been given the interface of a cache (`Cache.v`) and your next task is to implement the cache.
EECS151 students should build a direct-mapped cache, and EECS251 students are required to implement a cache that either:

1. is configurable to be either direct-mapped or at least 2-way set associative; or

2. is set-associative with configurable associativity.

You are welcome to implement a more performant cache if you desire.
Your cache should be at least 512 bytes; if you wish to increase the size, implement the 512 bytes
cache first and upgrade later.
Use the SRAMs that are available in

`/home/ff/eecs151/hammer/src/hammer-vlsi/technology/asap7/sram_compiler/memories/behavioral/sram_behav_models.v`

for your data and tag arrays.


The pin descriptions for these SRAMs are as follows:

|             |                                                 |
|-------------|-------------------------------------------------|
| `A`         | Address                                         |
| `CE`        | clock edge                                      |
| `OEB`       | output enable bar (tie this to 0)               |
| `WEB`       | write enable bar (1 is a read, 0 is a write)    |
| `CSB`       | chip select bar (tie this to 0)                 |
| `BYTEMASK`  | write byte mask                                 |
| `I`         | write data                                      |
| `O`         | read data                                       |

You should use cache lines that are 512 bits (16 words) for this project. The memory interface is
128 bits, meaning that you will require multiple (4) cycles to perform memory transactions.
Below find a description of each signal in `Cache.v`:

|                      |                                        |
|----------------------|----------------------------------------|
| `clk`                  | clock
| `reset`                | reset
| `cpu_req_valid`        | The CPU is requesting a memory transaction
| `cpu_req_rdy`          | The cache is ready for a CPU memory transaction
| `cpu_req_addr`         | The address of the CPU memory transaction
| `cpu_req_data`         | The write data for a CPU memory write (ignored on reads)
| `cpu_req_write`        | The 4-bit write mask for a CPU memory transaction (each bit corresponds to the byte address within the word). `4’b0000` indicates a read.
| `cpu_resp_val`         | The cache has output valid data to the CPU after a memory read
| `cpu_resp_data`        | The data requested by the CPU
| `mem_req_val`          | The cache is requesting a memory transaction to main memory
| `mem_req_rdy`          | Main memory is ready for the cache to provide a memory address
| `mem_req_addr`         | The address of the main memory transaction from the cache. Note that this address is narrower than the CPU byte address since main memory has wider data._
| `mem_req_rw`           | 1 if the main memory transaction is a write; 0 for a read.
| `mem_req_data_valid`   | The cache is providing write data to main memory.
| `mem_req_data_ready`   | Main memory is ready for the cache to provide write data.
| `mem_req_data_bits`    | Data to write to main memory from the cache (128 bits/4 words).
| `mem_req_data_mask`    | Byte-level write mask to main memory. May be `16’hFFFF` for a full write.
| `mem_resp_val`         | The main memory response data is valid.
| `mem_resp_data`        | Main memory response data to the cache (128 bits/4 words).
|                        |

To design your cache, start by outlining where the SRAMs should go. You should include an SRAM
per way for data, and a separate SRAM per way for the tags. Depending on your implementation, you
may want to implement the valid bits in flip flops or as part of the tag SRAM.

Next you should develop a state machine that covers all the events that your cache needs to handle
for both hits and misses. You can do it without an explicit state machine, but you will suffer. Keep in
mind you will need to write any valid data back to main memory before you start refilling the cache (you
can use a write-back or a write-through policy). Both of these transactions will take multiple cycles.

### 3 Changes to the flow for this checkpoint
You should now be able to pass the `bmark` test. The test suite includes many C programs that do
various things to test your processor and cache implementation. You can observe the number of cycles
that each bmark test takes to run by opening `bmark_output/*.out` and taking note of the number
on the last line. The `make sim-rtl test bmark=all` target will also print this number for you.
To run a specific benchmark (e.g., cachetest), run
```
make sim-rtl test_bmark=cachetest.out
```
After completing your cache, run the tests with both the cache included and with the fake memory
(`no_cache_mem`) included. To use no_cache_mem be sure to have `+define+no_cache_mem` in the
simOptions variable in the `sim-rtl.yml` file. To use your cache, comment out `+define+no_cache_mem`.
Take note of the cycle counts for both, you should see the cycle counts increase when you use the cache.

### 4 Checkpoint 3 Deliverables
*Checkoff due: Apr 29 (Friday), 2022*

1. Show that all of the assembly tests and final pass using the cache

2. Show the block diagram of your cache

3. What was the difference in the cycle count for the `bmark` test with the perfect memory and the
cache?

4. Show your final pipeline diagram, updated to match the code

---


## Acknowledgement

This project is the result of the work of many EECS151/251 GSIs over the years including:
Written By:
- Nathan Narevsky (2014, 2017)
- Brian Zimmer (2014)
Modified By:
- John Wright (2015,2016)
- Ali Moin (2018)
- Arya Reais-Parsi (2019)
- Cem Yalcin (2019)
- Tan Nguyen (2020)
- Harrison Liew (2020)
- Sean Huang (2021)
- Daniel Grubb, Nayiri Krzysztofowicz, Zhaokai Liu (2021)
- Dima Nikiforov (2022)
