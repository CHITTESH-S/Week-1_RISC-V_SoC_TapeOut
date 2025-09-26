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

## ⚡ 3. Blocking vs Non-Blocking

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

---

## 🧪 4. Labs

Each lab gives hands-on clarity.

### Example - 1: GLS and Synth-Simulation Mismatch

**Case 1 - 2:1 MUX (Ternary)**

```verilog
module ternary_operator_mux (input i0 , input i1 , input sel , output y);
	assign y = sel?i1:i0;
endmodule
```

**iverilog & gtkwave for RTL MUX**
```
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v 
./a.out
gtkwave tb_ternary_operator_mux.vcd
```
<div align="center">

<img width="1024" height="1024" alt="wave_ter_op_mux" src="https://github.com/user-attachments/assets/d31376af-0d9c-415d-a028-c5d187ce1052" />

</div>

**Yosys**
```yosys
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
write_verilog -noattr ternary_operator_mux_net.v
show
```
<div align="center">

<img width="1024" height="1024" alt="yosys_ter_op_mux" src="https://github.com/user-attachments/assets/f23e2dda-d132-4a58-9d62-b144f27468fe" />

</div>

**iverilog & gtkwave for GLS MUX**
```verilog
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```

<div align="center">

<img width="1024" height="1024" alt="netlist_ter_op_mux" src="https://github.com/user-attachments/assets/1519b921-27b3-4f0e-a0ff-ec4b8e0160d1" />

</div>

**Case 2 - BAD MUX**

```verilog
module bad_mux (input i0 , input i1 , input sel , output reg y);
always @ (sel)
begin
	if(sel)
		y <= i1;
	else 
		y <= i0;
end
endmodule
```

**iverilog & gtkwave for RTL BAD MUX**
```
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v 
./a.out
gtkwave tb_ternary_operator_mux.vcd
```
<div align="center">

<img width="1024" height="1024" alt="wave_bad_mux" src="https://github.com/user-attachments/assets/f19dc1ef-5288-4119-91c7-34eae1320630" />

</div>

**Yosys**
```yosys
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
write_verilog -noattr ternary_operator_mux_net.v
show
```
<div align="center">

<img width="1024" height="1024" alt="yosys_bad_mux" src="https://github.com/user-attachments/assets/7068bf58-baab-4565-b548-8c569e40759c" />

</div>

**iverilog & gtkwave for GLS BAD MUX**
```verilog
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```

<div align="center">

<img width="1024" height="1024" alt="netlist_bad_mux" src="https://github.com/user-attachments/assets/3bfba2bd-122c-4a8a-ad04-9f48d235f073" />

</div>

> **Problem:** missing sensitivity list + wrong assignment type.

### Example - 2: Synth-Simulation Mismatch Blocking Caveat

```verilog
module blocking_caveat (input a , input b , input  c, output reg d); 
reg x;
always @ (*)
begin
	d = x & c;
	x = a | b;
end
endmodule  
```

**iverilog & gtkwave for RTL Blocking Caveat**
```
iverilog blocking_caveat.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd
```

<div align="center">

<img width="1024" height="1024" alt="wave_blocking" src="https://github.com/user-attachments/assets/cd39f425-80ab-4c7e-ab2e-67a99df073d0" />

</div>

**Yosys**
```yosys
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
read_verilog blocking_caveat.v
synth -top blocking_caveat
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
write_verilog -noattr blocking_caveat_net.v
show
```
<div align="center">

<img width="1024" height="1024" alt="yosys_blocking" src="https://github.com/user-attachments/assets/93bf2e11-b794-4635-9848-90b70779617b" />

</div>

**iverilog & gtkwave for GLS Blocking Caveat**
```verilog
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v blocking_caveat_net.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd
```
<div align="center">

<img width="1024" height="1024" alt="netlist_blocking" src="https://github.com/user-attachments/assets/80ba8088-d355-4a26-b793-d8db3a43353c" />

</div>

---

## 📌 5. Summary

🔗 GLS = check correctness post-synthesis.

⚠️ Mismatch = occurs from bad coding or non-synth constructs.

⚡ Blocking vs Non-Blocking:

- = → combinational
- <= → sequential

🧪 Labs show:

- Correct MUX design.
- Pitfalls (bad mux, blocking caveat).
- GLS validation.

---

## 🎯 Final Checklist for Designers

✅ Use non-blocking in clocked always blocks.

✅ Use blocking in combinational blocks.

✅ Always include all signals in sensitivity list (@(*)).

✅ Run both RTL sim & GLS sim.

---

<div align="center">

## 🌟 End Of Day - 4 of RISC-V SoC Tapeout 

</div>
