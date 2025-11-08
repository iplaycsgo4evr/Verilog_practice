⏰ 12-Hour Digital Clock Counter (Verilog)

🧩 Overview

This project implements a 12-hour digital clock counter in Verilog HDL.

The design consists of modular counters for seconds, minutes, and hours, chained together through carry signals.

It demonstrates:

Hierarchical modular design

Carry-based cascading counters

Realistic time-keeping logic (1–12 hours)

⚙️ Design Hierarchy

Top-Level Module: hour

This is the main module that integrates all submodules (mod59, mod10, and mod6)

to form a complete 12-hour digital clock.

Hour count (1–12)

🧠 Working Principle

Seconds counter (mod59) increments with every 1 Hz pulse.

When it reaches 59 → 00, it outputs a carry pulse (sec_carry_out).

Minutes counter (mod59) increments when the seconds counter rolls over.

When it reaches 59 → 00, it generates another carry (min_carry_out).

Hours counter increments every time the minute counter rolls over.

It counts from 1 → 12 and then wraps back to 1.

🧾 Verilog Source Code

This section shows the complete, concatenated source code for the design modules.

hour.v (Top-Level)

module hour(
    input wire clk,
    input wire rst,
    input wire enable_1hz, // Main 1-second tick

    // Outputs
    output wire [3:0] sec_ones,
    output wire [2:0] sec_tens,
    output wire [3:0] min_ones,
    output wire [2:0] min_tens,
    output wire [4:0] hours      // Will count 1-12
);

    // Wires for carries
    wire sec_carry_out; // Tick once per minute
    wire min_carry_out; // Tick once per hour

    // 1. Seconds Counter (mod59)
    mod59 seconds_counter (
        .clk(clk),
        .rst(rst),
        .enable(enable_1hz),
        .tens(sec_tens),
        .ones(sec_ones),
        .hour_carry(sec_carry_out)
    );

    // 2. Minutes Counter (mod59)
    mod59 minutes_counter (
        .clk(clk),
        .rst(rst),
        .enable(sec_carry_out),
        .tens(min_tens),
        .ones(min_ones),
        .hour_carry(min_carry_out)
    );

    // 3. Hours Counter (1–12)
    reg [4:0] hours_reg;
    assign hours = hours_reg;

    always @(posedge clk) begin
        if (rst) begin
            hours_reg <= 5'd1; // Start from 1
        end
        else if (min_carry_out) begin
            if (hours_reg == 5'd12)
                hours_reg <= 5'd1;
            else
                hours_reg <= hours_reg + 1'b1;
        end
    end
endmodule


Submodules (mod59.v, mod10.v, mod6.v)

module mod59(
    input wire clk,
    input wire rst,
    input wire enable,
    output wire [2:0] tens,
    output wire [3:0] ones,
    output wire hour_carry
);

    wire carry_out;

    mod10 ones_counter (
        .clk(clk),
        .rst(rst),
        .count(ones),
        .carry(carry_out),
        .enable(enable)
    );

    mod6 tens_counter (
        .clk(clk),
        .rst(rst),
        .count(tens),
        .enable(carry_out)
    );

    assign hour_carry = (tens == 3'd5) && carry_out;
endmodule

module mod10(
    input  wire clk,
    input  wire rst,
    input  wire enable,
    output reg  [3:0] count, 
    output wire carry
);

    always @(posedge clk) begin
        if (rst)
            count <= 4'd0;
        else if (enable) begin
            if (count == 4'd9)
                count <= 4'd0;
            else
                count <= count + 1'b1;
        end
    end

    assign carry = (count == 4'd9) && enable;
endmodule

module mod6(
    input wire clk,
    input wire rst,
    input wire enable,
    output reg [2:0] count
);
    always @(posedge clk) begin
        if (rst)
            count <= 3'd0;
        else if (enable) begin
            if (count == 3'd5)
                count <= 3'd0;
            else
                count <= count + 1'b1;
        end
    end
endmodule


🧰 Tools Used

Yosys — Synthesis and schematic generation

Icarus Verilog (iverilog) — Simulation

GTKWave — Waveform visualization

🧪 How to Run

(Assumes you have all modules in separate files: hour.v, mod59.v, mod10.v, mod6.v, and tb_hour.v)

Simulation

# 1. Compile all design files and the testbench
iverilog  hour.v mod59.v mod10.v mod6.v tb_hour.v

# 2. Run the simulation
./a.out

This will print the H:M:S output to the console and generate tb_hour.vcd.

# 3. (Optional) View the waveforms
gtkwave tb_hour.vcd


Synthesis & Visualization (with Yosys)

# 1. Start Yosys in interactive mode
yosys


# 2. Load all design files (NOT the testbench)
read_verilog hour.v mod59.v mod10.v mod6.v

# 3. Set 'hour' as the top module
hierarchy -top hour

# 4. Run synthesis
synth

# 5. Show the synthesized circuit
show hour

<img width="1514" height="694" alt="image" src="https://github.com/user-attachments/assets/e83aaec5-eed2-4505-8b35-1cf27506dcf1" />

