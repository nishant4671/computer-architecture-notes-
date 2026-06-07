Unit III – Control Unit Design (6 Hours)
The control unit generates signals that tell the datapath what to do each cycle (e.g., RegWrite, MemRead, ALUSrc). Two classic approaches: hardwired and micro-programmed.

1. Hardwired Control Unit
Concept: Combinational logic (gates) directly decodes instruction opcode and generates control signals.

Block diagram:

text
Instruction Register → Decoder Logic → Control Signals
                      (with flags)
Example (single-cycle MIPS-like):

Opcode field (6 bits) determines instruction type (R-type, I-type, J-type).

For R-type: ALU operation from funct field.

Control signals: RegDst, ALUSrc, MemtoReg, RegWrite, MemRead, MemWrite, Branch, Jump.

Truth table approach: For each opcode, set control bits.

Opcode	RegWrite	ALUSrc	MemRead	MemWrite	Branch
lw	1	1	1	0	0
sw	0	1	0	1	0
add	1	0	0	0	0
beq	0	0	0	0	1
Advantages:

Fast (direct logic).

Efficient in area.

Disadvantages:

Complex for large instruction sets (CISC).

Changes require rewiring → redesign chip.

Used in: RISC-V, ARM (simpler instructions), x86 micro-ops.

2. Micro-programmed Control Unit
Concept: A "program" (microcode) stored in a ROM inside the CPU interprets machine instructions. Each machine instruction triggers a sequence of microinstructions.

Components:

Control store (ROM or PLA): Stores microinstructions.

Microprogram counter (μPC): Addresses current microinstruction.

Microinstruction decoder: Produces actual control signals.

Next-address logic: Determines next μPC (branch on condition).

Microinstruction format example (horizontal vs vertical):

Horizontal: One bit per control signal (wide, fast).

Vertical: Encoded (narrow, requires decoding, slower).

Execution steps:

Fetch machine instruction → use opcode to jump to start of microcode.

Execute a microinstruction (e.g., "send ALU result to register").

Increment μPC or branch based on condition (e.g., completion).

Example (for a lw (load word) instruction):

μPC	Microinstruction
0	MAR ← PC; Read memory
1	IR ← Memory; PC ← PC+4
2	MAR ← IR.address; Read memory
3	Register[IR.rt] ← MemoryData
Advantages:

Easy to change (update microcode ROM) – can fix bugs after chip is made.

Regular structure, good for CISC.

Disadvantages:

Slower (multiple ROM accesses per instruction).

Uses more chip area.

Historical: Intel 8086 (microcoded), many CISC CPUs.

3. Recent Trends
Hybrid Control
Hardwired for common instructions (fast path).

Microcoded for rare, complex instructions (string move, decimal adjust). Example: x86 REP MOVS.

Microcode Patching
CPUs can load microcode updates at boot (e.g., Intel Spectre/Meltdown fixes).

Stored in BIOS or OS kernel, loaded via wrmsr instruction.

Nanocode (deeper level)
Microcode has subroutines called “nanocode” – rarely used now.

Decoupled Access/Execute
Control unit separates memory access from execution to hide latency.

Modern trend: Just-in-time (JIT) translation
Not exactly control unit, but related: x86 instructions are translated to micro-ops (μops) in a trace cache.