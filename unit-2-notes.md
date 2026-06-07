Unit II – Digital Logic Design, Simulation and Debugging with HDLs (8 Hours)
1. Introduction to Hardware Description Languages (HDLs)
What is an HDL?
A language to describe digital circuits at various abstraction levels (gate, register-transfer, behavioral).

Key difference from software languages:

Software: sequential execution (line by line).

HDL: concurrent and event-driven; it describes hardware where everything happens simultaneously.

Two major HDLs:

VHDL (Very High Speed IC HDL): strongly typed, verbose, used in defense/aerospace.

Verilog: C-like syntax, more common in industry. We will use Verilog.

Simulation vs. Synthesis
Simulation: Run HDL code in a simulator (ModelSim, VCS) to verify behavior. Can use $display, #delay.

Synthesis: Convert HDL to a netlist of gates/flip-flops (using Synopsys Design Compiler, Yosys). Only synthesizable constructs allowed.

Steps:

Write HDL code.

Write testbench (stimulus).

Simulate → debug waveforms.

Synthesize → place & route → FPGA/ASIC.

2. Verilog Basics (Syntax and Constructs)
Module (the basic building block)
verilog
module name (port_list);
  // port declarations
  input  a, b;
  output reg y;   // reg means can store value
  // internal wires
  wire c;
  // logic
  assign c = a & b;
  always @(*) y = ~c;
endmodule
Data Types
wire: Continuous assignment (combinational). Cannot hold value; driven by assign.

reg: Variable that can hold a value (in always blocks). Synthesized to wire or flip-flop depending on usage.

integer, time, real: For simulation only.

Number Specification
text
<size>'<base><value>
4'b1010   // 4-bit binary
8'hFF     // 8-bit hex
16'd255   // 16-bit decimal
Operators
Bitwise: &, |, ^, ~

Logical: &&, ||, !

Reduction: &a (AND all bits of a)

Arithmetic: +, -, *, / (synthesis limited)

Shift: <<, >>

Conditional: ? :

Assignments
Continuous (assign): For combinational logic. Output changes whenever RHS changes.

Procedural (inside always):

Blocking (=): Executes in order (like software). Used for combinational always blocks.

Non-blocking (<=): All RHS evaluated at beginning of timestep, then all assigned. Use for sequential logic (clocked) to avoid race conditions.

verilog
// Sequential: non-blocking
always @(posedge clk) begin
  q <= d;   // old d is sampled, then q gets it
end

// Combinational: blocking
always @(*) begin
  y = a & b;   // immediate update
end
3. Logic Gate and Circuit Design using Verilog
Basic Gates
verilog
module basic_gates (a, b, and_out, or_out, xor_out);
  input a, b;
  output and_out, or_out, xor_out;
  assign and_out = a & b;
  assign or_out  = a | b;
  assign xor_out = a ^ b;
endmodule
Multiplexer (2:1)
verilog
module mux2to1 (a, b, sel, y);
  input a, b, sel;
  output y;
  assign y = sel ? b : a;   // conditional assignment
endmodule
4:1 multiplexer using case
verilog
module mux4to1 (in, sel, y);
  input [3:0] in;
  input [1:0] sel;
  output reg y;
  always @(*) begin
    case (sel)
      2'b00: y = in[0];
      2'b01: y = in[1];
      2'b10: y = in[2];
      2'b11: y = in[3];
      default: y = 1'bx;   // don't care
    endcase
  end
endmodule
D Flip-Flop (positive edge triggered)
verilog
module dff (clk, d, q);
  input clk, d;
  output reg q;
  always @(posedge clk)
    q <= d;
endmodule
Register (multiple bits)
verilog
module reg8 (clk, d, q);
  input clk;
  input [7:0] d;
  output reg [7:0] q;
  always @(posedge clk)
    q <= d;
endmodule
Counter (4-bit)
verilog
module counter (clk, rst, count);
  input clk, rst;
  output reg [3:0] count;
  always @(posedge clk or posedge rst) begin
    if (rst)
      count <= 4'b0;
    else
      count <= count + 1;
  end
endmodule
Finite State Machine (FSM) – Mealy/Moore
verilog
// Moore machine: outputs depend only on state
module fsm_moore (clk, rst, in, out);
  input clk, rst, in;
  output reg out;
  reg [1:0] state, next_state;
  parameter S0=2'b00, S1=2'b01, S2=2'b10;
  
  // state register
  always @(posedge clk or posedge rst)
    if (rst) state <= S0;
    else state <= next_state;
  
  // next state logic
  always @(*) begin
    case (state)
      S0: next_state = in ? S1 : S0;
      S1: next_state = in ? S2 : S1;
      S2: next_state = in ? S2 : S0;
      default: next_state = S0;
    endcase
  end
  
  // output logic
  always @(*) begin
    case (state)
      S2: out = 1;
      default: out = 0;
    endcase
  end
endmodule
4. Introduction to HDL Simulation and Debugging
Testbench (stimulus)
verilog
module tb_and_gate;
  reg a, b;      // generate inputs
  wire y;        // observe output
  
  // instantiate DUT (Device Under Test)
  and_gate uut (.a(a), .b(b), .y(y));
  
  initial begin
    // apply test vectors
    $monitor("time=%0t a=%b b=%b y=%b", $time, a, b, y);
    a = 0; b = 0; #10;
    a = 0; b = 1; #10;
    a = 1; b = 0; #10;
    a = 1; b = 1; #10;
    $finish;
  end
endmodule
Simulation Commands (common)
$display("...") – print at simulation time.

$monitor(...) – print whenever any variable changes.

$time – current simulation time.

$finish – end simulation.

#10 – delay 10 time units.

Debugging with Waveforms
Simulators (ModelSim, VCS, Icarus Verilog + GTKWave) produce VCD files.

Steps: compile → run simulation → dump VCD → open in waveform viewer.

Look for: glitches, off-by-one cycles, missing resets, X propagation.

Common debugging tips
X (unknown): Usually means a register not initialized, or conflict (multiple drivers).

Z (high-impedance): Tri-state buffer not driven.

Race conditions: Use non-blocking (<=) for clocked always blocks.

Latch inference: Always block without a full assignment (missing else or default) infers a latch – usually unwanted.

5. Introduction to FPGA
FPGA (Field-Programmable Gate Array): A chip containing programmable logic blocks and interconnects.

Internal structure:

Logic blocks (LUTs): Look-up tables implementing any 4-6 input Boolean function.

Flip-flops: Registers.

Block RAM (BRAM): Small on-chip memories.

DSP slices: Dedicated multiply-accumulate units.

IO pads: Connect to outside pins.

How programming works:

Write Verilog/VHDL.

Synthesize to a netlist.

Place & route – map to FPGA resources.

Generate bitstream – configure the FPGA.

Vendors: Xilinx (now AMD), Intel (formerly Altera), Lattice.

Design flow:

text
RTL design → Simulation → Synthesis → Implementation → Bitstream → FPGA
Advantages over ASIC:

Reconfigurable (fix bugs, change functionality).

Lower non-recurring engineering cost (no mask set).

Faster prototyping.

Disadvantages:

Slower, more power-hungry, less dense than ASIC.

Applications: Hardware acceleration (Microsoft Catapult), prototyping CPUs, DSP, education