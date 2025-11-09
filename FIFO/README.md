# FIFO Buffer in Verilog

This project implements a **synchronous FIFO (First-In-First-Out)** buffer using Verilog HDL.  
It includes the design module, a testbench for simulation, and the corresponding waveform and synthesized circuit.

---

## 1. FIFO.v (Design Code)

```verilog
module FIFO #(
    parameter WIDTH = 8,
    parameter DEPTH = 16
)(
    input  wire clk,
    input  wire rst,
    input  wire wr_en,
    input  wire r_en,
    input  wire [WIDTH-1:0] din,
    output reg  [WIDTH-1:0] dout,
    output wire full,
    output wire empty
);
    reg [$clog2(DEPTH)-1:0] wr_ptr;
    reg [$clog2(DEPTH)-1:0] r_ptr;
    reg [$clog2(DEPTH):0]   count;  // one extra bit for range 0..DEPTH
    reg [WIDTH-1:0] mem [DEPTH-1:0];
    integer i;

    always @(posedge clk) begin
        if (rst) begin
            for (i = 0; i < DEPTH; i = i + 1)
                mem[i] <= 0;
            wr_ptr <= 0;
            r_ptr  <= 0;
            count  <= 0;
            dout   <= 0;
        end
        else if (!full && wr_en) begin
            mem[wr_ptr] <= din;
            wr_ptr <= wr_ptr + 1;
            count  <= count + 1;
        end
        else if (!empty && r_en) begin
            dout <= mem[r_ptr];
            r_ptr <= r_ptr + 1;
            count <= count - 1;
        end
    end

    assign full  = (count == DEPTH);
    assign empty = (count == 0);
endmodule
```
## 2. tb_FIFO.v (Testbench Code)
```verilog
`timescale 1ns/1ps
module tb_FIFO;
reg clk, rst, wr_en, r_en;
reg  [WIDTH-1:0] din;
wire [WIDTH-1:0] dout;
wire full, empty;
parameter WIDTH =8;
parameter DEPTH =16;
integer i;
FIFO uut(.clk(clk),.rst(rst),.wr_en(wr_en),.r_en(r_en),.din(din),.dout(dout),.empty(empty),.full(full));
initial clk=0;
always #5 clk=~clk;
initial begin
$dumpfile("tb_FIFO.vcd");
$dumpvars(0,tb_FIFO.uut);
for (i = 0; i < 16; i = i + 1)
    $dumpvars(1, tb_FIFO.uut.mem[i]); // dump each element manually
end
initial begin
rst = 1;
wr_en = 0;
r_en = 0;
din = 0;
#10;
rst = 0;
wr_en = 1;
repeat (16) begin
din = $random % 256;
#10;
end
wr_en = 0;
#20;
r_en = 1;
#160;
r_en = 0;
#20;
$finish;
end
endmodule
```
#3. Simulation Result
After running the simulation using Icarus Verilog and viewing in GTKWave,
the waveform shows proper FIFO operation — data is written sequentially and read back in order.
```bash
iverilog FIFO.v tb_FIFO.v
./a.out
gtkwave tb_FIFO.vcd
```
<img width="1360" height="712" alt="image" src="https://github.com/user-attachments/assets/dd9b1028-527e-474b-899d-73e94e31afb7" />

#4.Synthesized Circuit using yosys
```bash
yosys
read_verilog FIFO.v
synth
show
```
<img width="733" height="775" alt="image" src="https://github.com/user-attachments/assets/22e04241-5071-4364-8739-020404a88a00" />
(Not possible to fit the entire picture :) )




