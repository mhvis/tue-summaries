# Parallelization, Compilers and Platforms

## 8-2-2021: LLVM

* Intermediate representation (=bitcode or .ll human readable text): 'high level assembly language'


(To human readable IR: `clang -emit-llvm -S`)

### SSA

* Static Single Assignment (SSA): each intermediate value is assigned only once. 
* phi instruction: it selects which definition/variable to use depending on from which basic block we came from.
  * Always at the start of a basic block.

### CodeGen pipeline

* IR layer
* Selection DAG (SDAG) layer
  * Lowering, legalization and instruction selection
  * DAG combine
* Machine instructions (MI) layer: register scheduling and allocation
* Machine code (MC) streamer layer

IR level passes:

* Late optimizations: loop strength reduction (LSR), elimination of dead Bbs)
* IR level lowering: garbage collection, exception handling
* ...
* CodeGenPrepare

SDAG layer:

* DAG legalization: turn non-legal operations into legal one, e.g. Arduino doesn't support floating point operations, so needs to be done in software.

MI layer:

* Peephole optimizations, simple optimizations on small parts:
  * MachineLICM: machine loop invariant code m..
  * DCE: dead code elimination
  * CSE: ?

Register allocator: fast, greedy (default) or partial boolean quadratic problem (PBQP) solving algorithm. Post-RA passes:

* Prologue/epilogue insertion: adds local variables on the stack at the start of function, and clean them at the end
* Tail duplication: duplicate code after if-then-else inside the then and else part.
* Basic block (BB) placement to optimize hot paths

Customization: a target can insert its own passes, e.g. for hardware loop support.

### The backend

* Mixed C++ code + TableGen (DSL)
* .td files for TableGen, e.g. FooCallingConv.td
* FooISelDAGToDAG
* FooInstrInfo
* FooInstrInfo.td, defines the instruction patterns, DAG: level of input & output operands, MI: instruction encoding, assembly

### Application binary interface (ABI)

Defines how data structures are stored in memory and how they are exchanged between functions and other program parts

The ABI is hardware dependent

* Special registers
* Function arguments calling conventions
* Data storage, memory organization, storage sizes, alignment
* System calls
* Object file format, common sections, program libraries


## 11-2-2021

View as DAG: `llc -view-legalize-dags test.ll`

Instruction selection: NP-complete, usually using dynamic programming to achieve linear running time. NOLTIS: near optimal linear time instruction selection.

Debug output for instruction selection: `llc -debug-only=avr-isel test.ll`

## 22-2-2021: The life of a function part II: scheduling

Function: int test(int a, int b) { return 2*a + b + 3; }

Scheduling: change from graph/tree notation to a sequence.

Why scheduling?:

* Filling delay slots of multi-cycle operations, e.g. if LW takes multiple cycles, reschedule LW $1; ADD $1 to put independent operations in between.
* Architectures that support parallelism: VLIW or superscalar. For superscalar, is re-ordering really needed? Not necessarily. It can help to schedule instructions so that parallel instructions are inside the 'lookahead window' of the superscalar.

Scheduling algorithms:

* Basic Block Scheduling
  * Basic block = piece f code which can only be entered from the top and left at the bottom
  * Create DAG
  * Determine:
    * ASAP cycle = earliest cycle instruction can be scheduled
    * ALAP cycle = latest cycle " " " (without delay the program)
    * Slack of each operation: ALAP - ASAP
    * ASAP and ALAP are inputs for our heuristic to schedule
  * List scheduling: place each operation in first cycle with sufficient resources, use a priority function to determine the order
  * Commonly used heuristics:
    * Earliest (ASAP) start time
    * Latest (ALAP) start time
    * \# children
    * \# registers born
    * \# registers killed



2nd half: still to watch

## 25-2-2021: Part III: register allocation

Live range of a variable: execution range between definitions and uses of a variable.

Register allocation:

1. Build conflict graph
2. Coalesce
3. Attempt coloring the graph (register allocation). If not k-colorable, spill and go back to 1. Otherwise done.

Possibly additional constraints if specific (not just any) registers need to be used.

PBQP: partial boolean quadratic problem.

Register allocation in general is NP-complete. However SSA (static single assignment) graphs are acyclic.

Chordal graph: can find a perfect coloring in O(|V|^2). IN reality we often have more constraints which makes life difficult.

Register allocation results: `kill`: end of lifetime, `imp-def`/`imp-use`/`dead`/`tied0`.


## 11-3-2021

Polyhedral model: mathematical model of a loop structure

Static Control Parts (ScoP). Static control: control flow in the loop only depends on loop iterators.

LLVM: Polly:

1. Start with LLVM IR: detect SCoP parts in it.
2. Do transformations on polyhedral SCoP.
3. Code generation: different kinds, e.g. for SIMD.
4. Generate optimized LLVM IR.

Reading material which can be helpful for assignment 3: see slide.

LLVM developers' meeting tutorial session: https://www.youtube.com/watch?v=mIBUY20d8c8


## 18-3-2021: OpenMP

Hyper-threading: interleave different processes on the same processor, as opposed to multi-core where you have multiple processors.

OpenMP: abstraction for multi-threading to allow for executing on different types of machines.

Fork-join programming model: initial thread splits of (forks) and later the fork joins together.

Outlining: opposite of inlining. LLVM uses early outlining because implementing late outlining would need a lot of changes in the backend. LLVM early outlining happens in the clang frontend.

In OpenMP, the loop index (i) is private, all other variables are shared.

Nested loops: OpenML by default only makes the outer loop index private. We need to make the inner loop index private manually using `private(i)`. Or make everything private by default using `default(none)`.

False sharing: each thread has its own variable defined as an element of an array, however they are in consecutive locations in the memory and thus on the same cache line.

OpenML has a high level `reduction()` directive in cases for a reduction like a sum.

### New additions to OpenMP

Tasks: developer specify the task, OpenMP schedules the tasks in parallel.


## 22-3-2021

Architecture things