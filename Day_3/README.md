## 🌟 RISC-V SoC Tapeout – Week 1: Day 3 (Combinational and Sequential Optimizations)

Welcome to **Day 3** of the RISC-V SoC Tapeout journey!

> **Focus:** Practical techniques to improve combinational logic area/timing and sequential (register) optimization for robust, high-performance RTL → netlist results.

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

### 2️⃣ Boolean Logic Optimization
- Apply Boolean algebra identities.
- Example:
```verilog
assign y = a ? ( b ? c : ( c ? a : 0 ) ) : !c ) -> y = a ⊕ c
```
---

## 🔁 Sequential Optimizations

### 1️⃣ Constant Propagation in Sequential Logic
- Remove registers driven by constants.
- Example: a flop that always loads `1’b0` can be pruned.

### 2️⃣ State Optimization
- Reduce FSM state encoding.
- Merge equivalent/unused states to minimize flip-flops.

### 3️⃣ Sequential Logic Cloning
- Duplicate registers feeding multiple far-apart loads.
- ✅ Reduces routing delay at cost of area.

### 4️⃣ Retiming
- Move registers across combinational paths.
- ✅ Balances path delays, improves Fmax.

---
