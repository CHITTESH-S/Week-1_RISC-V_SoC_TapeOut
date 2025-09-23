## 🌟 RISC-V SoC Tapeout – Week 1: Day 2

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

<div align="center">

<img width="1024" height="1024" alt="hier_and_flat" src="https://github.com/user-attachments/assets/0282418f-34c8-4c5f-b741-a9998839706e" />

</div>

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

---

