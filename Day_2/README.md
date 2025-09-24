## 🌟 RISC-V SoC Tapeout – Week 1: Day 2 (Timing Libraries, Synthesis Approaches & Flip-Flop Coding Styles)

Welcome to **Day 2** of the RISC-V SoC Tapeout journey!

Here, we focus on **Introduction to dot lib, Hierarchical and flatten Synthesis and Various Flop Coding Styles and Optimization**.

---

## 🖥️ 1️⃣ Introduction to .lib

### What is a `.lib` (Liberty) file?

A Liberty file is a text description of a standard cell library that contains: logical function names, timing arcs, setup/hold/clock‑to‑Q, leakage/dynamic power numbers, physical area, and operating conditions for a given PVT corner. Tools use it to map RTL primitives to real silicon cells and for timing analysis.

### Liberty file structure — key fields

- `library()` — top-level name and attributes
- `cell()` — describes each cell (e.g., `and2`, `or2`, `dff`)
- `area` — physical area in µm²
- `pin()` / `pg_pin()` — pin & power definitions
- `leakage_power()` — leakage depending on input vector
- `timing()` — timing arcs, delays (often `table_lookup` entries)
- `operating_conditions {}` — PVT corner described by process, voltage, temperature

**Units**: `time_unit`, `voltage_unit`, `capacitive_load_unit`

> Note: delays are commonly represented as tables (input transition vs output load). STA uses these multi‑dimensional lookup tables to compute arrival/required times.

<div align="center">

<img width="1024" height="1024" alt="vim" src="https://github.com/user-attachments/assets/f7cc2dd1-5cb9-47d8-af76-975ae71cc8be" />

</div>

### PVT corners & cell flavors

* **PVT** = Process (Slow/Typical/Fast, e.g., SS/TT/FF), Voltage (e.g., 1.8V), Temperature (e.g., 25°C)
* Libraries often provide several drive strengths or flavors of the same logical cell (different transistor widths). These trade speed for area/power.

### Verilog models vs `.lib` view

* `.v` files provide *functional/behavioral* or *structural* models for simulation (no timing unless annotated), while `.lib` gives the *timing/power/area* information used by synthesis and STA. Both must match logically.

### Fast vs Slow cells — tradeoffs

* Fast: lower delay, higher area and dynamic/leakage power.
* Slow: higher delay, smaller area, lower power.
* Use fast cells only in timing‑critical paths; use slow cells elsewhere.

<div align="center">

<img width="1024" height="1024" alt="cell" src="https://github.com/user-attachments/assets/d198f45a-ae4b-485a-853d-b2cc78821a42" />

</div>

---

## ⚡ 2️⃣ Hierarchical vs Flat synthesis — tradeoffs

## Hierarchical synthesis

- preserves module boundaries.

- Pros: readability, IP reuse, faster incremental re‑synthesis.

- Cons: limited optimization across module boundaries.

## Flat synthesis

- inlines modules to a single netlist.

- Pros: global optimizations, potentially better area/timing.

- Cons: harder to debug, larger memory/time for synthesis.

## 🛠️ Lab Files & Example RTL

**`../multiple_modules.v`**

```verilog
module sub_module2 (input a, input b, output y);
  assign y = a | b;
endmodule

module sub_module1 (input a, input b, output y);
  assign y = a & b;
endmodule

module multiple_modules (input a, input b, input c, output y);
  wire net1;
  sub_module1 u1(.a(a), .b(b), .y(net1)); // net1 = a & b
  sub_module2 u2(.a(net1), .b(c), .y(y)); // y = net1 | c = (a & b) | c
endmodule
```

<div align="center">

Logic: `y = (a & b) | c`

</div>

So, the top-level module (`multiple_modules`) is composed of two submodules:

- `sub_module1` → AND gate
- `sub_module2` → OR gate

**Sky130 library & models**

* `lib/sky130_fd_sc_hd__tt_025C_1v80.lib` (Liberty)
* `verilog_model/sky130_fd_sc_hd.v` (functional models)
* `verilog_model/primitives.v`

---

## 🛠️ Lab: Hierarchical Synthesis (step‑by‑step)

**Objective:** Synthesize `multiple_modules` and preserve hierarchy.

**Script `scripts/synth_hier.ys`**

```yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ../rtl/multiple_modules.v
synth -top multiple_modules
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show -prefix ../results/multiple_modules_hier
write_verilog ../results/multiple_modules_hier.v
```

<div align="center">

<img width="1024" height="1024" alt="hier_show" src="https://github.com/user-attachments/assets/43a97a79-4665-4249-a467-fe7e61e329de" />

</div>

**Run:** `yosys -s scripts/synth_hier.ys`

**Expected output:**

* `results/multiple_modules_hier.v` keeps `sub_module1` and `sub_module2` as modules instantiated by top module.
* Each submodule will be mapped to cells (e.g., `sky130_fd_sc_hd__and2_*`, `sky130_fd_sc_hd__or2_*`).

**Checks:**

* Run `stat` in the Yosys session to show gate/flop counts.
* `hierarchy -check` to verify module instantiations.

---

## 🛠️ Lab: Flat Synthesis (step‑by‑step)

**Objective:** Flatten the hierarchy and synthesize the entire design as a single module.

**Script `scripts/synth_flat.ys`**

```yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ../rtl/multiple_modules.v
synth -top multiple_modules
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
flatten
show -prefix ../results/multiple_modules_flat
write_verilog -noattr ../results/multiple_modules_flat.v
```

<div align="center">

<img width="1024" height="1024" alt="flat_show" src="https://github.com/user-attachments/assets/befb2472-f491-4fad-b304-e46fb69d0c5e" />

</div>

**Run:** `yosys -s scripts/synth_flat.ys`

**Expected output:**

* `results/multiple_modules_flat.v`: only one module `multiple_modules` with standard cells instantiated directly inside it.
* Net names will be auto‑generated (e.g., `u1.*`), which is normal.

**Comparison:** Use `stat` after each flow to compare gate counts and `wc -l` on the written Verilog files to check verbosity.

---

## 🛠️ Lab: Submodule Synthesis

**Objective:** Synthesize only a submodule (e.g. `sub_module1`) to generate a reusable gate‑level block.

**Script `scripts/synth_submodule.ys`**

```yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ../rtl/multiple_modules.v
synth -top sub_module1
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show -prefix ../results/sub_module1
write_verilog ../results/sub_module1_syn.v
```
<div align="center">

<img width="1024" height="1024" alt="show_submodule1" src="https://github.com/user-attachments/assets/ae91bd97-7bc2-4f1e-9294-ec2d1ae1c6b2" />

</div>

**Objective:** Synthesize only a submodule (e.g. `sub_module2`) to generate a reusable gate‑level block.

**Script `scripts/synth_submodule.ys`**

```yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ../rtl/multiple_modules.v
synth -top sub_module2
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show -prefix ../results/sub_module1
write_verilog ../results/sub_module1_syn.v
```
<div align="center">

<img width="1916" height="1075" alt="show_submodule2" src="https://github.com/user-attachments/assets/14d55258-5396-4c40-ad1e-8ea21b38a7f1" />

</div>

**Why:** 

- Useful for IP reuse and hierarchical flows where multiple instances of the same block can share a synthesized implementation.

- **Divide and Conquer** rule -> When the design is massive, module synthesis have been used to obtain good process.

## 🧩 Hierarchical vs Flatten Synthesis Code

<div align="center">

<img width="1024" height="1024" alt="hier_and_flat" src="https://github.com/user-attachments/assets/0282418f-34c8-4c5f-b741-a9998839706e" />

</div>

---

## 🔹3️⃣ Various Flop Coding Styles and Optimization

## ❓ What Problem Do Flops Solve?

Digital circuits can be divided into:
- 🔄 **Combinational logic** → outputs depend only on current inputs.
- 🗂️ **Sequential logic** → outputs depend on inputs *and* stored history.

⚠️ Problem with combinational circuits → **glitches** (unwanted short pulses).

### 🔍 Example
Equation:  

```
y = (a · b) + c
```

- Inputs: `a`, `b`, `c`
- Process: `a & b → net1`, then `net1 | c → y`

⏳ Due to propagation delays:
- OR gate may update faster,
- AND gate may update slower,
- Output `y` flips temporarily → **glitch**.

---

## ✅ Why Do We Need Flops?

Flops = **stability + memory**

- ⏱️ **Samples input only on a clock edge**
- 🛑 **Holds value until next clock**
- 🛡️ **Shields downstream logic from glitches**

> 💡 Analogy: Flops are like 🚦 traffic lights —  
> Inputs may arrive at different times, but updates happen **only when the clock says GO**.

---

## 🧩 Where Do Flops Fit?

Typical structure:

```
[ Comb Logic ] → [ Flop ] → [ Comb Logic ] → [ Flop ] ...
```

- ❌ Without flops → glitches pass through.
- ✅ With flops → stable pipeline stages.

👉 This is the basis of **pipelining** in processors.

---

## 🎛️ Control Pins of Flops

Flops need **defined initial states** at power-up.

- 🔻 **Reset (Clear)** → force output = `0`
- 🔺 **Set (Preset)** → force output = `1`

Types of control:
1. ⚡ **Asynchronous (Async)** → reacts immediately, ignores clock
2. ⏱️ **Synchronous (Sync)** → reacts only on clock edge

---

## 🔄 Reset / Set Behavior

| 🔧 Case          | 📌 Behavior                                   | ⏲️ Clock Dependence |
|------------------|-----------------------------------------------|----------------------|
| **Async Reset**  | Output goes low immediately                   | ❌ Independent       |
| **Async Set**    | Output goes high immediately                  | ❌ Independent       |
| **Sync Reset**   | Output goes low *only on clock edge*          | ✅ Waits for clock   |
| **Sync + Async** | Some flops support both mechanisms            | ⚙️ Mixed             |

---

## 📝 Key Takeaways

- ⚠️ **Glitches** → due to unequal propagation delays
- ⏱️ **Flops** → store values, cut timing paths, prevent glitches
- 🛡️ **Clock edge** → ensures safe state updates
- 🔻 **Reset/Set** → define known startup conditions
- ⚡ **Sync vs Async** → defines *when* reset/set takes effect

---

✨ **Final Insight:**  
Flops = the **building blocks of synchronous design**.  
They make circuits predictable, reliable, and scalable 🚀.

---

## ⚡ Interesting Optimization: Mult2 & Mult8 in Verilog

### 🔍 Overview  

When multiplying by powers of two (like 2, 4, 8, etc.), synthesis tools optimize the multiplication into **bit shifts** instead of using a multiplier.  

- `a * 2` → `a << 1` (shift left by 1)  
- `a * 8` → `a << 3` (shift left by 3)  

This reduces hardware cost because:  
✅ No multiplier circuit is required  
✅ Only wiring/bit-shifting logic is used  
✅ Faster and more resource-efficient  

---

## 🖥️ Verilog Example

### Mult2 (×2)
```verilog
module mult2 (
    input  wire [2:0] a,   // 3-bit input
    output wire [3:0] y    // needs 4 bits (max 14)
);
    assign y = a << 1;     // optimized shift-left
endmodule
```

### Mult8 (×8)
```verilog
module mult8 (
    input  wire [2:0] a,   // 3-bit input
    output wire [6:0] y    // needs 7 bits (max 56)
);
    assign y = a << 3;     // optimized shift-left
endmodule
```

---

## ⚙️ How It Works  

- **Shift-left by 1 (×2):** Appends a `0` at the LSB, moving bits left.  
  Example: `a=3'b101 (5)` → `y=4'b1010 (10)`  

- **Shift-left by 3 (×8):** Appends three `0`s at the LSB.  
  Example: `a=3'b101 (5)` → `y=7'b0101000 (40)`  

---

## 📊 Output Table  

| a (bin) | a (dec) | a×2 (bin) | a×2 (dec) | a×8 (bin)   | a×8 (dec) |
|---------|---------|-----------|-----------|-------------|-----------|
| 000     | 0       | 0000      | 0         | 0000000     | 0         |
| 001     | 1       | 0010      | 2         | 0001000     | 8         |
| 010     | 2       | 0100      | 4         | 0010000     | 16        |
| 011     | 3       | 0110      | 6         | 0011000     | 24        |
| 100     | 4       | 1000      | 8         | 0100000     | 32        |
| 101     | 5       | 1010      | 10        | 0101000     | 40        |
| 110     | 6       | 1100      | 12        | 0110000     | 48        |
| 111     | 7       | 1110      | 14        | 0111000     | 56        |

---

## 🚀 Key Takeaways  

- Multiplication by powers of two is **just a shift**.  
- Synthesizers replace `*2`, `*4`, `*8`, … with **shift-left operations**.  
- This saves **area, power, and time**.  

✨ Pro Tip: Always check synthesis reports to confirm whether the tool inferred a shift instead of a multiplier!  

---

<div align="center">

## 🌟 End Of Day - 2 of RISC-V SoC Tapeout 

</div>
