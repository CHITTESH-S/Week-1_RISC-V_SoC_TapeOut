## 🌟 RISC-V SoC Tapeout – Week 1: Day 3 (Combinational and Sequential Optimizations)

Welcome to **Day 3** of the RISC-V SoC Tapeout journey!

> **Focus:** Practical techniques to improve combinational logic area/timing and sequential (register) optimization for robust, high-performance RTL → netlist results.

---

💡 We focus on three main areas:

- ⚡ Combinational Logic Optimization – Simplify logic, propagate constants, reduce gates.

- 🔁 Sequential Logic Optimization – Optimize FFs, retiming, state pruning.

- 🗑 Unused Outputs Optimization – Remove redundant sequential nodes.

---

## 🎯 Objectives

✅ Understand common combinational optimizations (Constant Propogation, Boolean Logic Optimization, etc).

✅ Learn sequential optimizations: Constant Propogation, State Optimization, Sequential Logic Cloning, Retiming, etc.

✅ Adopt Verilog coding patterns that are synthesis‑friendly and produce clean netlists.

✅ Run Yosys flows to compare unoptimized vs optimized results and interpret `stat`/`show` outputs.

---

## 🧠 Combinational Optimizations

### 1️⃣ Constant Propagation
- Replace constant-driven logic with fixed values.
- Example:
```verilog
assign y = a & 1'b0; // becomes y = 1'b0
assign z = b | 1'b1; // becomes z = 1'b1
```
- ✅ Simplifies netlist and reduces area.

### 💻 Example: Constant Propagation

```verilog
module opt_check1(input a, input b, output y);
  assign y = a ? b : 0;
endmodule
```

<div align="center">



</div>

### 🖥 Yosys Commands

```yosys
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check.v
synth -top opt_check
opt_clean -purge
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

<div align="center">



</div>

### 2️⃣ Boolean Logic Optimization / K-Map Simplification
- Apply Boolean algebra identities.
- Example:
```verilog
assign y = a ? ( b ? c : ( c ? a : 0 ) ) : !c ) -> y = a ⊕ c
```
---

<div align="center">



</div>

## 🔁 Sequential Optimizations

### 1️⃣ Constant Propagation in Sequential Logic
- Remove registers driven by constants.
- Example: a flop that always loads `1’b0` can be pruned.

### 💻 Example: Constant FF

```verilog
always @(posedge clk or posedge reset) begin
    if(reset)
        q <= 0;
    else
        q <= 1;
end
```

### 2️⃣ State Optimization
- Reduce FSM state encoding.
- Merge equivalent/unused states to minimize flip-flops.

### 3️⃣ Sequential Logic Cloning
- Duplicate registers feeding multiple far-apart loads.
- ✅ Reduces routing delay at cost of area.

### 4️⃣ Retiming
- Move registers across combinational paths.
- ✅ Balances path delays, improves Fmax.

```yosys
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const.v
synth -top dff_const
dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show

```

<div align="center">



</div>

---
