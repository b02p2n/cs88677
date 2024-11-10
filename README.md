java c
CSE x25 Lab 4 
Welcome to CSE 125/225! Each lab is formed from one or more parts. Each part is
relatively independent and parts can normally be completed in any order. Each part will teach a concept by implementing multiple modules.
Each lab will be graded on multiple categories: correctness, style/lint, git hygiene, and demonstration. Correctness will be assessed by our autograder. Lint will be assessed by
Verilator (make lint). Style. and hygiene will be graded by the TAs.
To run the test scripts in this lab, run make test from one of the module directories.   This will run (many) tests against your solution in two simulators: Icarus Verilog, and Verilator. Both will generate waveform. files (.fst) in a directory: run/
values>/. You will need to run make extraclean after make test to clean your previous output files. You may use any waveform. viewer you would like. The codespace has Surfer installed.  It is also an excellent web-based viewer. You can also download GTKWave, which is a bit more finicky.
Each Part will have a demonstration component that must be shown to a TA or instructor for credit. We may manually grade style/lint after the assignment deadline. Style. checking and linting feels pedantic, but it is the gold standard in industry.
At any time you can run make help from inside one of the module directories to find all of the commands you can run.
When you have questions, please ask. Otherwise, happy hardware hacking!
Assignment 4 Link: https://classroom.github.com/a/J8m2GxTt
Part 1
Part 2
Part 3
Part 4
Part 1: Synchronous Memories In this lab, our goal is to demonstrate synchronous memories, which are a bit trickier to  use than asynchronous memories. However, synchronous memories are much more dense (on FPGAs).
To this end, you will write a 1R1W Synchronous Memory (Read-priority, 1 Write Port, 1     Read Port, synchronous memory). You will need to pass simulation and then verify that the code you have written synthesizes to an FPGA memory using Yosys.
You may use whatever operators and behavioral description you prefer, except you may not use always@(*). You are encouraged to reuse whatever modules see fit from previous labs  or this lab
● ram_1r1w_sync: Using behavioral SystemVerilog, create a read-priority (aka read-first) synchronous memory with 1 write port and 1 read port. It must be parameterizable in
depth (number of elements) and width.
Note: You do not need to use $readmemh for this module. Ignore the memory initialization parameter.
A portion of the grade for this part is to demonstrate that Yosys is correctly interpreting your memory as an FPGA memory. Use the following command to generate the schematic
$ make mapped.pdf
You should see a single RAM instance in the schematic. If you do not, your design is not being recognized as a memory; do not continue to the next part until this is fixed

There will be additional circuitry in the schematic, but you are specifically looking for a memory that looks like this somewhere in the schematic.
● fifo_1r1w_sync: Using behavioral SystemVerilog, create synchronous FIFO with 1 write port and 1 read port using your ram_1r1w_sync.  It must be parameterized in depth_log 2 p (log base 2 of the number of elements) and width_p (number of input/output bits).
If you get stuck on the fifo, feel free to move on to Parts 2 and 3 and come back. Don’t spend all of your time here.
There are two challenges you must address:
First, because reads are synchronous, this is slightly trickier than the previous FIFO.
Using a synchronous read pointer means data will take two cycles to be read from memory (one to update the counter, one to read), compared to the previous cycle (one to update, asynchronous read). As a hint: you should consider how your counter module must be modified to make it work for this solution, so that it doesn’t take a cycle for the RAM to get an updated address. How do Mealy machines help? 
Solving the challenge above will get ~60% of the autograder points for the FIFO.
Second: For full points, when the fifo is empty, or the read pointer trails the write pointer by one slot, it will take two cycles to get the data out of the memory. Two tips: What additional circuitry can you write that forwards the data without passing through the memory? The solution feels very similar to hazard forwarding, and sort of like an express lane on a highway.
Solving the second challenge will get 100% of the autograder points for the FIFO. test_by
Demonstration:
First, demonstrate to the TAs that your ram_1r1w_sync produces a schematic like shown above.
Second, demonstrate your working FIFO by using it to connect between audio input and output on your FPGA board. You will plug the PMOD I2S2 into PMOD Port B on your board, and then use 3.5mm cables to connect to the Audio I/O ports to/from your computer/speaker. You can set your FIFO to a larger depth in this implementation.
What is the maximum value for depth_p that you can use on your FPGA, before the toolchains fail to compile? This fifo can be much less efficient for small width_p and small depth_p values. However, it is much more space efficient for large width_p and large depth_p values. Why? Your answer may be in the schematic viewer for both labs.
Part 2: Delay Buffers You may use whatever operators and behavioral description you prefer, except you may not use always@(*). You are encouraged to reuse whatever modules see fit from previous labs  or this lab.
● regdelaybuffer: Using behavioral SystemVerilog, create a multi-bit delay buffer with a
ready/valid interface on both sides. The module must be parameterized in width_p, and delay_p. The module must consume an input data item, and provide that item
delay_p ready/valid handshakes later.
delay_p  is the number of handshakes that must pass before an input reappears. So if delay_p is 2, and the sequence is:
Input: A B C D
Where A B C and D are data that was present at the handshake, the output is: Output: x x A B C D
Where x is “don’t care” . It will be x in your simulation when simulation starts. The following is roughly the python code I use to test your module:
buf = [0] * delay_p  expected = buf[0]   buf[0] = input_data buf = np.roll(buf, 1)
Note: This is basically an elastic pipeline stage (from Lab 3) but with extra datapath registers to delay the output data. There will be one valid_o state bit – and the logic is exactly the same as the elastic stage. It is OK to provide “valid_o” when data is x/unknown for the first few cycles (I don’t check this data). With ramdelaybuffer, you are replacing these registers with a circular buffer in memory. 
Note: The following may be helpful, to show the contents of your buffer in the waveform.	
logic [width_p-1:0] buffer [delay_p:0];
generate
for (genvar i = 0; i <= delay_p; i++)
wire [width_p-1:0] temp = buffer [i];
endgenerate
● srldelaybuffer: Shift registers are so common in digital logic that modern FPGAs have specific primitives to support them (Plot twist: they’re really LUTs.) The following is the  instantiation template for an SRL16E. The documentation is here.
module SRL16E (Q, A0, A1, A2, A3, CE, CLK, D);
parameter INIT = 16'h0000;
output Q;
input A0, A1, A2, A3, CE, CLK, D;
Using the SRL16E, create a multi-bit delay buffer with a ready/valid interface on both sides. You may write behavioral SystemVerilog for the ready/valid logic, but not the   actual delay buffer.
You do not need to handle delay_p > 代 写CSE x25 Lab 4Python
代做程序编程语言16. The key sentence from the documentation is:
The inputs A3, A2, A1, and A0 select the depth of the shift register. 
● counter: Using behavioral SystemVerilog, create a counter module that rolls over at
max_val_p, and rolls under from 0 to max_val_p. All other behavior. should remain the same.
The fact that this counter is here, should be a massive hint for the next module.
● ramdelaybuffer: Using behavioral SystemVerilog, create a multi-bit delay buffer with a     ready/valid interface on both sides. The module must be parameterized in width_p, and delay_p. You may not use the regdelaybuffer, though you may use always_ff, you
must use your ram_1r1w_sync module (above).
You are highly encouraged to use the counter module that you have written. The counter should count up when data is read –  ready_i  valid_o.
Note: This looks like a FIFO, but has slightly different behavior. A fifo has latency that is dynamic and depends on how full it is, but the latency of the delay buffer is only
dependent on delay_p
Demonstration:
Show the TAs your post-synthesis schematics (make mapped.pdf) for the
regdelaybuffer and the ramdelaybuffer modules. Then answer these questions for the TA:
Which implementation of the delay buffer (above) actually synthesizes to the FPGA? If they both synthesize, which one takes less time to compile? Which one takes less area? Be prepared to    explain why to the TAs.
Part 3: Using Optimized IP 
Writing behavioral verilog is the most straightforward way to describe circuits. When
doing chip design, or even FPGA design, it is uncommon to use a verilog arithmetic operator for high-performance logic. Why? There are many different circuits for implementing arithmetic
operations. Some projects provide highly optimized implementations of Floating Point
Hardware, for example, Berkeley Hard Float. Sometimes there are options (e.g. rounding
modes) that are not inferred via Synthesis. Some IP libraries, provide high performance
modules that cannot be directly synthesized. Finally, the tools occasionally do not infer FPGA DSP blocks, or fail to correctly infer particular features.
As demonstrated in class, you can use Yosys to get parts of the solution here. The command I used in class was:
yosys -p 'synth_xilinx -top multiplier -family xc6s ;
write_verilog xilinx_synth.sv ; write_json synth.json ' multiplier .sv
● adder: Using the provided Xilinx DSP48A1 primitive, implement a pipelined addition operation with a ready/valid interface. Here is the DSP48A1 Datasheet: link. Thesimulation module itself is in the provided folder (you are encouraged to read it).   Your solution must have a latency of at least 1 cycle and use the DSP48A1 module. I recommend following this path to getting your adder working:
1. Implement a pipelined adder using the Verilog operator and former elastic module, confirming that your elastic pipeline works. (This is your behavioral model)
2. Replace the Verilog addition operator with the Xilinx DSP48A1 primitive, and determine all of the parameter/signal configurations needed to implement a combinational-only addition. (This is your intermediate model.)
3. Use the internal pipeline registers instead of your elastic pipeline module. The control logic will be similar, if not the same.
● multiplier: Using the provided Xilinx DSP48A1 primitive, implement a pipelined multiplication operation with a ready/valid interface.
Your solution must have a latency of at least 1 cycles, and use the Xilinx DSP48A1 DSP.
● multiply-accumulate: Using only the provided Xilinx DSP48A1 primitive, implement a pipelined multiply-accumulate operation with a ready/valid interface. You may use as    many or as few pipeline registers as you would like, but you will need at least one.
A multiply-accumulate is a common operation that is performed in just about …
everything. Most processor ISAs provide fused-multiply accumulation instructions. Dot products are multiply-accumulates of two vectors. Matrix multiplications are just structured dot products. Matrix multiplications are used in everything. This core is a streaming dot product.
You can use this drawing as a reference:

Your final solution must use a maximum of 1 DSP. To accomplish this, you must use the internal DSP48A1 accumulate path. See the documentation for more information: link.
Notes:
1. Like above, we recommend writing the Behavioral Verilog first, and then incrementally adding the DSP.
2. You can reuse the logic from your elastic pipeline stage, but be careful. There is one parameter in your elastic pipeline that is critical for correctness.
Demonstration
Demonstrate that your solutions use only one DSP, and no external pipeline registers.
Part 4: Fixed-Point Practice 
Have you ever heard anyone complain about how complicated IEEE 754 floating point    is? The problem is that it’s easy to use (in software), until it isn’t: List of Failures from IEEE 754. For this reason, floating point arithmetic isn’t used in many safety critical applications and signal processing. Fixed point operations are vastly less complicated than floating point operations,  require vastly less area, and are numerically stable. This part is a brief intro to fixed point
computations, in the guise of implementing pipelined arithmetic.
Fixed point arithmetic follows the same rules as normal two’s-complement arithmetic. In that sense, you already know the basics. The difference is that when two fixed-point numbers    are multiplied, the number of fractional bits increases. For example, .5 * .5, which is
representable with one fractional bit (binary : 0.1), produces .25, which needs two fractional bits to represent (binary : 0.01). So, when you multiply 0.1 and 0.1, the result will be two bits, 0.01.
I like to handle fractional bits by declaring the fractional bits in the negative range of the  bus. For example, wire [11 :-4] foo, has 12 integer bits, and 4 fractional bits. When foo is multiplied by itself, it produces 24 integer bits, and 8 fractional bits, or  [23:-8]. However, if
[-1 :-4] bus is multiplied by a  [11 :-4] bus, the result is only a  [11 :-8] bus. The widths declared here are a hint for the multiply-accumulate module. 
Here are a few good tutorials:
From Berkeley: https://inst.eecs.berkeley.edu/~cs61c/sp06/handout/fixedpt.html From UW: https://courses.cs.washington.edu/courses/cse467/08au/labs/l5/fp.pdf 
One final note, when using signed numbers in verilog it is useful to use the signed  keyword. The biggest benefit is that the number will be sign-extended when you increase the number of bits, but it also handles other edge-cases.
Here’s are a few good overviews of the signed keyword:
From Utah: Link From UW: Link 
● rgb2gray: Using SystemVerilog, implement a RGB to Grayscale Colorspace Conversion module in rgb2gray .sv. The dataflow graph below shows the computation you need to implement.

The computation for RGB to Grayscale with fixed point arithmetic does not use a divide. 
We provide it as a hint.
Important Notes:
-    Your solution must use at least two elastic pipeline stages for credit. Some tests only check for correctness. Some tests only check latency.
-    You must use fixed point arithmetic in your implementation, and determine the multiplication constants by hand. Using automatic conversions in Verilog is
error-prone, so any decimal/floating point constants (e.g. .002) will result in a 0.





         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
