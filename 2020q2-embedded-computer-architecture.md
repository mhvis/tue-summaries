
# Embedded Computer Architecture (2020-Q2)


* CPI: cycles per instruction
* t_exec = N_instr * CPI * t_cycle


### RISC characteristics

Need to know the 6 or 7 RISC characteristics and the reasoning.

## 25-11-2020: Something on Virtual Memory

* Processor architecture: user mode vs supervisory mode
* Translation Lookaside Buffer (TLB): a cache for address translations.
* Page fault!

Some questions:

* Explain how caches operate
* Calculate the cache impact on CPI
* Explain TLBs
* Why do we need virtual memory, how does it work
* Which cache optimizations reduce hit-time, or miss-rate, or miss-time
* Understand modern processor memory architectures (e.g. ARM)


## 30-11-2020: Instruction Level Parallelism

* Out-of-Order execution
* CPI = CPI_ideal + SUM{ f_hazard }

RECAP: Pipelining hazards, which delay the pipeline and increase CPI:

* Structural: multiple instructions need access to the same hardware at the same time.
* Data dependence
* Control dependence: when jumping, the pipeline has to be squashed and refilled.

CPI = CPI_base + sum_i {CPI_{hazard i}}

CPI_hazard = f_hazard * Cycle_penalty_hazard

f_hazard = fraction of occurence of this hazard [0..1]

Data dependencies:

* RaW: read after write
* WaR: write after read
* WaW: write after write

WaR and WaW are false dependencies which can be avoided by renaming, if sufficient registers are available.

### Avoiding pipeline stalls

* Structural: buy more hardware
* Data dependence:
  * Real: add forwarding/bypassing logic
  * False: use renaming
* Control dependence: branch prediction, avoiding branches, add pipeline HW to reduce branching impact

### Out-of-Order architectures (H&P 3.4-3.6)

Dynamic scheduling: HW rearranges instruction execution to reduce stalls

Advantages of dynamic scheduling: handles cases when dependencies are unknown at compile time, simplifies the compiler, binary compatible (code doesn't need to be recompiled), allows hardware speculation.

Superscalar concept: reservation stations in front of FUs. Instruction decoder can keep on working until a reservation station is full.

Reservation station entry contains 3 fields: instruction, buffered operand values, reservation station number of instruction providing the operand values.

Speculation: add a reorder buffer (RoB), which delays commits to register files and allow speculation of branches without committing results immediately. On misprediction, speculated entries in RoB are cleared.

Other way of register renaming: Register Alias Table (RAT), a mapping between logical and physical registers. The physical register file is larger than logical register file. Need multiple RATs for branches, to store state before branching.

### Branch prediction

#### 1-bit Branch Prediction Buffer or Branch History Table (BHT)

1 bit which remembers if that branch was recently taken, based on least significant bits of the branch.

Problems: aliasing+loops are predicted wrong twice. Inner loops are predicted wrong twice, once at entry and once at exit.

#### 2-bit Branch Prediction Buffer

Solves loop problem by only changing prediction when mispredicted twice.

## 2-12-2020: Guest lecture GPUs, Van de Braak

SIMD: single instruction multiple data, e.g. using SSE instructions.

CUDA: thread hierarchy, grid of thread blocks, each block contain a number of threads.

SIMT: single instruction multiple thread

Q: what are fundamental differences between vector SIMD and SIMT?

"Programmers convert data level parallelism (DLP) into thread level parallelism (TLP)."

Warps: groups of threads, e.g. 32 threads per warp.

"What we have learned so far:
* Pros and cons of massive threading
* How threads are executed on GPUs"

Tensor core: specialized for deep learning. Usual operation: 4x4x4 matrix-multiply and accumulate.

FP32: 1 bit sign, 8 bits exponent, 23 bits mantissa

### Roofline model

* Operational intensity (flops/byte)
* How to optimize: estimate operational intensity, find out where you are in the roofline graph to find out whether the bottleneck is in bandwidth, computation or latency.

### Recap

* GPU architecture (SIMD vs SIMT)
* GPU programming (vector vs thread)
* Performance analysis (potential bottlenecks)

## 7-12-2020: Instruction Level Parallelism part II (H&P ch. 3)

Two solutions:

* Reorder buffer
* Large renaming register file (mapping), makes speculation possible and avoids false dependencies

### Dynamic Branch Prediction (H&P 3.3+3.9)

* 1/5 or 1/6 of instructions are branch instructions, if pipeline length is around 10 the impact is high.
* CPI = CPI_base + f_branch * f_mispredict * cycles_penalty

1-bit/2-bit prediction: see above.

#### Correlating branches

Idea: behaviour of current branch is related to recently executed branches.

General scheme: (a, k, m, n)
* m bits select the table to use
* a bits to determine which BHT entry
* k bits in each BHT entry
* n-bits in each Pattern History Table entry (in practice always 2)

Two varietes:

1. GA: global history, a=0
2. PA: per address history, a>0

Tournament prediction: implements both PA and GA and a third predictor which remembers if PA or GA was last time correct, to choose between the two if they are unequal.

#### Branch Target Buffer

Where to jump to after prediction? A cache which is a kind of return stack with a bounded/limited size.

#### Avoiding branches

Expansion of 'Predicated Instructions'.

Guarding/if-conversion.

### Multiple issue

Issue multiple instructions per cycle.

Two types:

* Static, by compiler: Very Long Instruction Word
* Dynamic, by hardware: Superscalar (OoO execution, speculation, ... as seen before)

Very Long Instruction Word (VLIW), also parallel execution but not by hardware, instead using a compiler.

* Hardware much simpler
* Limitations:
  * Smart compiler
  * Loop unrolling increases code size
  * **Binary incompatibility**: need to recompile code
  * ...

Single Issue RISC vs Superscalar: code is the same, HW is different (multiple instructions in parallel)

Single Issue RISC vs VLIW: compiler compresses multiple operations into a single instruction.

## 9-12-2020: ILP architectures part III

### Explicit Parallel Instruction Computer (EPIC)

Like VLIW, but the compiler adds a control bit indicating whether the instruction can be run in parallel with the previous one. Solves the binary incompatibility problem. Looks more like a superscalar.

* 41-bit instructions, more than usual 32 because we need more registers because the compiler needs to resolve false dependencies
* 128-bit bundles of 3 instructions with 5 template bits
* possible to guard instructions using a 1-bit boolean register, using 6 bits to point to the boolean register id.
* FU pipeline latencies architectural invisible


### Architecture options

See image.

The more you make visible in the architecture (e.g. number of FUs), the less compatible it becomes.

Transport Triggered Architecture (TTA): we program the transports. Like VLIW, but you specify the movement on data over the buses.

### ILP architecture summary

* Superscalar (dynamic)
  * Binary compatible
  * Multi-issue, OoO, branch speculation
* VLIW (static)
  * Not binary compatible
  * More energy efficient
* TTA, similar to CGRA (even more static, transports also static)
  * Kind of VLIW
  * Solves remaining VLIW inefficiencies when scaling up
  * RF #ports reduction
  * Connectivity reduction


### How much ILP is there?

See graphs in slides.

Value prediction: interesting, but too costly. More used: address prediction, e.g. if you're going through an array. (Outcome of value prediction.)

GPU: should be in order of 1000s.

In general, for general programs it would be between 5 and 10, assuming a smart compiler. Only for e.g. pixel processing you would need in order of 1000s.



### What can the compiler do?

#### Code Scheduling

Typical exam question: how to reschedule a loop to avoid stalls.

Example loop iteration:

* load f0, 0(r1)
* stall
* add f4, f0, f2
* stall
* stall
* store 0(r1), f4
* addi r1, r1, 8
* stall
* bne r1, r2, begin of the loop
* stall

Could become after code rescheduling:

* load f0, 0(r1)
* addi r1, r1, 8
* add f4, f0, f2
* stall
* bne r1, r2, begin of the loop
* store **-8(r1)**, f4

#### Loop unrolling/strip mining

E.g. for 1000 iterations, combine every 4 iterations into 1. What if unknown number *n* of loop iterations? First do n mod k iterations, then unroll and do n/k iterations.

Loop unrolling requires more registers.


#### Speculative loads

(On 14-12-2020)

Load value that is only needed in one of the execution paths before the decision point, so that it is ready when the decision is taken. If the decision is not taken, the value is not used. Issues with this: the load could generate a page fault. Even worse, the load could generate an index-out-of-bounds exception, this may never happen but could happen if the branch is predicted wrongly. This is not tolerable. Solution: 2 new instructions, speculative load (sld) and speculation check (speck). sld does not generate exceptions, speck tests if an exception should be raised.


## 14-12-2020: Data Level Parallel (DLP) architectures

Examples: GPU, vector architecture, Single Instruction Multiple Data (SIMD) architecture.

Vector operations: e.g. `addv v1, v2, v3`.
Executed using either:

* Vector architecture: highly pipelined (fast clocked) function unit
* SIMD: multiple FUs acting in parallel

Only applicable for specific applications with high data-level parallelism. E.g.: matrix-oriented scientific computing, media-oriented image, video, sound processing.

SIMD is more energy efficient than MIMD

Big advantage of vector machine: low instruction bandwidth needed, e.g. for 6 instructions on 100-length vectors you need 6 instructions instead of 600.

Convoys: combine vector instructions, e.g. LV -> MULVS.D, such that parts of the vector that are already loaded can directly be sent to MUL.

### Intermezzo: Memory Banks

* Memory banks are usually single ported, 1 read/write port
* To the load-store units this memory system looks multi-ported, however except for *bank conflicts*.

Avoiding bank conflicts: example: 32 processors, each generating 2 loads and 1 stores/cycle. Processor cycle time is 2ns, SRAM cycle time is 10ns.
How many memory banks needed? Answer: think about how many accesses are needed in 15ns! -> Nr_accesses/15ns = 3 * 32 * 10/2 = 480 banks that are needed in order to make sure that bank conflicts are theoretically always possible to avoid.

### Vector Length Register / Strip Mining

* Vector length not known at compile time? Use Vector Length Register (VL)
* Use strip mining for vectors over the maximum length: first do modulo maximum vector length (MVL) to do odd-size piece, then do a loop with n/MVL iterations, each iteration handles a part of the original vector with length MVL.

### Vector Mask Registers: handling if-statements

Vector machines have vector mask registers (VM) to 'disable' elements.

### Stride (memory bank conflicts)

Stride: offset/distance between elements in an array. E.g. for matrix stored in row-major order the stride for a single row is small while the stride for a column is bigger.
For column stride the stride can be non-unit and therefore can lead to bank conflicts.

Bank conflict doesn't occur when: LCM(stride, N_banks) / Stride < bank busy time.

Example: stride=6, 16 banks, you hit bank *0, 6*, 12, 2, 8, 14, 4, 10, *0, 6* ...
You hit the same bank after LCM(6, 16)/6 = 8 cycles, if busy time is >8 you have to wait.

Other example: 8 memory banks, busy time 6 cycles, memory latency 12 cycles.
Question: how long does it take to complete a 64-element vector load with strid 1 or stride 32?

Answers:

* Stride 1: 12+64 = 76 cycles
* Stride 32: 32=4*8, every access goes to the same bank! Every access after the first has to wait 6 cycles busy time, leading to 12+1+6x63=391 cycles.

### Scatter-Gather: Indirect Vector Access

Instruction `LVI`/`SVI` for accessing specific elements from a vector using a index vector, a vector which has the indices of the elements in the original vector to use.

### Sub-word Parallelism

For SIMD architecture: divide word into multiple parts (sub-words) and perform operations on these parts in parallel.

Becoming popular (DLP in general), most new processors have sub-word parallelism SIMD instructions.

### Roofline Performance Model

Arithmetic intensity: the number of arithmetic operations per memory access

Low arithmetic intensity: memory becomes bottleneck, otherwise processor can become bottleneck (easier to improve).

---

As GPUs become more popular, CPU-GPU memory transfer becomes bottleneck, a solution: unified physical memories for CPU and GPU.

Example questions:

* Cleary explain the differences between Vector, SIMD and GPU
* Interpret the different Rooflines


## 16-12-2020: Loop Transformations

CNN: only 10% energy used for computation, 25%/43% for input/output, 22% for weights.

### Loop-Based Strength Reduction

* Replace an expensive operation, e.g. multiplication. Smart compiler could maybe replace a multiplication by an addition if it is incremented by a constant in each iteration. E.g. `i*c` becomes `T; T=T+c`.

Most compilers do this.

### Induction Variable Elimination

Induction variable `i` can be replaced by starting address and a while loop which runs until address is >= the last address.

Most compilers do this.

### Loop-Invariant Code Motion

For inner loop, if an array access doesn't change because it uses an index from the outer loop, bring that element outside of the inner loop.

Most compilers can perform this.

### Loop Unswitching

If there's a condition in the body, which doesn't depend on the iteration variable, you can split the loop to two loops, one for which the condition holds and the other one where it doesn't, so that the condition is checked upfront instead of in each iteration.

Most compilers do this.

### Iteration Reordering Transformations

For these transformation compilers need to be quite smart, only recently compilers are getting better in this.

Iteration Space: space of indices covered by the loop, e.g. variables `i` and `j`.

Matrices can be stored with row major ordering or column major ordering.

The order in which we iterate through `i` and `j` can have huge impact on:

* Memory Hierarchy (reuse of data)
* Parallelism
* Loop or control overhead

Different Reordering Transformations:

* Loop Interchange or Loop Permutation
  * E.g. `for(i){ for(j) A[j][i] = ..; }}`, assuming row major order and large N this will lead to cache miss in every iteration. Swapping the inner loop can fix this: `for(j) { for(i) { ... }}`.
  * Normal compilers will not do this automatically, although compilers are getting smarter. Can lead to huge improvements.
* Tiling Iteration Order
  * Change inner loop to a tiled version and tile it over the outer loop.

Interchange and tiling is not always possible, dependencies can prevent transformations. E.g. when `a[j][i]` updates `a[j+1][i-1]`.

Lexicographical ordering: (1,2,1) > (1,1,2), (1,4,2) < (2,1,1).
A dependency is plausible if it is lexicographically non-negative, for instance (1,-1) and (0,1) are plausible, (0,-1) and (-1,0) are not.

### Multi-Threading (not part of loop transformations)

Multiple threads share the functional units of *1* processor.
Purpose: fast context switching, full processor context switching takes long (100s/1000s of cycles).

When to switch? Fine grain: each instruction, coarse grain: when a thread is stalled, e.g. for a cache miss.

Multiprocessing: instead of running the instructions on 4 functional units and switching fine grained or coarse grained, you run two threads in parallel, one on the first 2 FUs and one on the other 2.

Simultaneous multithreading: Intel's Hyperthreading. Can be costly: need lots of virtual registers to hold the register sets of independent threads. However not too much effort needed to implement this, you need more registers but most administration is already in a superscalar, e.g. reservation table. Usually only IF, D0 and CP need to be duplicated. IF=instruction fetch, D0=decoder, CP=commit.
Performance gain is not double of one thread, it's less.

* Fine-grained
  * Usually round-robin, skipping stalled.
* Coarse-grained:
  * Not necessary to do fast thread-switching


## 4-1-2021: Multi-Threading & Multi-Processor Systems

Multi-Processing: time sharing.

Why Multi-Processing? Exploiting ILP has diminishing returns, ILP is more for single tasks only. Clock speed is leveling of from around 2003 to 3GHz, reasoning is because power consumption is then around 100W and we don't want to go higher. Power consumption is bottleneck.

Trends: since 2003 number of transistors still follows Moore's law, but not the frequency and performance per core. After 2003 we went multi-core because ILP is limited.

Multi-core architecture is based on a communication bus between all cores and I/O and memory.
How to express in language? Extensions to API, like Pthreads.

Parallelization is done by:

* Parallelizing compiler
* Yourself

Data parallelism:

* SPMD: single program multiple data
* Partition the data set, apply same function to each partition
* (Usually massive parallelism)

Function parallelism: more complex, processors need to run different functions in parallel, inbalance in processor usage.

Two philosophies: shared memory or messaging.

Shared memory characteristics:

* All memory addresses visible by all processors
* Inter-processor communication by (normal) loads and stores
* 3 problems:
  * Coherence: often processors have local copies, e.g. L1 cache.
  * Memory consistency: stores can be seen out of order by other processors.
  * Synchronization
* Types:
  * Type 1
    * Uniform memory access (UMA)
  * Type 2: distributed shared memory (DSM)
    * Memory distributed along processors
    * Non-uniform memory access (NUMA)
    * Whole (distributed) memory space is visible by all cores
    * Processors connected via direct (switched) and non-direct (multi-hop) interconnection networks.

Message passing: using communication primitives, e.g. send/receive and FIFO buffers. Note that MP (message passing) can be build on top of SM (shared memory) and vice versa, MP on top of SM is usually easier.

* Typically blocking, but may use direct memory access (DMA). DMA: simple piece of dedicated hardware used to offload message sending/receiving.
* Advantages:
  * Communication protocol enforces synchronization
* Disadvantages:
  * Prone to deadlocks
  * Block threads: no overlap of communication with computation
* Send/receive can be asynchronous

### Interconnection Networks

Network On Chip (NOC)

Types:

* Switched: multiple routers/switches/hubs in between
  * Decentralized
  * Multicast: one to many, broadcast: one to all
  * Messages broken down in packets, containing: payload, header+trailer, error correction code (ECC)
* Bus: shared medium
  * Less scalable
  * Only one can send at same time, multiple receive is possible
  * Need arbitration: make sure that only one is accessing at the same time
  * Properties:
    * Centralized
    * Low cost
    * Shared
    * Low bandwidth (single hop)

Design characteristics:

* Topologies: e.g. mesh
  * Degree: number of links from a node
  * Diameter: longest distance between two distinct nodes
  * Average distance: number of links to random destination
  * Bisection bandwidth: bandwidth across the smallest cut that divides the network into 2 equal halves.
* Routing algorithm
* Switching strategy
* Flow control and buffering
* Quality of service (QoS) guarantees
* Error handling
* Topology examples:
  * Linear array
    * Diameter: n-1, average distance ~n/3, bisection bandwidth = 1
    * Degree: 2 (number of links from a node)
  * Torus/ring: natural for algorithms that work with 1D arrays
    * Diameter: n/2
    * Average distance: ~n/4
    * Bisection bandwidth: 2
    * Degree: 2
  * Two dimensional mesh
    * Diameter: 2 * (sqrt(n)-1)
    * Bisection bandwidth: sqrt(n)
  * Two dimensional torus
    * Diameter: sqrt(n)
    * Bisection bandwidth: 2*sqrt(n)
  * Hypercubes: 0d, 1d, 2d, 3d, 4d, popular in early machines
    * Diameter: d
    * Bisection bandwidth: n/2
  * Greycode addressing: each node connected to another node differs 1 bit in its address
  * Trees
    * Diameter: log n
    * Bisection bandwidth: 1
    * Easy layout
    * Many tree algorithms
    * Fat trees avoid bisection bandwidth problem: more or wider links to the top
  * Multistage networks:
    * Butterflies
* Do not learn diameter/degree/average distance and bisection bandwidth by hard, you should be able to derive them from the topology
* Routing
  * Dimensional routing: first one direction, then the other, e.g. x-y routing
    * Avoids deadlocks


## 6-1-2021

### Deadlock

4 necessary conditions that all need to hold for deadlock, given a set of agents accessing shared resources:

* Mutual exclusion
* Hold and wait
* No preemption
* Circular wait

In NW: agents = packets, resources = physical or logical channels

Avoidance: dimensional routing: always first horizontally, then vertically. Prevents cycles, however restricts routing options.

Alternative avoidance: virtual channels, each virtual channel has its own buffer, they share the same physical link.

### Switching strategies

Circuit switching: establish a connection for the duration of the network service

Packet switching: every packet finds its own route and has a destination address

* flit = flow control unit
* phit = physical transfer unit
* A flit is made of consecutive phits
* A phit is basically the width of a link (= number of bits transferred per clock)
* A flit is a fraction of the packet

How are packets forwarded:

* store-and-forward: stored in buffers in each node
* cut-through: in pipelined fashion, such that the entire packets moves through several nodes at one time
  * virtual cut-through switching: if there's a conflict, i.e. the link to the next switch is busy, buffer locally.
  * wormhole switching: whole packet blocks, each node has enough buffering for a flit

### Latency models

End-to-end packet latency:

1. Sender Overhead
2. Time of Flight
3. Transmission Time
4. Routing Time
5. Receiver Overhead

Example calculation: packet from A to B: see packet_latency_calculation.png

### Shared memory architecture issues

* Coherence: do I see the most recent data?
* Consistency: when do I see a written value? In the right order?
* Synchronization:
  * How to protect access to shared data?
  * How to make sure 2 processes communicate correctly?


Inclusive cache: includes everything in lower caches, e.g. L3 inclusive means it includes everything from L2 and L1.

Write-back: cache dirty flag is set, write-through: writes are directly done to storage.

Coherence: writes to the same location by any two processors are seen in the same order by all processors.

Consistency: observed order of all reads and writes by the different processors. At least should be valid: if a processor writes location A followed by a write to location B, any processors that sees the new value of B should also see the new value in A(?)

Coherent caches provide:

* replication: multiple copies of the same data
* migration: movement of data

Cache coherence protocols:

* snooping: each core tracks sharing status of each block
  * Snoopy Bus, works well with a bus
  * Send all requests for data to all processors
  * Processors snoop the bus to see if they have a copy, and respond accordingly
  * Requires broadcast
  * Not so scaleable, typically used for a small number of cores
* directory based: sharing status of each block is kept in 1 location
  * Meant for bigger systems, many cores
  * Keeps track of what is being shared in one centralized place

### Snooping

Core i7 uses snooping to keep L1 and L2 caches coherent, snooping occurs at L3.

Example snooping protocol: MSI protocol.
3 states for each cache line: invalid, shared (read only, available in multiple caches), modified (also called exclusive, i.e. you may write to it). MSI: modified, shared, invalid. We need a finite state machine (FSM) to control.
When you write, you invalidate instances in other caches using the snooping bus before doing the actual write.

Snooping Write Invalidate:

* Get exclusive access, invalidate all other copies
* If two processors attempt to write simultaneously, one of them is first.
* Use the bus to perform invalidates. Only one processor has access to the bus at any time, using arbiter.

In i7, the L3 cache has 4 extra bits per cache line, one for each processor, indicating if it has a shared copy.

2 FSMs for snoopy-cache, MSI protocol. 1 FSM for putting requests on the bus, 1 FSM for reading the bus.

### Directory

Protocol agents:

* Home node (h)
* Requester node (r)
* Dirty node (d)
* Shared nodes (s)

Can use the same protocols as for snooping, i.e. MSI, MESI, MOESI.

Memory requirements:

* n processors
* m memory blocks per node
* b block size in bits

Size of total directory: m * n^2

Alternative: limited pointer scheme

## 11-1-2021: Guest lecture Maurice Peenen


## 13-1-2021

### Shared memory issues continued

* Synchronization
  * Critical section: section that can only be accessed by 1 processor at a time
  * Memory system needs to support atomic read-write action
  * Solutions:
    * Atomic exchange/swap of a register file with a memory location. Need to avoid too much snooping by first checking if 0, then do atomic exchange to change to 1, then check if we got the lock.
* Consistency
  * It could be the case that writes are delayed, e.g. when still in the write buffer and not yet in the (L1) cache (in that case it will be invalidated in other caches)
  * Sequential consistency (SC) = all processors see all memory accesses in the same order
    * Enforcing SC can be expensive
    * Might be overkill, most programs are already synchronized
    * Orderings can be relaxed, which is usually in practice:
      * Relax write after read
      * Relax write after write
      * Relax read after write *and* read after read
    * SC means that all above orderings are enforced, *not* relaxed


### Wrap-up

* Model of the 4 Cs
* SMT
* Gustafson's law: should be able to make calculations with this law
* Should know when to use what: arithmetic or weighted mean, harmonic mean, geometric mean
* MTTF & FIT: calculate them for a system containing many parts, MTTF=mean time to failure
* Power has to do with cooling, energy with battery lifetime
* P = P_static+P_dynamic, static is usually proportional to chip area, dynamic is from switching bits
* Exam question: translate C code into MIPS assembly code
* Other architecture styles: stack, load/store (risc), memory-memory, register-memory, accumulator architecture
* Branch/jump:
  * Branching: PC_next = PC+4+.....
  * Jump: PC_next = [ 2].......
* Micro Architecture Hazards
  * Structural: not enough hardware
  * Data: read after write (true), write after read (false), write after write (false)
    * False dependencies can be solved by register renaming, virtual register files
    * True dependencies can be solved using bypasses
  * Control: jumps
  * Questions:
    * How to deal with hazards, SW/HW solutions
    * Calculate t_execution impact of hazards: t_execution = N_instr * CPI * t_cycle, where CPI = CPI_base + SUM freq_{hazard_i} * cycles_penalty_{hazard_i}
* RISC design principles
  * Reduced number of instructions, avoid complex
  * Limited addressing modes
  * Large register set
  * Limited number of instruction sizes (preferably one)
  * Limited number of instruction formats
  * Memory alignment restrictions
* Multi-issue architectures
  * VLIW: by compiler, superscalar: in hardware
  * TTAs: let compiler do everything, program transports
  * Superscalar: reorder buffer, reservation stations, many more physical registers than architectural
  * VLIW: one instruction, multiple operations, typically RISC like operations
* Caches
  * Make sure to understand the bits of cache lines/entries
  * Associative cache
* Loop transformations: tiling is an important one
* Multi processors communication models:
  * Shared memory
  * Messaging passing (MPI)
* Network: shared bus or switched network
  * Need to know the 4 topology metrics (degree, ...)
* Packet latency
  * L_packet = SO+ToF+.....
