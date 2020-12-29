
# Embedded Computer Architecture (2020-Q2)


* CPI: cycles per instruction
* t_exec = N_instr * CPI * t_cycle


## 25-11-2020

Virtual memory:

* Processor architecture: user mode vs supervisory mode
* Translation Lookaside Buffer: a cache for address translations.
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

### Out-of-Order architectures

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


