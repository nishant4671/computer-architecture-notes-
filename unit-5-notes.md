Unit V – Multiprocessor Architectures (4 Hours)
1. Shared Memory Multiprocessor
Definition: All processors share a single address space. Communication via shared variables.

UMA (Uniform Memory Access)
All processors access all memory at equal speed.

Usually a single bus or crossbar connecting processors and memory modules.

Limitation: Bus bandwidth limits scalability (≤ 32 processors).

NUMA (Non-Uniform Memory Access)
Memory is physically distributed (each CPU has local memory).

Accessing local memory is fast; remote (other CPU’s memory) is slower (through interconnect).

Example: AMD EPYC (each chiplet has DRAM controllers). Linux numactl binds processes to local memory.

Cache coherence in NUMA: Directory-based protocol (e.g., AMD’s MOESI with probe filter).

2. Distributed Memory Multiprocessor
Definition: Each processor has its own private memory. No shared address space.

Communication: Explicit message passing (send/receive).

Hardware:

Processors + memory + network interface.

Network: InfiniBand, Ethernet, Myrinet.

Advantages: Scales to millions of cores (Blue Gene). No coherence overhead.

Disadvantages: Programmer manages data distribution and communication.

3. Message-Passing Multiprocessors
Programming model: MPI (Message Passing Interface).

Key operations:

MPI_Send(&buf, count, datatype, dest, tag, comm)

MPI_Recv(&buf, count, datatype, source, tag, comm, &status)

MPI_Bcast (broadcast), MPI_Reduce (sum/min/max across ranks).

Hardware support:

RDMA (Remote Direct Memory Access): Network interface can write directly to remote memory without CPU intervention.

Collective offload: Special hardware for barrier, broadcast, all-reduce.

Example supercomputer: Fugaku (A64FX) uses MPI+OpenMP.

4. Dataflow Machine Architecture
Philosophy: Instead of control-driven (program counter), instructions execute when all operands are available (data-driven).

Node (instruction) format: Operation, operands (or pointers), destination.

Execution model:

An instruction node has input arcs.

When all input arcs have tokens (values), instruction fires.

Produces output tokens on its output arcs.

Advantages: Inherent parallelism; no program counter; no control hazards.

Disadvantages:

Huge overhead for token matching.

Limited locality.

Never became mainstream. Influence: Out-of-order execution (reservation stations) and dataflow in GPUs (NVIDIA’s “dataflow” scheduling).

5. Supercomputer Architecture
Definition: A computer at the frontline of processing capacity, especially for scientific computing.

Top500 list: Ranks supercomputers by LINPACK (solves dense linear equations) FLOPS.

Two dominant architectures today
a. Massive clusters of standard CPUs + fast interconnect
Example: Fugaku (Japan) – 7.6 million ARM cores, Tofu interconnect.

Programming: MPI only.

b. Heterogeneous: CPUs + Accelerators (GPUs, TPUs, FPGAs)
Example: Summit (ORNL) – IBM Power9 + NVIDIA V100 GPUs.

Frontier (ORNL) – EPYC + AMD Instinct GPUs; first exascale (1.1 EFLOPS).

Key features of supercomputers:
Low-latency, high-bandwidth network (e.g., Cray Slingshot, InfiniBand).

Parallel file system (Lustre, GPFS).

Liquid cooling (for high power density).

Fault tolerance: Checkpoint/restart.