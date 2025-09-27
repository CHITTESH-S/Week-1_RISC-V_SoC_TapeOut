## 🌟 RISC-V SoC Tapeout – Week 1: Day 5 (Optimization in Synthesis)

> 🔎 **Understanding how Verilog constructs translate to hardware and the critical role of synthesis optimization in digital design**

---

## 🎯 Day 5 – Learning Objectives

🔎 Understand how synthesis tools **interpret RTL constructs** into hardware.

⚡ Identify **common pitfalls** that lead to unwanted latches or inefficient logic.

✅ Learn best practices for writing **synthesis-optimized if–else statements.**

🔲 Explore **correct usage and caveats of case statements.**

🔁 Differentiate between **for loops (unrolled logic)** and **for-generate loops (structural replication).**

🛡️ Apply coding techniques to **avoid synthesis mismatches** and ensure predictable hardware.

---

## 🔀 If-Case Constructs: The Foundation

### 🧩 Understanding If Statements in Hardware Context

When we write an `if` statement in Verilog, we're not just creating a conditional check - we're describing hardware behavior. The synthesis tool interprets these constructs and creates actual digital circuits.

**🔹 Priority Logic Architecture**

```verilog
// This RTL description...
always @(*) begin
    if (condition1)
        output = input1;
    else if (condition2)
        output = input2;
    else if (condition3)
        output = input3;
    else
        output = default_value;
end
```

> Creates a **priority encoder** followed by a **multiplexer chain** in hardware. The first condition has the highest priority, and the evaluation proceeds sequentially until a match is found.

**🔹 The Latch Inference Trap**

The most critical issue in synthesis occurs when we write incomplete conditional statements:

```verilog
// ❌ DANGEROUS: Creates unwanted latch
always @(*) begin
    if (en)t
        data_out = data_in;
    // Missing else clause!
end
```

**Why does this happen?** When `enable` is false, the synthesis tool doesn't know what value `data_out` should take. To maintain the previous value, it infers a **D-latch** - creating sequential behavior in what should be combinational logic.

**🔹 Proper Combinational Logic**

```verilog
// ✅ CORRECT: Complete specification
always @(*) begin
    if (enable)
        data_out = data_in;
    else
        data_out = 1'b0;  // Explicit default
end
```

**🔹 When Latches Are Acceptable**

In sequential circuits, incomplete if statements are often intentional:

```verilog
// ✅ VALID: Sequential counter logic
always @(posedge clk or posedge reset) begin
    if (reset)
        counter <= 0;
    else if (increment)
        counter <= counter + 1;
    // No else needed - register holds value
end
```
---

## 🎚️ Case Statements: Parallel Selection Logic

Case statements provide a different approach to multi-way selection, synthesizing into **multiplexers** rather than priority logic.

**🔹 Basic Multiplexer Implementation**

```verilog
always @(*) begin
    case (selector)
        2'b00: output = input0;
        2'b01: output = input1;
        2'b10: output = input2;
        2'b11: output = input3;
    endcase
end
```

> This creates a clean 4:1 multiplexer in hardware - much more efficient than equivalent if-else chains for parallel selection.

**🔹 Three Critical Pitfalls**

**Caveat 1: Incomplete Case Coverage**
```verilog
// ❌ PROBLEM: Missing cases
case (select)
    2'b00: y = a;
    2'b01: y = b;
    // What happens for 2'b10 and 2'b11?
endcase
```
**Solution:** Always include a `default` case to handle uncovered conditions.

**Caveat 2: Partial Assignment**
```verilog
// ❌ PROBLEM: Inconsistent assignments
case (mode)
    2'b00: begin
        x = data1;
        y = data2;  // Both assigned
    end
    2'b01: begin
        x = data3;  // y not assigned - latch inferred!
    end
endcase
```
**Solution:** Ensure all outputs are assigned in every case branch.

**Caveat 3: Overlapping Cases**
```verilog
// ❌ PROBLEM: Ambiguous specification
case (sel)
    2'b00: y = a;
    2'b01: y = b;
    2'b10: y = c;
    2'b1?: y = d;  // Overlaps with 2'b10!
endcase
```

> Unlike if-else statements (which have built-in priority), overlapping cases create undefined behavior.

---

## 🧪 Laboratory Investigations

### 🔬 Lab Session 1: Incomplete If Analysis

We begin our practical investigation with two critical test cases that demonstrate latch inference in action.

**🔹 Experiment: incomp_if.v**

```verilog
module incomp_if (input i0, i1, i2, output reg y);
always @(*) begin
    if(i0)
        y <= i1;
    // Deliberately incomplete
end
endmodule
```

**Laboratory Procedure:**
```bash
# RTL Simulation
iverilog incomp_if.v tb_incomp_if.v
./a.out
gtkwave tb_incomp_if.vcd
```
<div align="center">

<img width="1024" height="1024" alt="wave_incomp_if" src="https://github.com/user-attachments/assets/296f355d-393b-4fe9-8ddc-7824de7aa817" />

</div>

**Observations from RTL Simulation:** The waveform shows expected behavior - when `i0` is high, `y` follows `i1`. When `i0` is low, `y` maintains its previous value.

**Synthesis Analysis:**
```bash
# Yosys Synthesis Flow
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_if.v
synth -top incomp_if
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```
<div align="center">

<img width="1024" height="1024" alt="yosys_incomp_if" src="https://github.com/user-attachments/assets/9b801c38-4918-440b-9588-2f462597cc75" />

</div>

**Critical Discovery:** The synthesis output reveals a D-latch connected to a multiplexer. This is hardware we didn't intend to create! The latch enables the "memory" behavior observed in simulation.

**🔹 Experiment: incomp_if2.v**

```verilog
module incomp_if2 (input i0, i1, i2, i3, output reg y);
always @(*) begin
    if(i0)
        y <= i1;
    else if (i2)
        y <= i3;
    // Still incomplete - no final else
end
endmodule
```
<div align="center">

<img width="1024" height="1024" alt="wave_incomp_if2" src="https://github.com/user-attachments/assets/9f9260cf-278a-4654-a008-397a07b19288" />

</div>

**Synthesis Analysis:**
```bash
# Yosys Synthesis Flow
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_if2.v
synth -top incomp_if2
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```
<div align="center">

<img width="1024" height="1024" alt="yosys_incomp_if2" src="https://github.com/user-attachments/assets/39fc88ca-5d69-47d8-aaca-02100b733d80" />

</div>

**Key Learning:** Even with an `else if` clause, the absence of a final `else` still causes latch inference. The pattern demonstrates that **partial coverage** in combinational logic is problematic regardless of complexity.

---

### 🔬 Lab Session 2: Case Statement Exploration

Our investigation continues with systematic analysis of case statement behaviors and their synthesis implications.

**🔹 Experiment: incomp_case.v**

```verilog
module incomp_case (input i0, i1, i2, input [1:0] sel, output reg y);
always @(*) begin
    case(sel)
        2'b00: y = i0;
        2'b01: y = i1;
        // Missing 2'b10 and 2'b11 cases
    endcase
end
endmodule
```
<div align="center">

<img width="1024" height="1024" alt="wave_incomp_case" src="https://github.com/user-attachments/assets/24b4fb50-5042-4128-8d89-dbe26850a1ca" />

</div>

**Synthesis Result:** Similar to incomplete if statements, the missing cases result in latch inference. The synthesized circuit includes memory elements for the uncovered selector values.

**🔹 Experiment: comp_case.v (Complete Case)**

```verilog
module comp_case (input i0, i1, i2, input [1:0] sel, output reg y);
always @(*) begin
    case(sel)
        2'b00: y = i0;
        2'b01: y = i1;
        default: y = i2;  // Covers remaining cases
    endcase
end
endmodule
```
<div align="center">

<img width="1024" height="1024" alt="wave_comp_case" src="https://github.com/user-attachments/assets/87d43499-d4c2-46a1-89a8-be2f2163a2e7" />

</div>

**Synthesis Result:** Clean multiplexer implementation with no latches. This demonstrates the importance of complete case coverage.

**🔹 Experiment: bad_case.v (Overlapping Cases)**

```verilog
module bad_case (input i0, i1, i2, i3, input [1:0] sel, output reg y);
always @(*) begin
    case(sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b1?: y = i3;  // Wildcard overlaps with 2'b10
    endcase
end
endmodule
```
<div align="center">

<img width="1024" height="1024" alt="wave_bad_case" src="https://github.com/user-attachments/assets/4fdeffba-a26e-4be7-96b5-9690ac44269c" />

</div>

**Critical Insight:** The wildcard pattern `2'b1?` matches both `2'b10` and `2'b11`, creating ambiguity. Synthesis tools may resolve this differently than simulation, leading to **functional mismatches**.

**🔹 Experiment: partial_case_assign.v**

```verilog
module partial_case_assign (input i0, i1, i2, input [1:0] sel, 
                           output reg y, x);
always @(*) begin
    case(sel)
        2'b00: begin
            y = i0;
            x = i2;  // Both outputs assigned
        end
        2'b01: y = i1;  // Only y assigned - x gets latch!
        default: begin
            x = i1;
            y = i2;
        end
    endcase
end
endmodule
```
<div align="center">

<img width="1024" height="1024" alt="yosys_partial_assign_case" src="https://github.com/user-attachments/assets/e6fcf476-0626-4e6a-ae03-dfc32702d8c6" />

</div>

**Synthesis Analysis:** The output reveals **mixed behavior** - `y` is purely combinational, while `x` has a latch for the `2'b01` case. This creates a circuit with both combinational and sequential elements from a single always block.

---

## 🔁 Looping Constructs: Behavioral vs Structural

### 🎯 The Dual Nature of Loops in Verilog

Verilog provides two distinct looping mechanisms that serve fundamentally different purposes in hardware design.

**🔹Procedural For Loops: Behavioral Description**

Located inside `always` blocks, these loops describe **how logic should be evaluated**:

```verilog
// 32:1 Multiplexer using behavioral for loop
module mux32x1_for (input [31:0] data, input [4:0] sel, output reg out);
integer i;
always @(*) begin
    out = 0;
    for (i = 0; i < 32; i = i + 1) begin
        if (sel == i)
            out = data[i];
    end
end
endmodule
```

**Synthesis Behavior:** The loop is completely **unrolled** during synthesis, creating 32 parallel conditional assignments that resolve to a large multiplexer structure.

**🔹Generate For Loops: Structural Replication**

Located outside `always` blocks, these loops create **multiple instances of hardware**:

```verilog
// 8-bit Ripple Carry Adder using generate
module rca_8bit (input [7:0] a, b, input cin, 
                 output [7:0] sum, output cout);
wire [8:0] carry;
assign carry[0] = cin;
assign cout = carry[8];

genvar i;
generate
    for (i = 0; i < 8; i = i + 1) begin : RCA_STAGE
        full_adder fa_inst (
            .a(a[i]),
            .b(b[i]),
            .cin(carry[i]),
            .sum(sum[i]),
            .cout(carry[i+1])
        );
    end
endgenerate
endmodule
```

**Synthesis Behavior:** Creates 8 **separate instances** of the full_adder module, each representing physical hardware.

**🔹Ripple Carry Adder**

The RCA example demonstrates how generate loops enable **parameterized hardware design**:

```verilog
module rca (input [7:0] num1 , input [7:0] num2 , output [8:0] sum);
wire [7:0] int_sum;
wire [7:0]int_co;

genvar i;
generate
	for (i = 1 ; i < 8; i=i+1) begin
		fa u_fa_1 (.a(num1[i]),.b(num2[i]),.c(int_co[i-1]),.co(int_co[i]),.sum(int_sum[i]));
	end

endgenerate
fa u_fa_0 (.a(num1[0]),.b(num2[0]),.c(1'b0),.co(int_co[0]),.sum(int_sum[0]));


assign sum[7:0] = int_sum;
assign sum[8] = int_co[7];
endmodule
```

This approach enables **scalable design** - the same module can instantiate 4-bit, 8-bit, 16-bit, or any width adder simply by changing the parameter.

---

### 🔬 Lab Session 3: Loop Construct Investigation

**🔹 Experiment: mux_generate.v**

This experiment explores how generate statements can create **array-like structures**:

```verilog
module mux_generate (input i0 , input i1, input i2 , input i3 , input [1:0] sel  , output reg y);
wire [3:0] i_int;
assign i_int = {i3,i2,i1,i0};
integer k;
always @ (*)
begin
for(k = 0; k < 4; k=k+1) begin
	if(k == sel)
		y = i_int[k];
end
end
endmodule
```
<div align="center">

<img width="1024" height="1024" alt="wave_mux_generate" src="https://github.com/user-attachments/assets/408c9982-9c2f-4849-8da4-4d12a0c0effb" />

</div>

**🔹 Experiment: demux_generate.v**

**Structural approach** to the same demultiplexer functionality:

```verilog
module demux_generate (output o0 , output o1, output o2 , output o3, output o4, output o5, output o6 , output o7 , input [2:0] sel  , input i);
reg [7:0]y_int;
assign {o7,o6,o5,o4,o3,o2,o1,o0} = y_int;
integer k;
always @ (*)
begin
y_int = 8'b0;
for(k = 0; k < 8; k++) begin
	if(k == sel)
		y_int[k] = i;
end
end
endmodule
```
<div align="center">

<img width="1024" height="1024" alt="wave_demux_generate" src="https://github.com/user-attachments/assets/de155867-9c46-42b8-bf89-3f58d299ad24" />

</div>

**Comparative Analysis:** Both approaches create the same functionality, but the generate version is more **scalable** and **parameterizable**.

**🔹 Experiment: fa.v and rca.v Integration**

**Full Adder Module:**
```verilog
module fa (input a , input b , input c, output co , output sum);
	assign {co,sum}  = a + b + c ;
endmodule
```

**Ripple Carry Adder Integration:**
```verilog
module rca (input [7:0] num1 , input [7:0] num2 , output [8:0] sum);
wire [7:0] int_sum;
wire [7:0]int_co;

genvar i;
generate
	for (i = 1 ; i < 8; i=i+1) begin
		fa u_fa_1 (.a(num1[i]),.b(num2[i]),.c(int_co[i-1]),.co(int_co[i]),.sum(int_sum[i]));
	end

endgenerate
fa u_fa_0 (.a(num1[0]),.b(num2[0]),.c(1'b0),.co(int_co[0]),.sum(int_sum[0]));


assign sum[7:0] = int_sum;
assign sum[8] = int_co[7];
endmodule
```
<div align="center">

<img width="1024" height="1024" alt="wave_rca" src="https://github.com/user-attachments/assets/0388f7c1-6625-4520-9eb7-7ea759e2663d" />

</div>

---

### 🎯 Key Synthesis Insights

🔎 The journey through Day 5 reveals fundamental truths about hardware design:

📝 Verilog = Hardware Description
- Every construct maps to hardware → understanding implications ensures efficient & reliable designs.

⚡ Synthesis Tools = Faithful Translators
- They implement exactly what you write (including mistakes).
- Incomplete specs → unwanted hardware inference.

✅ Good RTL Coding = Fewer Issues

- Cover all conditions
- Use explicit default in case
- Pick constructs wisely → Prevents most synthesis problems.

🔍 Verification Beyond RTL
- RTL simulation alone ≠ enough.
- Must include synthesis verification + GLS to catch mismatches.

🏆 Mastery of Synthesis Optimization
Key milestone in digital design → enables creation of robust, optimized hardware from high-level descriptions.

---

<div align="center">

## 🌟 End Of Day - 5 of RISC-V SoC Tapeout 

</div>
