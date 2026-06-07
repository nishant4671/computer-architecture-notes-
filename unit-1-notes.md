Unit I – Recent Advances (4 Hours)
1. Technology Trends in Computer Architecture
Historical Trend: Moore’s Law
Observation (1965, Gordon Moore): Number of transistors on an integrated circuit doubles every ~2 years.

Consequence: Every 2 years, chips became cheaper, faster, and more power-efficient.

End of Dennard Scaling (≈2006): Transistors stopped becoming more energy-efficient. Clock speeds stalled around 3-4 GHz due to heat.

Current Trends (Post-Moore)
Heterogeneous Computing: Different core types (e.g., big.LITTLE: performance cores + efficiency cores).

Domain-Specific Architectures: Hardware designed for AI (TPU), networking, cryptography.

3D Stacking: Stacking dies vertically (e.g., HBM memory, AMD’s 3D V-Cache).

Near-Memory Computing: Processing inside or very close to memory to reduce data movement.

2. Performance Metrics
Key Metrics
Metric	Definition	Unit
Latency (Response Time)	Time to complete a single task	seconds, cycles
Throughput (Bandwidth)	Number of tasks per unit time	tasks/sec, bytes/sec
CPU Time	Time CPU spends on a program	seconds
CPI (Cycles Per Instruction)	Average cycles per instruction	cycles/instr
IPC (Instructions Per Cycle)	Inverse of CPI	instr/cycle
Performance Equation
text
CPU Time = Instruction Count × CPI × Clock Cycle Time
Instruction Count: Program length (compiler + ISA).

CPI: Architecture + microarchitecture (pipelining, caching).

Clock Cycle Time: Determined by technology and design.

Speedup Formula
text
Speedup = Old Execution Time / New Execution Time
If speedup > 1, improvement.

If speedup = 2, twice as fast.

Benchmarks
SPEC (Standard Performance Evaluation Corporation): Industry-standard suites (SPECint, SPECfp).

Toy benchmarks: Dhrystone (integer), Whetstone (floating point) — less realistic.

3. Improving Performance
Three Levers (from performance equation)
a. Reduce Instruction Count
Better compilers (optimization flags: -O2, -O3)

New instructions (e.g., AES encryption, vector instructions like AVX)

RISC-V compressed instructions (16-bit encoding)

b. Reduce CPI
Pipelining (overlap instruction execution)

Superscalar (issue multiple instructions per cycle)

Out-of-order execution (execute instructions as operands ready)

Branch prediction (reduce control hazard stalls)

c. Reduce Clock Cycle Time
Improved transistor technology (FinFET, GAAFET)

Reduced wire delays (shorter paths, better layout)

But cycle time reduction is limited by power/heat (dark silicon)

Power–Performance Trade-off
Dynamic power: P ∝ C × V² × f (voltage scaling is powerful)

Dark silicon: Large portion of chip must be powered off due to thermal limits.

4. Moore’s Law – Deeper Look
What it really says
Transistor density doubles every 2 years. It does not say performance doubles automatically.

Why it’s slowing
Lithography limits (currently 3nm, 2nm, but cost skyrockets)

Leakage current increases at small sizes

Voltage can’t scale below ~0.7V (transistors won’t switch reliably)

Post-Moore strategies
More than Moore: Integration of analog, sensors, MEMS.

Advanced packaging: Chiplets (AMD’s EPYC, Intel’s EMIB).

New devices: TFETs, spintronics, photonics.

5. Cluster Computing
Definition: A group of independent computers (nodes) connected by a high-speed network, working together as a single system.

Characteristics:

Each node runs its own OS (Linux, Windows).

Nodes communicate via message passing (MPI).

Head node manages job scheduling (SLURM, PBS).

Advantages:

Cheap: built from commodity hardware.

Scalable: add nodes to increase throughput.

Disadvantages:

Programming is harder (distributed memory model).

Fault tolerance requires checkpointing.

Example: Beowulf cluster (low-budget supercomputer).

6. Cloud Computing
Definition: Delivery of computing resources (servers, storage, databases) over the internet on a pay-as-you-go basis.

Service Models:

IaaS: Virtual machines (AWS EC2, Google Compute Engine).

PaaS: Platform to deploy apps (Heroku, Google App Engine).

SaaS: Software as a service (Gmail, Office 365).

Key Enablers:

Virtualization: One physical machine runs many VMs (hypervisor: KVM, Xen).

Containerization: Lightweight isolation (Docker, Kubernetes).

Relationship to cluster computing: Cloud providers run huge clusters and virtualize them for customers.

7. Quantum Computers
Basic concepts (no physics degree needed):

Qubit: Can be 0, 1, or superposition (both at once).

Entanglement: Qubits can be correlated; measuring one affects the other.

Quantum gates: Operate on qubits (Hadamard, CNOT).

What they’re good for:

Factoring large numbers (Shor’s algorithm) – breaks RSA.

Searching unsorted databases (Grover’s algorithm) – quadratic speedup.

Simulating quantum chemistry.

What they’re NOT good for:

General-purpose computing (email, web browsing).

Tasks with little parallelism.

Current state: Noisy Intermediate-Scale Quantum (NISQ) era – tens to hundreds of qubits, high error rates.

Hardware approaches:

Superconducting qubits (Google, IBM)

Trapped ions (IonQ)

Silicon spin qubits (Intel)

8. Hardware Support for Operating Systems
Memory Management Unit (MMU)
Translates virtual addresses (used by programs) to physical addresses (actual RAM).

Uses page tables (OS-managed) and TLB (hardware cache for translations).

Privilege Levels (Rings)
Ring 0 (Kernel mode): Can execute privileged instructions, access all memory.

Ring 3 (User mode): Restricted; cannot access I/O or other processes’ memory.

Switching: syscall instruction (fast) or int (interrupt). Saves state to kernel stack.

Timer Interrupts
Hardware timer generates interrupt every few milliseconds.

Forces control back to OS scheduler (prevents infinite loops).

Synchronization Primitives (hardware support)
Test-and-set: Atomic read-modify-write instruction.

Compare-and-swap (CAS): if (mem == old) { mem = new; return true; } – atomic.

Used to implement mutexes, spinlocks.

I/O and DMA
DMA (Direct Memory Access): Device transfers data to/from memory without CPU intervention.

CPU sets up transfer, then continues. Interrupt signals completion.

9. Computer Architecture for AI/ML
CPU – General purpose
Few large cores (4-16), deep out-of-order pipelines.

Good for serial, control-heavy code.

Poor matrix multiply performance.

GPU – Graphics Processing Unit
Key idea: Massively parallel SIMD (Single Instruction, Multiple Threads).

Architecture (NVIDIA example):

SM (Streaming Multiprocessor): Contains many CUDA cores (e.g., 64).

Warp: 32 threads executing same instruction in lockstep.

Memory: Global (off-chip), shared (on-chip, programmer-managed), registers.

Why good for AI: Neural networks are mostly matrix multiplies and convolutions – trivially parallel.

TPU – Tensor Processing Unit (Google)
ASIC (Application-Specific IC), not a general-purpose processor.

Systolic array: A grid of multiply-accumulate units (MACs) that pass data in a wave-like motion.

Example: 256×256 array → 65k MACs per cycle.

Reduced control logic: No branch prediction, no out-of-order execution.

Software: TensorFlow compiled to TPU instructions.

Comparison
Feature	CPU	GPU	TPU
Core count	4–64	1000s	1 big systolic array
Control logic	heavy	light	almost none
Memory bandwidth	moderate	very high	extremely high
Best for	serial, branching	data-parallel	matrix ops (DNN)
10. Multicore Architecture
Definition: Multiple independent CPU cores on a single die.

Why multicore?

Single-core performance plateaued (power wall).

Cheaper to add more cores than to make one core faster.

Types:

Homogeneous: All cores identical (e.g., 8x ARM Cortex-A78).

Heterogeneous: Different cores (ARM big.LITTLE: 4 performance + 4 efficiency cores).

Challenges:

Cache coherence: Cores have private caches; need protocols (MESI) to keep data consistent.

Memory bandwidth: All cores share DRAM; can saturate.

Software parallelism: Not all programs can use many cores (Amdahl’s law).

Example multicore processors:

Intel Core i9 (10–24 cores)

AMD Ryzen 9 (16 cores)

Apple M-series (8–16 cores, heterogeneous)