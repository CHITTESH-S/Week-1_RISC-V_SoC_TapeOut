## 🌟 RISC-V SoC Tapeout – Week 1: Day 3 (Combinational and Sequential Optimizations)

Welcome to **Day 3** of the RISC-V SoC Tapeout journey!

> **Focus:** Practical techniques to improve combinational logic area/timing and sequential (register) optimization for robust, high-performance RTL → netlist results.

---

💡 We focus on three main areas:

⚡  1. Combinational Logic Optimization – Simplify logic, propagate constants, reduce gates.

🔁 2. Sequential Logic Optimization – Optimize FFs, retiming, state pruning.

🗑 3. Unused Outputs Optimization – Remove redundant sequential nodes.

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
module opt_check(input a, input b, output y);
  assign y = a ? b : 0;
endmodule
```

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

<img width="1024" height="1024" alt="opt_check" src="https://github.com/user-attachments/assets/c09adbb5-fd1e-4681-ab17-56c920c7bb1c" />

</div>

### 2️⃣ Boolean Logic Optimization / K-Map Simplification
- Apply Boolean algebra identities.
- Example:
```verilog
assign y = a ? ( b ? c : ( c ? a : 0 ) ) : !c ) -> y = a ⊕ c
```

### 🖥 Yosys Commands

```yosys
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check2.v
synth -top opt_check2
opt_clean -purge
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

<div align="center">

<img width="1024" height="1024" alt="opt_check2" src="https://github.com/user-attachments/assets/0aaf173d-5c98-4986-9d3d-3edf238b6393" />

</div>

---

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

<div align="center">

<img width="1024" height="1024" alt="wave_const1" src="https://github.com/user-attachments/assets/ec7d79a4-b4c2-4c1b-ada8-0ba576d022b3" />

</div>

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

<img width="1024" height="1024" alt="yosys_const1" src="https://github.com/user-attachments/assets/d5c1b20d-779a-4986-9b5c-d682b06273aa" />

</div>

---

## 🗑 Section 3 – Unused Outputs Optimization

### Why:

- FF outputs or wires not used consume area & dynamic power.

### Benefits:

🏭 Reduces chip area

⚡ Saves power

🔍 Simplifies netlist readability

### 💻 Example: Counter (Case - 1) 

```verilog
reg [2:0] count;
assign q = count[0];

always @(posedge clk or posedge reset) begin
  if(reset)
    count <= 3'b000;
  else
    count <= count + 1;
end
```

<div align="center">

<img width="1024" height="1024" alt="count_opt_case1" src="https://github.com/user-attachments/assets/5eb8dc40-3507-48ad-a9dc-ce0433997534" />

</div>

### 💻 Example: Counter (Case - 2)

```verilog
reg [2:0] count;
assign q = (count[2:0] == 3'b100);

always @(posedge clk or posedge reset) begin
  if(reset)
    count <= 3'b000;
  else
    count <= count + 1;
end
```

<div align="center">

<img width="1024" height="1024" alt="count_opt_case2" src="https://github.com/user-attachments/assets/47195a6d-6ca7-4522-8f5e-9027abc35a9e" />

</div>

---

## ✅ Summary

| Section           | Key Points                                                         |
| ----------------- | ------------------------------------------------------------------ |
| ⚡ Combinational   | Constant propagation, Boolean simplification, hierarchy flattening |
| 🔁 Sequential     | Constant FF removal, retiming, cloning, FSM state optimization     |
| 🗑 Unused Outputs | Remove unconnected FFs/wires to save area & power                  |

---

<div align="center">
  
## 🌟 End Of Day - 3 of RISC-V SoC Tapeout

</div>
