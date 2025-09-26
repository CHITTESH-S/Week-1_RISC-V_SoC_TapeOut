## 🌟 RISC-V SoC Tapeout – Week 1: Day 4 (GLS, Blocking vs Non-Blocking and Synthesis-Simulation Mismatch)

### 🌟 Topics of the Day:

🧩 Gate-Level Simulation (GLS)

⚡ Blocking vs Non-Blocking in Verilog

⚠️ Synthesis–Simulation Mismatch

> 📚 Today we’ll explore how RTL → Gates → Simulation flow works, how coding style affects results, and how to avoid classic pitfalls.

---

### 🧩 1. Gate-Level Simulation (GLS)

- GLS = simulating the post-synthesis gate-level netlist 

- Running the testbench with the Netlist as DUT

### 🔑 Why GLS?

✅ Verify synthesis did not change functionality.

⏱️ Check timing (setup/hold violations with SDF annotation).

🔗 Validate test structures (scan chains, DFT logic).

- **GLS using iverilog**

> (Design, GLS, Testbench) -> iverilog -> .vcd file -> gtkwave -> waveform

### ⏳ When to run GLS?
- After synthesis, before physical design.
- Useful as a sanity check before P&R.

> 💡 Note: GLS helps catch glitches, X-propagation, or latches that RTL sim might hide.

---

## ⚠️ 2. Synthesis–Simulation Mismatch

> **Mismatch = RTL sim ≠ Gate-level sim ≠ Hardware behavior**

### 🛑 Causes

- Using non-synthesizable constructs (initial, #delay, $display).

- Incomplete sensitivity lists → latches get inferred.

- Order of assignments → different in RTL vs hardware.

- Tool interpretation differences.

### 🔧 Debugging Strategy

1. Compare RTL vs GLS waveforms.

2. Check synthesis logs for warnings (inferred latch, multiple drivers).

3. Inspect netlist structure (extra latches or buffers?).

4. Ensure resets are defined, no floating signals.

### 🏆 Best Practices

✔️ Always use always @(*) for combinational.

✔️ Separate combinational and sequential always blocks.

✔️ Never rely on initial blocks in ASIC flows.

---

## ⚡ 3. Blocking vs. Non-Blocking

> Verilog has two assignment styles, and choosing the right one prevents subtle bugs.

### ➡️ Blocking (=)

- Executes immediately.

- Good for combinational logic.

```verilog
always @(*) y = a & b;
```

### ⏳ Non-Blocking (<=)

- Executes at end of time step.

- Good for sequential logic.

```verilog
always @(posedge clk) q <= d;
```

### 📊 Comparison:

| Feature   | Blocking (`=`)        | Non-Blocking (`<=`)   |
| --------- | --------------------- | --------------------- |
| Execution | Sequential, immediate | Concurrent, scheduled |
| Use       | Combinational         | Sequential            |
| Risk      | Wrong order bugs      | Mixed usage issues    |





