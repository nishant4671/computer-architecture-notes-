Unit IV – Processor and Memory Design (6 Hours)
1. Basic Design of a Processor
Data Path
All components that manipulate data: ALU, registers, PC, memory interface.

Control Path
Generates signals that select data path operations.

Single-cycle processor (simplest)
One instruction executes in one clock cycle.

Components:

PC: Program Counter → Instruction Memory.

Register file: 32 registers, two read ports, one write port.

ALU: Performs arithmetic/logic.

Data memory: For loads/stores.

Sign extend: For immediate values.

Path for add $r1, $r2, $r3:

Fetch instruction from I-Mem at PC.

Read registers 
r
2
,
r2,r3.

ALU adds.

Write result to $r1.

Control signals: RegWrite=1, ALUSrc=0, MemRead=0, etc.

Single-cycle problems:

Clock cycle must be long enough for slowest instruction (e.g., lw).

Long cycle → low performance.

Multi-cycle processor (one instruction over several cycles)
Each instruction takes 3–5 cycles.

Allows different latencies per instruction, faster clock.

Control unit becomes a finite state machine.

5-stage pipelined processor (classic RISC)
Stages:

IF (Instruction Fetch) – PC → I-Mem.

ID (Instruction Decode) – read register file, decode, generate control.

EX (Execute) – ALU operation, address calculation.

MEM (Memory access) – load/store to D-Mem.

WB (Write Back) – write ALU result or memory data to register file.

Pipeline registers (IF/ID, ID/EX, EX/MEM, MEM/WB) hold data between stages.

Performance: Ideal CPI = 1 (one instruction per cycle) ignoring hazards.

2. Cache Memory – Working Principle
Memory hierarchy: Registers → L1 Cache (32KB) → L2 Cache (256KB) → L3 Cache (8-32MB) → DRAM → SSD.

Why caches work: Principle of Locality

Temporal locality: If you accessed an address, you’ll likely access it again soon.

Spatial locality: If you accessed address X, you’ll likely access X+1 soon.

Cache block (line): Unit of transfer between memory and cache (e.g., 64 bytes).

On a cache miss:

Fetch entire block from lower level (DRAM).

May evict an existing block.

3. Mapping Functions
How a memory address maps to a cache location.

Assume:

Main memory size = 2^m bytes

Cache size = 2^n bytes

Block size = 2^b bytes

Number of cache lines = 2^(n-b)

Direct Mapped
Formula: Cache line = (Block address) mod (Number of lines)

Each block has exactly one possible location.

Address breakdown: Tag | Index | Block offset

Index: selects which cache line.

Tag: stored with the line; compare with tag from address to know if hit.

Offset: selects byte within block.

Advantage: Fast (only one possible location).
Disadvantage: Thrashing – two frequently used blocks map to same line, repeatedly evict each other.

Fully Associative
Any block can be placed in any cache line.

Address breakdown: Tag | Block offset (no index)

The cache must compare address tag with all tags simultaneously (associative lookup).

Advantage: No thrashing.
Disadvantage: Expensive hardware (content-addressable memory, CAM).

N-way Set Associative
Compromise. Cache divided into sets, each set holds N lines.

Formula: Set = (Block address) mod (Number of sets).

Address breakdown: Tag | Set Index | Block offset

Example: 4-way set associative, 64 sets → 256 lines total. A block can go to any of the 4 lines in its set.

Most modern L1 caches are 8-way or 16-way set associative.

4. Replacement Algorithms
When all lines in a set are full, which one to evict?

LRU (Least Recently Used): Replace the line not used for longest time.

Pros: Good hit rate.

Cons: Needs extra bits and logic; for 8-way, need 8! states.

FIFO (First In First Out): Replace oldest loaded line.

Simple, but can evict a heavily used line.

Random: Pick a victim randomly.

Surprisingly good, cheap to implement (just a pseudo-random generator).

Optimal (Belady’s): Replace the line that will be used furthest in the future (impossible to implement in hardware, used as theoretical upper bound).

5. Cache Coherence
Problem: Multicore systems: each core has its own private L1/L2 caches. If Core1 writes to address X, Core2’s cache may have an old copy.

Solution: Coherence protocols.

MESI Protocol (most common)
Each cache line has one of four states:

State	Meaning
M (Modified)	Line is dirty (different from memory); only this cache has valid copy.
E (Exclusive)	Line is clean (same as memory); only this cache has a copy.
S (Shared)	Line is clean; may exist in other caches.
I (Invalid)	Line not present or stale.
Transitions triggered by:

PrRd / PrWr (processor read/write)

BusRd / BusRdX (other core reads/writes)

Flush (write back to memory)

Example write by Core1 to line in S state:

Core1 sends BusRdX (read exclusive) to invalidate others.

All other caches set their copies to I.

Core1 changes state to M.

Snooping vs. Directory-based:

Snooping: All caches watch the bus (for small systems).

Directory-based: A centralized directory tracks which caches have each line (for large systems).

6. Examples of Cache (numerical)
Example: 32 KiB direct-mapped cache, 64-byte blocks.

Block size = 2^6 → offset = 6 bits.

Number of lines = 32KB / 64B = 512 lines → index = 9 bits (2^9=512).

Address (32-bit): Tag = 32 - (9+6) = 17 bits.

Hit/miss calculation: Access address 0x00001234.

Offset = bits [5:0]

Index = bits [14:6]

Tag = bits [31:15]

7. Modern Memory Technologies
Atomic Memory
Not mainstream; refers to memory that supports atomic read-modify-write operations directly in memory bank (emerging).

UFFO (Ultrastable Fast Flash Optimized)
Research memory combining speed of SRAM with non-volatility.

UltraRAM
Compound semiconductor (GaInAs) with ferroelectric gates; promises DRAM speed and flash non-volatility. Still in lab.

3D NAND
Flash memory with vertical stacking of cells. Current: 200+ layers (e.g., Kioxia BiCS, Samsung V-NAND).

Increases density without shrinking lithography.

Intel Optane Memory (3D XPoint)
Phase-change memory technology (non-volatile, slower than DRAM but faster than NAND).

Used as “persistent memory” (memory bus, not PCIe).

Discontinued for consumer, but concept important.

Recent Trends
HBM (High Bandwidth Memory): Stacked DRAM with TSVs (through-silicon vias), ultra-wide bus (1024 bits).

CXL (Compute Express Link): Protocol to attach memory, accelerators coherently.

PIM (Processing In Memory): Logic inside memory bank to reduce data movement.