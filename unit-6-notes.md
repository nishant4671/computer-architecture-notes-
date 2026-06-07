Unit VI – Parallel Computing and Programming (4 Hours)
1. Pipelining – Data and Control Hazards
Pipeline stages (5-stage RISC)
IF → ID → EX → MEM → WB

Hazards (prevent next instruction from executing in next cycle)
Data Hazards
RAW (Read After Write) – true dependence

text
I1: add r1, r2, r3   ; writes r1
I2: sub r4, r1, r5   ; reads r1
If I2 reads r1 before I1 writes, wrong value.

WAR (Write After Write)

text
I1: sub r4, r1, r2   ; writes r4
I2: add r4, r5, r6   ; writes r4
Only matters in out-of-order (can rename).

WAW (Write After Read) – rare.

Solution to RAW: Forwarding (bypassing) – send ALU result from EX stage to next instruction’s EX stage via muxes.

Control Hazards (branch)
Next PC is unknown until EX stage. Pipeline will fetch wrong instructions.

Solutions:

Stall (flush): Wait until branch resolved (penalty 2 cycles).

Branch prediction: Guess which way branch goes.

Static: Always not taken, backwards taken.

Dynamic: 2-bit saturating counter (last two outcomes).

Branch target buffer (BTB): Cache of previous branch targets.

Structural Hazards
Two instructions need same hardware unit (e.g., ALU). Solved by duplicating resources (separate I-cache and D-cache).

2. Stalls and Bubbles
A stall (or bubble) is an idle cycle inserted in pipeline.

Example (without forwarding):

text
add r1, r2, r3
sub r4, r1, r5
Insert 2 stalls → CPI=3 for that sequence.

With forwarding: no stalls needed for many cases.

3. RISC Pipeline (classic 5-stage)
RISC-V: Simple pipeline, forwarding unit, hazard detection unit.

Hazard detection: Compare ID/EX register write address with IF/ID read addresses.

Branch handling: Compute branch target early (in ID stage) and flush mispredicted instructions.

4. Complex Pipelines (x86 P4)
Pentium 4 (NetBurst) had 20+ stages. Problems:

Long pipeline increases branch misprediction penalty (20 cycles).

High power consumption.

Modern approach: Balanced pipeline (10-14 stages in Intel Core).

5. Out-of-Order Execution (OOO)
Key idea: Execute instructions as soon as operands are ready, not necessarily in program order. Then commit results in order.

Why needed: To hide long latencies (cache miss, division) and exploit ILP.

Hardware components:

Reservation stations (RS): Buffer waiting instructions, listen to CDB.

Reorder buffer (ROB): Holds results until they can commit (in-order retirement).

Register renaming: Eliminates WAR/WAW hazards.

6. Tomasulo’s Algorithm
Historical: IBM 360/91 (1967). Basis for modern OOO.

Key features:

Reservation stations per functional unit (e.g., ALU RS, Load RS).

Common Data Bus (CDB): Broadcasts result tag + value to all RS.

Register renaming via tag mapping: Instead of physical registers, use tags (pointers to RS or ROB).

Steps for an instruction:

Issue: If RS free, read registers (if ready) or set tag (if waiting).

Execute: When all operands ready, start execution.

Write result: On CDB; update RS waiting for this tag; also update ROB.

Example (pseudocode):

text
I1: DIV.D F0, F2, F4
I2: ADD.D F10, F0, F8
I2 issues with tag for F0 (points to I1’s RS).

When I1 finishes, broadcasts result on CDB → I2 receives F0 value and can start.

7. Register Renaming
Problem with WAR/WAW:

text
I1: ADD R1, R2, R3   ; writes R1
I2: ADD R4, R1, R5   ; reads R1 (RAW – fine)
I3: ADD R1, R6, R7   ; writes R1 (WAW with I1)
Without renaming, I3 cannot execute until I1 commits (false dependency).

Renaming solution: Physical register file larger than architectural (e.g., 256 physical for 32 architectural). Map architectural R1 to different physical registers for I1 and I3.

Implementation:

Register Alias Table (RAT): Holds mapping from architectural reg to physical reg tag.

On issue: allocate new physical register for destination; update RAT.

Result: I1 and I3 can execute in parallel (no WAW).

8. Register Scoreboarding (pre-Tomasulo)
Used in CDC 6600 (1965). Centralized scoreboard tracks:

Functional unit status (busy/ready).

Register status (which FU will write it, which read it).

Difference from Tomasulo:

No CDB; no register renaming.

Instructions stall if result register is still in use (WAR/WAW hazards cause stalls).

Less efficient, but simpler.

9. Basic Compiler Techniques for Exposing ILP
Loop unrolling: Replicate loop body several times, adjust loop count. Reduces branch overhead.

Software pipelining: Overlap iterations (modulo scheduling).

Trace scheduling: For VLIW; optimize straight-line path (trace) and compensate with fix-up code.

Function inlining: Remove call overhead, expose more ILP.

Profile-guided optimization (PGO): Use runtime profiles to reorder basic blocks.

10. Vector Processors
Concept: One instruction operates on a vector of data elements (e.g., 64 floats).

Example (Cray-1): VADD V1, V2, V3 adds corresponding elements of V2 and V3, stores into V1. All elements processed in parallel or pipelined.

Advantages:

Low instruction fetch overhead.

Highly predictable memory access patterns (stride).

Modern descendant: SIMD (Single Instruction Multiple Data) instructions: Intel AVX-512 (512-bit vectors), ARM SVE, RISC-V V extension.

11. Array Processors
Also called SIMD machines (distinct from vector processors).

Many simple ALUs arranged in an array, each with its own memory.

One control unit broadcasts instruction to all ALUs.

Example: ILLIAC IV, Connection Machine CM-2.

Modern: GPUs are essentially array processors with hierarchical control and memory.

12. VLIW Architecture (Very Long Instruction Word)
Concept: Compiler packs multiple independent operations into one wide instruction (e.g., 128 bits). Hardware executes them in parallel without dynamic scheduling.

VLIW instruction (4 operations):

text
[ ALU op, ALU op, Load, Branch ]
All 4 are issued in one cycle.

Pros: Simple hardware (no OOO), low power.
Cons: Compiler must handle hazards, code bloat, binary incompatibility across implementations.

Examples: IA-64 (Itanium – failed), TI C6000 DSPs, AMD’s GCN (pre-RDNA) GPUs have VLIW-like.

13. Multithreaded Architecture
Fine-grained multithreading (cycle-by-cycle):

Switch thread each cycle. Hides long latencies (memory, FP division). Used in Sun’s Niagara.

Coarse-grained multithreading:

Switch only on long-latency events.

Simultaneous multithreading (SMT):

Multiple threads issue instructions in same cycle (Intel Hyper-Threading).

One physical core appears as two logical cores. Resources (rename registers, ROB, functional units) shared dynamically.

14. GPU Computing Architecture (NVIDIA Maxwell)
Maxwell SM (Streaming Multiprocessor) details:

128 CUDA cores (FP32 ALUs).

Warp: 32 threads, execute same instruction in SIMD fashion (actually SIMT – single instruction multiple threads).

Scheduler: 4 warp schedulers, each picks warp every cycle.

Memory:

Shared memory (96KB): L1 + scratchpad.

Registers (64KB per SM).

L2 cache (2MB shared).

Global DRAM (GDDR5 or HBM).

Key concept: Warp divergence – if threads in a warp take different branches, the warp serializes (both paths executed, threads disabled as needed).

Programming: CUDA (NVIDIA), OpenCL, HIP.

15. CUDA (Compute Unified Device Architecture)
Kernel: Function that runs on GPU, launched with many threads.

Thread hierarchy:

Grid → blocks → threads.

kernel<<<numBlocks, threadsPerBlock>>>(args)

Example: Vector addition

cuda
__global__ void vecAdd(float *a, float *b, float *c, int n) {
  int i = blockIdx.x * blockDim.x + threadIdx.x;
  if (i < n) c[i] = a[i] + b[i];
}
// Host code:
int blockSize = 256;
int gridSize = (n + blockSize - 1) / blockSize;
vecAdd<<<gridSize, blockSize>>>(d_a, d_b, d_c, n);
Memory spaces:

Global: Large, slow, all threads see.

Shared: Small, fast, within a block.

Local: Per-thread (spills).

Constant/Texture: Read-only, cached.

16. Writing a Simple Parallel Algorithm (Example: Sum of array)
Shared memory (OpenMP):

c
sum = 0;
#pragma omp parallel for reduction(+:sum)
for (i=0; i<N; i++) sum += a[i];
Distributed (MPI):

c
MPI_Comm_rank(MPI_COMM_WORLD, &rank);
MPI_Comm_size(MPI_COMM_WORLD, &size);
local_sum = partial_sum(local_data);
MPI_Reduce(&local_sum, &global_sum, 1, MPI_INT, MPI_SUM, 0, MPI_COMM_WORLD);
17. Parallel Programming Languages & Libraries
OpenMP (shared memory, C/C++/Fortran)
Uses pragmas:

c
#pragma omp parallel for schedule(static)
for (...) ...
#pragma omp critical
{ /* one thread at a time */ }
MPI (distributed memory)
Send/receive, collective operations, communicators.

Pthreads (low-level threading)
pthread_create, pthread_mutex_lock, pthread_join.

18. Amdahl’s Law
Formula: Speedup = 1 / (s + p/N)

s = serial fraction (0..1)

p = parallel fraction = 1 - s

N = number of processors

Example: s = 0.1 (10% serial), N = 16 → speedup = 1 / (0.1 + 0.9/16) = 1 / (0.1 + 0.05625) = 1 / 0.15625 = 6.4.

Limit as N → ∞: Speedup = 1/s. With 10% serial, max speedup = 10x.

Implication: To get high speedup, must parallelize almost everything.

19. Gustafson-Barsis’s Law
Assumes problem size grows with N. Formula:

Speedup = s + p·N

Where s = serial fraction on single processor (not constant, decreases with problem size).

Example: s=0.1 (10% serial) on one CPU, N=16 → speedup = 0.1 + 0.9×16 = 0.1 + 14.4 = 14.5.

Implication: For scalable problems, nearly linear speedup possible.

20. Karp-Flatt Metric
Measures the actual experimentally determined serial fraction (includes overhead).

e = (1/Speedup - 1/N) / (1 - 1/N)

Where Speedup is measured with N processors.

If e is much larger than code’s estimated serial fraction, it indicates parallel overhead (communication, synchronization, load imbalance).

21. Isoefficiency
Definition: A measure of how problem size must grow with processor count to maintain a fixed efficiency (e.g., 80%).

Mathematically: For efficiency E constant, problem size W must satisfy:

W ∝ f(N)

Example: For a parallel algorithm with overhead T_o(N, p), isoefficiency function is derived from E = W / (W + T_o). If T_o grows faster than W, efficiency drops unless W increases.

Purpose: Compare scalability of different parallel algorithms. Lower isoefficiency means more scalable.

Final Summary Table of Laws & Metrics
Law/Metric	Formula	What it tells you
Amdahl	1/(s + p/N)	Upper bound on speedup with serial part
Gustafson	s + p·N	Speedup with scaled problem
Karp-Flatt	(1/S - 1/N)/(1-1/N)	Experimental serial overhead
Isoefficiency	W = K·f(N)	How problem size must scale to keep efficiency
These notes cover every topic in your syllabus across all six units. For deeper understanding, I recommend:

Simulating a simple Verilog module (e.g., counter) with free tools (Icarus + GTKWave).

Drawing the 5-stage pipeline with forwarding paths.

Implementing a small parallel program with OpenMP or pthreads.

Using a CPU performance counter (perf) to measure CPI, cache misses.

If you need example exam questions or step-by-step solved problems (e.g., cache mapping, pipeline diagram, Tomasulo simulation), let me know.

