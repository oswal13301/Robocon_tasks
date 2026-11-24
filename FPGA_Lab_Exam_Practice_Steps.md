# EEE F314 FPGA Lab Exam (25 Nov 2026): Full Practice Steps, Question by Question

**Mohnish Oswal · 2024A3PS0414P · PYNQ-Z2 (xc7z020clg400-1)**

Each question below is complete on its own: follow it from Part A to the end without looking anywhere else. Files come from the exam pack (`.v`, `.py`, `.c`, `.m`, `.slx`). Paths assume `C:/fpga_exam/`; change them to your folder.

**Before the exam (once):** in MATLAB, run `build_q9_model`, `build_q10_model`, `build_q11_model` and `build_q12_model` inside their folders. This creates the `.slx` files. Copy everything to your pen drive.

**Contents:** [Q1](#q1) · [Q2](#q2) · [Q3](#q3) · [Q4](#q4) · [Q5](#q5) · [Q6](#q6) · [Q7](#q7) · [Q8](#q8) · [Q9](#q9) · [Q10](#q10) · [Q11](#q11) · [Q12](#q12)

---

<a id="q1"></a>

## Q1: 16-bit Adder with AND, XOR and NOT

**Files:** none. This question uses only built-in Vivado IPs, with no custom RTL.

**Board connections:** PYNQ-Z2, micro-USB cable to the PC, boot jumper on **JTAG**.

### Part A: Create the Vivado project

1. Open **Vivado**. On the Quick Start page click **Create Project**, then **Next**.
2. Project name: `q1_adder`. Project location: a folder with no spaces, e.g. `C:/fpga_exam`. Keep **Create project subdirectory** ticked. Click **Next**.
3. Select **RTL Project** and tick **Do not specify sources at this time**. Click **Next**.
4. Click the **Boards** tab, type `pynq` in **Search** and select **PYNQ-Z2**. Click **Next**. If PYNQ-Z2 isn't listed, click the **Parts** tab, search `xc7z020clg400-1`, and select it.
5. Click **Finish**. The project opens.

### Part B: Create the block design

1. Flow Navigator → IP INTEGRATOR → **Create Block Design**.
2. Design name: `design_1`. Click **OK**. An empty **Diagram** opens.

### Part C: Add and configure the IPs

1. **How to add an IP:** click **+** (Add IP) on the Diagram toolbar or press **Ctrl+I**, type the name in **Search**, and double-click the result. **How to rename:** click the block, change **Name** in the **Block Properties** panel, and press **Enter**. **How to configure:** double-click the block.
2. **VIO:** add **VIO (Virtual Input/Output)**. Keep the name `vio_0`. Configure it:
   - **General Options:** Input Probe Count = **1**, Output Probe Count = **2**
   - **PROBE_IN Ports:** PROBE_IN0 width = **17**
   - **PROBE_OUT Ports:** PROBE_OUT0 width **16**, initial value **0x0**; PROBE_OUT1 width **16**, initial value **0x0**
   - Click **OK**.
3. **Constant (zero):** add **Constant**. Rename it `xlconstant_zero`. Configure: Const Width = **1**, Const Val = **0**. Click **OK**.
4. **Concat A:** add **Concat**. Rename it `xlconcat_A`. Configure: Number of Ports = **2**, In0 Width = **16**, In1 Width = **1**. Click **OK**.
5. **Concat B:** add another **Concat**. Rename it `xlconcat_B`. Configure: Number of Ports = **2**, In0 Width = **16**, In1 Width = **1**. Click **OK**.
6. **Adder/Subtracter:** add **Adder/Subtracter**. Keep the name `c_addsub_0`. Configure:
   - **Basic tab:** Implement using **Fabric**; Input Type **Unsigned**; Input A Width **17**; Input B Width **17**; Add Mode **Add**; Output Width **17**; Latency Configuration **Manual**; Latency **0**
   - **Control tab:** untick **Carry Out**, **Clock Enable**, **Sync Clear** and **Bypass**
   - Click **OK**.
7. **Constant 19 (first logic stage):** add **Constant**. Rename it `xlconstant_19`. Configure: Const Width = **17**, Const Val = **19**. Click **OK**.
8. **AND gate:** add **Utility Vector Logic** (called *Vector Logic* in newer Vivado). Rename it `and_gate`. Configure: C_SIZE = **17**, C_OPERATION = **and**. Click **OK**.
9. **Constant 512 (second logic stage):** add **Constant**. Rename it `xlconstant_512`. Configure: Const Width = **17**, Const Val = **512**. Click **OK**.
10. **XOR gate:** add **Utility Vector Logic**. Rename it `xor_gate`. Configure: C_SIZE = **17**, C_OPERATION = **xor**. Click **OK**.
11. **NOT gate:** add **Utility Vector Logic**. Rename it `not_gate`. Configure: C_SIZE = **17**, C_OPERATION = **not**. Click **OK**. Only Op1 exists.
12. **ILA:** add **ILA (Integrated Logic Analyzer)**. Keep the name `ila_0`. Configure:
   - **General Options:** Monitor Type **Native**; Number of Probes **4**; Sample Data Depth **1024**
   - **Probe_Ports(0..7):** PROBE0, PROBE1, PROBE2 and PROBE3 width = **17** each
   - Click **OK**.

### Part D: Connect the datapath (drag from pin to pin)

1. `vio_0/probe_out0[15:0]` → `xlconcat_A/In0`  (operand A)
2. `vio_0/probe_out1[15:0]` → `xlconcat_B/In0`  (operand B)
3. `xlconstant_zero/dout` → `xlconcat_A/In1`, and again from `xlconstant_zero/dout` → `xlconcat_B/In1`
4. `xlconcat_A/dout[16:0]` → `c_addsub_0/A`
5. `xlconcat_B/dout[16:0]` → `c_addsub_0/B`
6. `c_addsub_0/S` → `and_gate/Op1`, and `c_addsub_0/S` → `ila_0/probe0`  (**S**)
7. `xlconstant_19/dout` → `and_gate/Op2`
8. `and_gate/Res` → `xor_gate/Op1`, and `and_gate/Res` → `ila_0/probe1`  (**P**)
9. `xlconstant_512/dout` → `xor_gate/Op2`
10. `xor_gate/Res` → `not_gate/Op1`, and `xor_gate/Res` → `ila_0/probe2`  (**Q**)
11. `not_gate/Res` → `vio_0/probe_in0`, and `not_gate/Res` → `ila_0/probe3`  (**R**)

### Part E: Make the clock external

1. Right-click the `clk` pin of `vio_0` and choose **Make External**. A port named `clk_0` appears on the left. If it has another name, click it and rename it to `clk_0` in External Port Properties.
2. Drag from the `clk` pin of `ila_0` onto the `clk_0` wire.
3. If `c_addsub_0` still shows a `CLK` pin, drag it onto the `clk_0` wire too.
4. Click **Regenerate Layout** (the ⟳ icon) to tidy up, then **Ctrl+S**.

### Part F: Validate and create the HDL wrapper

1. Click **Validate Design** (the ✓ icon on the Diagram toolbar, or press **F6**). Expect **Validation successful**. Warnings are fine; errors must be fixed.
2. Press **Ctrl+S** to save the block design.
3. In **Sources** (Hierarchy tab), right-click **design_1 (design_1.bd)** and choose **Create HDL Wrapper**.
4. Choose **Let Vivado manage wrapper and auto-update** and click **OK**. `design_1_wrapper` becomes the top module (shown in bold).

### Part G: Create the constraints file lab.xdc (typed inside Vivado)

1. Flow Navigator → PROJECT MANAGER → **Add Sources**.
2. Select **Add or create constraints** and click **Next**.
3. Click **Create File**. File type: `XDC`. File name: `lab`. Click **OK**, then **Finish**.
4. In **Sources**, expand **Constraints → constrs_1** and double-click **lab.xdc**.
5. Type these lines exactly, then press **Ctrl+S**:
   ```
   set_property -dict {PACKAGE_PIN H16 IOSTANDARD LVCMOS33} [get_ports clk_0]
   create_clock -period 8.000 -name clk_0 [get_ports clk_0]
   ```

### Part H: Generate the bitstream

1. Flow Navigator → PROGRAM AND DEBUG → **Generate Bitstream**.
2. If asked to save, click **Save**. When asked to run synthesis and implementation, click **Yes**, then **OK** in the Launch Runs dialog.
3. Wait for it to finish (5–15 min; progress shows at the top right).
4. When **Bitstream Generation successfully completed** appears, select **Open Hardware Manager** and click **OK**.

### Part I: Program the PYNQ-Z2

1. Set the PYNQ-Z2 boot-mode jumper to **JTAG**.
2. Connect the board's micro-USB (PROG/UART) port to the PC and switch the board **ON**.
3. In Hardware Manager, click **Open Target → Auto Connect**. `xc7z020_1` appears in the **Hardware** window.
4. Click **Program Device → xc7z020_1**.
5. Check that **Bitstream file** = `…/design_1_wrapper.bit` and **Debug probes file** = `…/design_1_wrapper.ltx` (both filled in automatically). Click **Program**.
6. After programming, `hw_vio_1` and `hw_ila_1` appear under `xc7z020_1`, and their dashboards open. To open one yourself, double-click it.

### Part J: Set up the VIO dashboard

1. Open the **hw_vio_1** dashboard. Click **+** (Add Probes), select **all** probes and click **OK**.
2. Probe mapping (name in the dashboard → signal):
   - `probe_out0` = **A** (16-bit)
   - `probe_out1` = **B** (16-bit)
   - `probe_in0` = **R** (17-bit result)
3. To change a probe's radix, right-click it → **Radix** → **Unsigned Decimal** (or **Hex** where the steps say so).
4. To set an output probe, click its **Value** cell, type the value and press **Enter**. The FPGA gets the value immediately.
5. Set all three probes to **Unsigned Decimal**. A and B should read 0.

### Part K: Set up the ILA

1. Open the **hw_ila_1** dashboard. It has the windows **Settings**, **Status**, **Trigger Setup** and **Waveform**.
2. ILA probe mapping: `probe0` = **S**, `probe1` = **P**, `probe2` = **Q**, `probe3` = **R**.
3. In **Settings - hw_ila_1**, keep Trigger mode = BASIC_ONLY, Capture mode = ALWAYS, Data depth = **1024**, Trigger position in window = **512**.
4. In **Trigger Setup - hw_ila_1**, click **+**, select the probe connected to **S** (`…c_addsub_0_S` / probe0) and click **OK**.
5. In the S row set Operator **==** and Radix **[U]** (unsigned). The value is set per test case below.

### Part L: Test case 1 (A = 1000, B = 2000)

1. In **Trigger Setup**, set the S value to **3000**.
2. Click **Run trigger for this ILA core** (▶) in the ILA dashboard. Status shows **Waiting for Trigger**.
3. In the VIO dashboard set `probe_out0` (A) = **1000** and press Enter.
4. Set `probe_out1` (B) = **2000** and press Enter. The ILA triggers as soon as S = 3000, and the waveform appears.
5. In the **Waveform** window, select all four probes, right-click → **Radix → Unsigned Decimal**.
6. Read the values at the trigger marker: **S = 3000, P = 16, Q = 528, R = 130543**.
7. In the VIO dashboard, read `probe_in0` (R) = **130543** (0x1FDEF).
8. Take a screenshot of the waveform and the VIO, and write the values in the results table (Part P).

### Part M: Test case 2 (A = 12345, B = 23456)

1. In **Trigger Setup**, set the S value to **35801**.
2. Click **Run trigger for this ILA core** (▶) in the ILA dashboard. Status shows **Waiting for Trigger**.
3. In the VIO dashboard set `probe_out0` (A) = **12345** and press Enter.
4. Set `probe_out1` (B) = **23456** and press Enter. The ILA triggers as soon as S = 35801, and the waveform appears.
5. In the **Waveform** window, select all four probes, right-click → **Radix → Unsigned Decimal**.
6. Read the values at the trigger marker: **S = 35801, P = 17, Q = 529, R = 130542**.
7. In the VIO dashboard, read `probe_in0` (R) = **130542** (0x1FDEE).
8. Take a screenshot of the waveform and the VIO, and write the values in the results table (Part P).

### Part N: Test case 3 (given on the spot)

1. Write down the A and B the examiner gives you.
2. Compute **S = A + B**.
3. Compute **P = S AND 19** using **P = 16 × [ (S mod 32) ≥ 16 ] + (S mod 4)**. That is: add 16 if (S mod 32) is 16 or more, then add S mod 4.
4. Compute **Q = P + 512** and **R = 130559 − P**.
5. In **Trigger Setup**, set the S value to your computed **S**.
6. Click **Run trigger** (▶). Status shows **Waiting for Trigger**.
7. In the VIO set `probe_out0` (A) = given A and press Enter. Set `probe_out1` (B) = given B and press Enter.
8. In the Waveform, set all probes to Unsigned Decimal and read S, P, Q, R. They must match what you computed.
9. Read R on VIO `probe_in0`. Screenshot both and fill in the table.

### Part O: Trigger condition to state (Task 3)

1. Write: **"Trigger: probe0 (S) == A + B (the expected S for the test case), radix unsigned, comparator ==, Global AND, trigger position 512, data depth 1024. The ILA was armed before applying A and B from the VIO."**
2. Why trigger on S: S changes first and P, Q, R follow in the same cycle (there are no registers), so one trigger on S captures all four stages.

### Part P: Results table

| TC | A | B | S | P | Q | R (VIO) | Expected R |
|---|---|---|---|---|---|---|---|
| 1 | 1000 | 2000 | 3000 | 16 | 528 | | **130543** (0x1FDEF) |
| 2 | 12345 | 23456 | 35801 | 17 | 529 | | **130542** (0x1FDEE) |
| 3 | | | | | | | 130559 − P |

---

<a id="q2"></a>

## Q2: 16-bit Subtractor with OR, NAND and NOT

**Files:** none. This question uses only built-in Vivado IPs, with no custom RTL.

**Board connections:** PYNQ-Z2, micro-USB cable to the PC, boot jumper on **JTAG**.

### Part A: Create the Vivado project

1. Open **Vivado**. On the Quick Start page click **Create Project**, then **Next**.
2. Project name: `q2_subtractor`. Project location: a folder with no spaces, e.g. `C:/fpga_exam`. Keep **Create project subdirectory** ticked. Click **Next**.
3. Select **RTL Project** and tick **Do not specify sources at this time**. Click **Next**.
4. Click the **Boards** tab, type `pynq` in **Search** and select **PYNQ-Z2**. Click **Next**. If PYNQ-Z2 isn't listed, click the **Parts** tab, search `xc7z020clg400-1`, and select it.
5. Click **Finish**. The project opens.

### Part B: Create the block design

1. Flow Navigator → IP INTEGRATOR → **Create Block Design**.
2. Design name: `design_1`. Click **OK**. An empty **Diagram** opens.

### Part C: Add and configure the IPs

1. **How to add an IP:** click **+** (Add IP) on the Diagram toolbar or press **Ctrl+I**, type the name in **Search**, and double-click the result. **How to rename:** click the block, change **Name** in the **Block Properties** panel, and press **Enter**. **How to configure:** double-click the block.
2. **VIO:** add **VIO (Virtual Input/Output)**. Keep the name `vio_0`. Configure it:
   - **General Options:** Input Probe Count = **1**, Output Probe Count = **2**
   - **PROBE_IN Ports:** PROBE_IN0 width = **17**
   - **PROBE_OUT Ports:** PROBE_OUT0 width **16**, initial value **0x0**; PROBE_OUT1 width **16**, initial value **0x0**
   - Click **OK**.
3. **Constant (zero):** add **Constant**. Rename it `xlconstant_zero`. Configure: Const Width = **1**, Const Val = **0**. Click **OK**.
4. **Concat A:** add **Concat**. Rename it `xlconcat_A`. Configure: Number of Ports = **2**, In0 Width = **16**, In1 Width = **1**. Click **OK**.
5. **Concat B:** add another **Concat**. Rename it `xlconcat_B`. Configure: Number of Ports = **2**, In0 Width = **16**, In1 Width = **1**. Click **OK**.
6. **Adder/Subtracter:** add **Adder/Subtracter**. Keep the name `c_addsub_0`. Configure:
   - **Basic tab:** Implement using **Fabric**; Input Type **Unsigned**; Input A Width **17**; Input B Width **17**; Add Mode **Subtract**; Output Width **17**; Latency Configuration **Manual**; Latency **0**
   - **Control tab:** untick **Carry Out**, **Clock Enable**, **Sync Clear** and **Bypass**
   - Click **OK**.
7. **Constant 256 (first logic stage):** add **Constant**. Rename it `xlconstant_256_or`. Configure: Const Width = **17**, Const Val = **256**. Click **OK**.
8. **OR gate:** add **Utility Vector Logic** (called *Vector Logic* in newer Vivado). Rename it `or_gate`. Configure: C_SIZE = **17**, C_OPERATION = **or**. Click **OK**.
9. **Constant 256 (second logic stage):** add **Constant**. Rename it `xlconstant_256_nand`. Configure: Const Width = **17**, Const Val = **256**. Click **OK**.
10. **NAND gate:** add **Utility Vector Logic**. Rename it `nand_gate`. Configure: C_SIZE = **17**, C_OPERATION = **nand**. Click **OK**.
   - If **nand** isn't offered: set `nand_gate` to **and**, and add one more Utility Vector Logic named `nand_not` (C_SIZE **17**, **not**). In Part D, the wire `nand_gate/Res` then goes to `nand_not/Op1`, and **`nand_not/Res`** is Q: it goes to `not_gate/Op1` and `ila_0/probe2`.
11. **NOT gate:** add **Utility Vector Logic**. Rename it `not_gate`. Configure: C_SIZE = **17**, C_OPERATION = **not**. Click **OK**. Only Op1 exists.
12. **ILA:** add **ILA (Integrated Logic Analyzer)**. Keep the name `ila_0`. Configure:
   - **General Options:** Monitor Type **Native**; Number of Probes **4**; Sample Data Depth **1024**
   - **Probe_Ports(0..7):** PROBE0, PROBE1, PROBE2 and PROBE3 width = **17** each
   - Click **OK**.

### Part D: Connect the datapath (drag from pin to pin)

1. `vio_0/probe_out0[15:0]` → `xlconcat_A/In0`  (operand A)
2. `vio_0/probe_out1[15:0]` → `xlconcat_B/In0`  (operand B)
3. `xlconstant_zero/dout` → `xlconcat_A/In1`, and again from `xlconstant_zero/dout` → `xlconcat_B/In1`
4. `xlconcat_A/dout[16:0]` → `c_addsub_0/A`
5. `xlconcat_B/dout[16:0]` → `c_addsub_0/B`
6. `c_addsub_0/S` → `or_gate/Op1`, and `c_addsub_0/S` → `ila_0/probe0`  (**S**)
7. `xlconstant_256_or/dout` → `or_gate/Op2`
8. `or_gate/Res` → `nand_gate/Op1`, and `or_gate/Res` → `ila_0/probe1`  (**P**)
9. `xlconstant_256_nand/dout` → `nand_gate/Op2`
10. `nand_gate/Res` → `not_gate/Op1`, and `nand_gate/Res` → `ila_0/probe2`  (**Q**)
11. `not_gate/Res` → `vio_0/probe_in0`, and `not_gate/Res` → `ila_0/probe3`  (**R**)

### Part E: Make the clock external

1. Right-click the `clk` pin of `vio_0` and choose **Make External**. A port named `clk_0` appears on the left. If it has another name, click it and rename it to `clk_0` in External Port Properties.
2. Drag from the `clk` pin of `ila_0` onto the `clk_0` wire.
3. If `c_addsub_0` still shows a `CLK` pin, drag it onto the `clk_0` wire too.
4. Click **Regenerate Layout** (the ⟳ icon) to tidy up, then **Ctrl+S**.

### Part F: Validate and create the HDL wrapper

1. Click **Validate Design** (the ✓ icon on the Diagram toolbar, or press **F6**). Expect **Validation successful**. Warnings are fine; errors must be fixed.
2. Press **Ctrl+S** to save the block design.
3. In **Sources** (Hierarchy tab), right-click **design_1 (design_1.bd)** and choose **Create HDL Wrapper**.
4. Choose **Let Vivado manage wrapper and auto-update** and click **OK**. `design_1_wrapper` becomes the top module (shown in bold).

### Part G: Create the constraints file lab.xdc (typed inside Vivado)

1. Flow Navigator → PROJECT MANAGER → **Add Sources**.
2. Select **Add or create constraints** and click **Next**.
3. Click **Create File**. File type: `XDC`. File name: `lab`. Click **OK**, then **Finish**.
4. In **Sources**, expand **Constraints → constrs_1** and double-click **lab.xdc**.
5. Type these lines exactly, then press **Ctrl+S**:
   ```
   set_property -dict {PACKAGE_PIN H16 IOSTANDARD LVCMOS33} [get_ports clk_0]
   create_clock -period 8.000 -name clk_0 [get_ports clk_0]
   ```

### Part H: Generate the bitstream

1. Flow Navigator → PROGRAM AND DEBUG → **Generate Bitstream**.
2. If asked to save, click **Save**. When asked to run synthesis and implementation, click **Yes**, then **OK** in the Launch Runs dialog.
3. Wait for it to finish (5–15 min; progress shows at the top right).
4. When **Bitstream Generation successfully completed** appears, select **Open Hardware Manager** and click **OK**.

### Part I: Program the PYNQ-Z2

1. Set the PYNQ-Z2 boot-mode jumper to **JTAG**.
2. Connect the board's micro-USB (PROG/UART) port to the PC and switch the board **ON**.
3. In Hardware Manager, click **Open Target → Auto Connect**. `xc7z020_1` appears in the **Hardware** window.
4. Click **Program Device → xc7z020_1**.
5. Check that **Bitstream file** = `…/design_1_wrapper.bit` and **Debug probes file** = `…/design_1_wrapper.ltx` (both filled in automatically). Click **Program**.
6. After programming, `hw_vio_1` and `hw_ila_1` appear under `xc7z020_1`, and their dashboards open. To open one yourself, double-click it.

### Part J: Set up the VIO dashboard

1. Open the **hw_vio_1** dashboard. Click **+** (Add Probes), select **all** probes and click **OK**.
2. Probe mapping (name in the dashboard → signal):
   - `probe_out0` = **A** (16-bit)
   - `probe_out1` = **B** (16-bit)
   - `probe_in0` = **R** (17-bit result)
3. To change a probe's radix, right-click it → **Radix** → **Unsigned Decimal** (or **Hex** where the steps say so).
4. To set an output probe, click its **Value** cell, type the value and press **Enter**. The FPGA gets the value immediately.
5. Set all three probes to **Unsigned Decimal**. A and B should read 0.

### Part K: Set up the ILA

1. Open the **hw_ila_1** dashboard. It has the windows **Settings**, **Status**, **Trigger Setup** and **Waveform**.
2. ILA probe mapping: `probe0` = **S**, `probe1` = **P**, `probe2` = **Q**, `probe3` = **R**.
3. In **Settings - hw_ila_1**, keep Trigger mode = BASIC_ONLY, Capture mode = ALWAYS, Data depth = **1024**, Trigger position in window = **512**.
4. In **Trigger Setup - hw_ila_1**, click **+**, select the probe connected to **S** (`…c_addsub_0_S` / probe0) and click **OK**.
5. In the S row set Operator **==** and Radix **[U]** (unsigned). The value is set per test case below.

### Part L: Test case 1 (A = 1000, B = 200)

1. In **Trigger Setup**, set the S value to **800**.
2. Click **Run trigger for this ILA core** (▶) in the ILA dashboard. Status shows **Waiting for Trigger**.
3. In the VIO dashboard set `probe_out0` (A) = **1000** and press Enter.
4. Set `probe_out1` (B) = **200** and press Enter. The ILA triggers as soon as S = 800, and the waveform appears.
5. In the **Waveform** window, select all four probes, right-click → **Radix → Unsigned Decimal**.
6. Read the values at the trigger marker: **S = 800, P = 800, Q = 130815, R = 256**.
7. In the VIO dashboard, read `probe_in0` (R) = **256** (0x00100).
8. Take a screenshot of the waveform and the VIO, and write the values in the results table (Part P).

### Part M: Test case 2 (A = 12345, B = 2345)

1. In **Trigger Setup**, set the S value to **10000**.
2. Click **Run trigger for this ILA core** (▶) in the ILA dashboard. Status shows **Waiting for Trigger**.
3. In the VIO dashboard set `probe_out0` (A) = **12345** and press Enter.
4. Set `probe_out1` (B) = **2345** and press Enter. The ILA triggers as soon as S = 10000, and the waveform appears.
5. In the **Waveform** window, select all four probes, right-click → **Radix → Unsigned Decimal**.
6. Read the values at the trigger marker: **S = 10000, P = 10000, Q = 130815, R = 256**.
7. In the VIO dashboard, read `probe_in0` (R) = **256** (0x00100).
8. Take a screenshot of the waveform and the VIO, and write the values in the results table (Part P).

### Part N: Test case 3 (given on the spot)

1. Write down the A and B the examiner gives you.
2. Compute **S = A − B**. If A < B, use **S = A − B + 131072** (17-bit wrap-around).
3. Compute **P = S OR 256**: if (S mod 512) < 256 then P = S + 256, otherwise P = S.
4. **Q = 130815 (0x1FEFF)** and **R = 256 (0x00100)** for every input. P always has bit 8 set, so P AND 256 = 256, NAND gives ~256, and NOT gives 256.
5. In **Trigger Setup**, set the S value to your computed **S**.
6. Click **Run trigger** (▶). Status shows **Waiting for Trigger**.
7. In the VIO set `probe_out0` (A) = given A and press Enter. Set `probe_out1` (B) = given B and press Enter.
8. In the Waveform, set all probes to Unsigned Decimal and read S, P, Q, R.
9. Read R on VIO `probe_in0` (= 256). Screenshot both and fill in the table.

### Part O: Trigger condition to state (Task 3)

1. Write: **"Trigger: probe0 (S) == (A − B) mod 131072 (the expected S for the test case), radix unsigned, comparator ==, Global AND, trigger position 512, data depth 1024. The ILA was armed before applying A and B from the VIO."**
2. Why trigger on S: Q and R never change in this design (Q = 130815, R = 256 for every input), so they can't trigger. S is the signal that changes.

### Part P: Results table

| TC | A | B | S | P | Q | R (VIO) | Expected R |
|---|---|---|---|---|---|---|---|
| 1 | 1000 | 200 | 800 | 800 | 130815 | | **256** (0x00100) |
| 2 | 12345 | 2345 | 10000 | 10000 | 130815 | | **256** (0x00100) |
| 3 | | | | | | | **256** |

---

<a id="q3"></a>

## Q3: 4-Digit Password Lock (A-5-C-3)

**Files:** `q03/pass_lock.v`

**Board connections:** PYNQ-Z2, micro-USB cable to the PC, boot jumper on **JTAG**.

**How the design works:** state 0–3 = waiting for digit 1–4; 4 = unlocked; 5 = locked out. A wrong digit sends it back to state 0 and adds 1 to attempts. The 3rd wrong digit locks it out until rst. LEDs: LD0 and LD1 = attempts, LD2 = locked out, LD3 = unlocked.

### Part A: Create the Vivado project

1. Open **Vivado**. On the Quick Start page click **Create Project**, then **Next**.
2. Project name: `q3_lock`. Project location: a folder with no spaces, e.g. `C:/fpga_exam`. Keep **Create project subdirectory** ticked. Click **Next**.
3. Select **RTL Project** and tick **Do not specify sources at this time**. Click **Next**.
4. Click the **Boards** tab, type `pynq` in **Search** and select **PYNQ-Z2**. Click **Next**. If PYNQ-Z2 isn't listed, click the **Parts** tab, search `xc7z020clg400-1`, and select it.
5. Click **Finish**. The project opens.

### Part B: Add the Verilog file

1. Flow Navigator → PROJECT MANAGER → **Add Sources**.
2. Select **Add or create design sources** and click **Next**.
3. Click **Add Files**, browse to `q03/pass_lock.v`, and click **OK**.
4. Tick **Copy sources into project** and click **Finish**.
5. Check that `pass_lock` appears under **Sources → Design Sources**.

### Part C: Create the block design

1. Flow Navigator → IP INTEGRATOR → **Create Block Design**.
2. Design name: `design_1`. Click **OK**. An empty **Diagram** opens.

### Part D: Add the RTL module

1. Right-click an empty area of the Diagram and choose **Add Module**.
2. Select **pass_lock** and click **OK**. Block `pass_lock_0` appears, with inputs clk, rst, enter, digit[3:0] and outputs unlock, attempts[1:0], state[2:0], enter_pulse, led[3:0].

### Part E: Add and configure the VIO

1. **How to add an IP:** click **+** (Add IP) on the Diagram toolbar or press **Ctrl+I**, type the name in **Search**, and double-click the result. **How to rename:** click the block, change **Name** in the **Block Properties** panel, and press **Enter**. **How to configure:** double-click the block.
2. Add **VIO (Virtual Input/Output)**. Keep the name `vio_0`. Double-click it.
3. **General Options:** Input Probe Count = **3**, Output Probe Count = **3**.
4. **PROBE_IN Ports:** PROBE_IN0 width **1** (unlock); PROBE_IN1 width **2** (attempts); PROBE_IN2 width **3** (state).
5. **PROBE_OUT Ports:**
   - PROBE_OUT0 width **1**, initial value **0x1** (rst)
   - PROBE_OUT1 width **1**, initial value **0x0** (enter)
   - PROBE_OUT2 width **4**, initial value **0x0** (digit)
6. Click **OK**.

### Part F: Add and configure the ILA

1. Add **ILA (Integrated Logic Analyzer)**. Keep the name `ila_0`. Double-click it.
2. **General Options:** Monitor Type **Native**; Number of Probes **5**; Sample Data Depth **1024**.
3. **Probe_Ports(0..7):** PROBE0 width **3** (state); PROBE1 width **4** (digit); PROBE2 width **1** (enter_pulse); PROBE3 width **1** (unlock); PROBE4 width **2** (attempts).
4. Click **OK**.

### Part G: Connect (drag from pin to pin)

1. `vio_0/probe_out0` → `pass_lock_0/rst`
2. `vio_0/probe_out1` → `pass_lock_0/enter`
3. `vio_0/probe_out2[3:0]` → `pass_lock_0/digit[3:0]`, and `vio_0/probe_out2` → `ila_0/probe1`
4. `pass_lock_0/unlock` → `vio_0/probe_in0`, and `pass_lock_0/unlock` → `ila_0/probe3`
5. `pass_lock_0/attempts[1:0]` → `vio_0/probe_in1`, and `pass_lock_0/attempts` → `ila_0/probe4`
6. `pass_lock_0/state[2:0]` → `vio_0/probe_in2`, and `pass_lock_0/state` → `ila_0/probe0`
7. `pass_lock_0/enter_pulse` → `ila_0/probe2`

### Part H: Make clock and LEDs external

1. Right-click the `clk` pin of `pass_lock_0` and choose **Make External**. Port `clk_0` appears. If it's named differently, rename it to `clk_0`.
2. Drag the `clk` pin of `vio_0` onto the `clk_0` wire. Drag the `clk` pin of `ila_0` onto the `clk_0` wire.
3. Right-click the `led[3:0]` pin of `pass_lock_0` and choose **Make External**. Port `led_0[3:0]` appears; rename it to `led_0` if needed.
4. Click **Regenerate Layout** (⟳), then **Ctrl+S**.

### Part I: Validate and create the HDL wrapper

1. Click **Validate Design** (the ✓ icon on the Diagram toolbar, or press **F6**). Expect **Validation successful**. Warnings are fine; errors must be fixed.
2. Press **Ctrl+S** to save the block design.
3. In **Sources** (Hierarchy tab), right-click **design_1 (design_1.bd)** and choose **Create HDL Wrapper**.
4. Choose **Let Vivado manage wrapper and auto-update** and click **OK**. `design_1_wrapper` becomes the top module (shown in bold).

### Part J: Create the constraints file lab.xdc (typed inside Vivado)

1. Flow Navigator → PROJECT MANAGER → **Add Sources**.
2. Select **Add or create constraints** and click **Next**.
3. Click **Create File**. File type: `XDC`. File name: `lab`. Click **OK**, then **Finish**.
4. In **Sources**, expand **Constraints → constrs_1** and double-click **lab.xdc**.
5. Type these lines exactly, then press **Ctrl+S**:
   ```
   set_property -dict {PACKAGE_PIN H16 IOSTANDARD LVCMOS33} [get_ports clk_0]
   create_clock -period 8.000 -name clk_0 [get_ports clk_0]
   set_property -dict {PACKAGE_PIN R14 IOSTANDARD LVCMOS33} [get_ports {led_0[0]}]
   set_property -dict {PACKAGE_PIN P14 IOSTANDARD LVCMOS33} [get_ports {led_0[1]}]
   set_property -dict {PACKAGE_PIN N16 IOSTANDARD LVCMOS33} [get_ports {led_0[2]}]
   set_property -dict {PACKAGE_PIN M14 IOSTANDARD LVCMOS33} [get_ports {led_0[3]}]
   ```

### Part K: Generate the bitstream

1. Flow Navigator → PROGRAM AND DEBUG → **Generate Bitstream**.
2. If asked to save, click **Save**. When asked to run synthesis and implementation, click **Yes**, then **OK** in the Launch Runs dialog.
3. Wait for it to finish (5–15 min; progress shows at the top right).
4. When **Bitstream Generation successfully completed** appears, select **Open Hardware Manager** and click **OK**.

### Part L: Program the PYNQ-Z2

1. Set the PYNQ-Z2 boot-mode jumper to **JTAG**.
2. Connect the board's micro-USB (PROG/UART) port to the PC and switch the board **ON**.
3. In Hardware Manager, click **Open Target → Auto Connect**. `xc7z020_1` appears in the **Hardware** window.
4. Click **Program Device → xc7z020_1**.
5. Check that **Bitstream file** = `…/design_1_wrapper.bit` and **Debug probes file** = `…/design_1_wrapper.ltx` (both filled in automatically). Click **Program**.
6. After programming, `hw_vio_1` and `hw_ila_1` appear under `xc7z020_1`, and their dashboards open. To open one yourself, double-click it.

### Part M: Set up the VIO dashboard

1. Open the **hw_vio_1** dashboard. Click **+** (Add Probes), select **all** probes and click **OK**.
2. Probe mapping (name in the dashboard → signal):
   - `probe_out0` = **rst**
   - `probe_out1` = **enter**
   - `probe_out2` = **digit** (set radix to **Hex**)
   - `probe_in0` = **unlock**
   - `probe_in1` = **attempts**
   - `probe_in2` = **state**
3. To change a probe's radix, right-click it → **Radix** → **Unsigned Decimal** (or **Hex** where the steps say so).
4. To set an output probe, click its **Value** cell, type the value and press **Enter**. The FPGA gets the value immediately.
5. **Set `probe_out0` (rst) = 0.** The VIO starts with rst = 1, which holds the lock in reset. Check state = 0 and all LEDs OFF.
6. Rule for every entry: **set digit first, then enter = 1, then enter = 0**.

### Part N: Task 1: Correct password unlocks

1. digit = **A**. enter = **1**, then enter = **0**. VIO: state = **1**.
2. digit = **5**. enter = **1**, then enter = **0**. State = **2**.
3. digit = **C**. enter = **1**, then enter = **0**. State = **3**.
4. digit = **3**. enter = **1**, then enter = **0**. State = **4**, **unlock = 1**, **LD3 ON**.
5. Screenshot the VIO and photograph the board (LD3 ON).

### Part O: Task 2: Wrong password, attempts and lock-out

1. rst = **1**, then rst = **0**. State = 0, attempts = 0, LEDs OFF.
2. digit = **7** (wrong). enter = **1**, then enter = **0**. **attempts = 1**, state = 0, **LD0 ON**.
3. digit = **7**. enter = **1**, then **0**. **attempts = 2**, **LD1 ON** (LD0 OFF).
4. digit = **7**. enter = **1**, then **0**. **attempts = 3**, **state = 5 (LOCKED)**, **LD2 ON** (LD0 and LD1 also ON).
5. Enter A, 5, C, 3 (each: set digit, enter 1, enter 0). Nothing changes: still locked, unlock = 0.
6. Screenshot the VIO and photograph the board (LD2 ON).
7. rst = **1**, then rst = **0** to clear the lock-out.

### Part P: Tasks 3 and 4: Edge detector proof and ILA capture (trigger position 100)

1. In the VIO check that rst = 0, enter = 0 and state = 0. If not, do rst = 1 → 0.
2. Open the **hw_ila_1** dashboard. ILA probe mapping: `probe0` = state, `probe1` = digit, `probe2` = enter_pulse, `probe3` = unlock, `probe4` = attempts.
3. In **Settings - hw_ila_1**, set **Trigger position in window = 100** (Data depth 1024).
4. In **Trigger Setup**, click **+**, choose **probe2 (enter_pulse)**, and click **OK**. Set Operator **==** and Value **1** (or **R** for rising edge).
5. In the VIO set digit = **A**.
6. Click **Run trigger** (▶). Status shows **Waiting for Trigger**.
7. In the VIO set enter = **1** and **leave it at 1**.
8. The ILA triggers. In the waveform: enter_pulse is 1 for exactly **one** sample at sample 100, and state changes 0 → 1 right after and stays 1. **This is Task 4's capture.**
9. Keep enter = 1 for about 10 seconds. VIO state stays **1**: only one step was taken while enter was held. **This proves the edge detector (Task 3).**
10. Set enter = **0**. Screenshot the waveform (zoom near sample 100) and the VIO.

### Part Q: Expected results

| Task | Expected |
|---|---|
| 1 | state 0→1→2→3→4, unlock = 1, LD3 ON |
| 2 | attempts 1, 2, 3 (LD0/LD1); then state 5, LD2 ON; correct code ignored |
| 3 | enter held high, state advances once only |
| 4 | ILA: single-cycle enter_pulse at sample 100 |

---

<a id="q4"></a>

## Q4: Programmable Password with Master Override

**Files:** `q04/pass_prog.v`

**Board connections:** PYNQ-Z2, micro-USB cable to the PC, boot jumper on **JTAG**.

**How the design works:** IDLE = 0, VERIFIED = 1, LOCKED = 2. Default password 0x1234; master 0xFFFF always unlocks. A new password can be written (mode = 1 + write pulse) only in VERIFIED. rst does **not** reset the stored password. LEDs: LD0 and LD1 = attempts, LD2 = locked out, LD3 = unlocked.

### Part A: Create the Vivado project

1. Open **Vivado**. On the Quick Start page click **Create Project**, then **Next**.
2. Project name: `q4_prog_lock`. Project location: a folder with no spaces, e.g. `C:/fpga_exam`. Keep **Create project subdirectory** ticked. Click **Next**.
3. Select **RTL Project** and tick **Do not specify sources at this time**. Click **Next**.
4. Click the **Boards** tab, type `pynq` in **Search** and select **PYNQ-Z2**. Click **Next**. If PYNQ-Z2 isn't listed, click the **Parts** tab, search `xc7z020clg400-1`, and select it.
5. Click **Finish**. The project opens.

### Part B: Add the Verilog file

1. Flow Navigator → PROJECT MANAGER → **Add Sources**.
2. Select **Add or create design sources** and click **Next**.
3. Click **Add Files**, browse to `q04/pass_prog.v`, and click **OK**.
4. Tick **Copy sources into project** and click **Finish**.
5. Check that `pass_prog` appears under **Sources → Design Sources**.

### Part C: Create the block design

1. Flow Navigator → IP INTEGRATOR → **Create Block Design**.
2. Design name: `design_1`. Click **OK**. An empty **Diagram** opens.

### Part D: Add the RTL module

1. Right-click an empty area of the Diagram and choose **Add Module**.
2. Select **pass_prog** and click **OK**. Block `pass_prog_0` appears, with inputs clk, rst, mode, write, enter, data[15:0] and outputs unlock, attempts[1:0], state[2:0], stored_pass[15:0], write_pulse, led[3:0].

### Part E: Add and configure the VIO

1. **How to add an IP:** click **+** (Add IP) on the Diagram toolbar or press **Ctrl+I**, type the name in **Search**, and double-click the result. **How to rename:** click the block, change **Name** in the **Block Properties** panel, and press **Enter**. **How to configure:** double-click the block.
2. Add **VIO (Virtual Input/Output)**. Keep the name `vio_0`. Double-click it.
3. **General Options:** Input Probe Count = **3**, Output Probe Count = **5**.
4. **PROBE_IN Ports:** PROBE_IN0 width **1** (unlock); PROBE_IN1 width **2** (attempts); PROBE_IN2 width **16** (stored_pass).
5. **PROBE_OUT Ports:**
   - PROBE_OUT0 width **1**, initial **0x1** (rst)
   - PROBE_OUT1 width **1**, initial **0x0** (mode)
   - PROBE_OUT2 width **1**, initial **0x0** (write)
   - PROBE_OUT3 width **1**, initial **0x0** (enter)
   - PROBE_OUT4 width **16**, initial **0x0000** (data)
6. Click **OK**.

### Part F: Add and configure the ILA

1. Add **ILA (Integrated Logic Analyzer)**. Keep the name `ila_0`. Double-click it.
2. **General Options:** Monitor Type **Native**; Number of Probes **4**; Sample Data Depth **1024**; tick **Same Number of Comparators for All Probe Ports**.
3. **Probe_Ports(0..7):** PROBE0 width **3** (state); PROBE1 width **16** (data); PROBE2 width **1** (write_pulse); PROBE3 width **1** (unlock).
4. Click **OK**.

### Part G: Connect (drag from pin to pin)

1. `vio_0/probe_out0` → `pass_prog_0/rst`
2. `vio_0/probe_out1` → `pass_prog_0/mode`
3. `vio_0/probe_out2` → `pass_prog_0/write`
4. `vio_0/probe_out3` → `pass_prog_0/enter`
5. `vio_0/probe_out4[15:0]` → `pass_prog_0/data[15:0]`, and `vio_0/probe_out4` → `ila_0/probe1`
6. `pass_prog_0/unlock` → `vio_0/probe_in0`, and `pass_prog_0/unlock` → `ila_0/probe3`
7. `pass_prog_0/attempts[1:0]` → `vio_0/probe_in1`
8. `pass_prog_0/stored_pass[15:0]` → `vio_0/probe_in2`
9. `pass_prog_0/state[2:0]` → `ila_0/probe0`
10. `pass_prog_0/write_pulse` → `ila_0/probe2`

### Part H: Make clock and LEDs external

1. Right-click the `clk` pin of `pass_prog_0` and choose **Make External**. Port `clk_0` appears; rename it to `clk_0` if needed.
2. Drag the `clk` pin of `vio_0` onto the `clk_0` wire. Drag the `clk` pin of `ila_0` onto the `clk_0` wire.
3. Right-click the `led[3:0]` pin of `pass_prog_0` and choose **Make External**. Port `led_0[3:0]` appears; rename it to `led_0` if needed.
4. Click **Regenerate Layout** (⟳), then **Ctrl+S**.

### Part I: Validate and create the HDL wrapper

1. Click **Validate Design** (the ✓ icon on the Diagram toolbar, or press **F6**). Expect **Validation successful**. Warnings are fine; errors must be fixed.
2. Press **Ctrl+S** to save the block design.
3. In **Sources** (Hierarchy tab), right-click **design_1 (design_1.bd)** and choose **Create HDL Wrapper**.
4. Choose **Let Vivado manage wrapper and auto-update** and click **OK**. `design_1_wrapper` becomes the top module (shown in bold).

### Part J: Create the constraints file lab.xdc (typed inside Vivado)

1. Flow Navigator → PROJECT MANAGER → **Add Sources**.
2. Select **Add or create constraints** and click **Next**.
3. Click **Create File**. File type: `XDC`. File name: `lab`. Click **OK**, then **Finish**.
4. In **Sources**, expand **Constraints → constrs_1** and double-click **lab.xdc**.
5. Type these lines exactly, then press **Ctrl+S**:
   ```
   set_property -dict {PACKAGE_PIN H16 IOSTANDARD LVCMOS33} [get_ports clk_0]
   create_clock -period 8.000 -name clk_0 [get_ports clk_0]
   set_property -dict {PACKAGE_PIN R14 IOSTANDARD LVCMOS33} [get_ports {led_0[0]}]
   set_property -dict {PACKAGE_PIN P14 IOSTANDARD LVCMOS33} [get_ports {led_0[1]}]
   set_property -dict {PACKAGE_PIN N16 IOSTANDARD LVCMOS33} [get_ports {led_0[2]}]
   set_property -dict {PACKAGE_PIN M14 IOSTANDARD LVCMOS33} [get_ports {led_0[3]}]
   ```

### Part K: Generate the bitstream

1. Flow Navigator → PROGRAM AND DEBUG → **Generate Bitstream**.
2. If asked to save, click **Save**. When asked to run synthesis and implementation, click **Yes**, then **OK** in the Launch Runs dialog.
3. Wait for it to finish (5–15 min; progress shows at the top right).
4. When **Bitstream Generation successfully completed** appears, select **Open Hardware Manager** and click **OK**.

### Part L: Program the PYNQ-Z2

1. Set the PYNQ-Z2 boot-mode jumper to **JTAG**.
2. Connect the board's micro-USB (PROG/UART) port to the PC and switch the board **ON**.
3. In Hardware Manager, click **Open Target → Auto Connect**. `xc7z020_1` appears in the **Hardware** window.
4. Click **Program Device → xc7z020_1**.
5. Check that **Bitstream file** = `…/design_1_wrapper.bit` and **Debug probes file** = `…/design_1_wrapper.ltx` (both filled in automatically). Click **Program**.
6. After programming, `hw_vio_1` and `hw_ila_1` appear under `xc7z020_1`, and their dashboards open. To open one yourself, double-click it.

### Part M: Set up the VIO dashboard

1. Open the **hw_vio_1** dashboard. Click **+** (Add Probes), select **all** probes and click **OK**.
2. Probe mapping (name in the dashboard → signal):
   - `probe_out0` = **rst**
   - `probe_out1` = **mode**
   - `probe_out2` = **write**
   - `probe_out3` = **enter**
   - `probe_out4` = **data** (radix **Hex**)
   - `probe_in0` = **unlock**
   - `probe_in1` = **attempts**
   - `probe_in2` = **stored_pass** (radix **Hex**)
3. To change a probe's radix, right-click it → **Radix** → **Unsigned Decimal** (or **Hex** where the steps say so).
4. To set an output probe, click its **Value** cell, type the value and press **Enter**. The FPGA gets the value immediately.
5. **Set `probe_out0` (rst) = 0.** The VIO starts with rst = 1. Check stored_pass = **1234** and all LEDs OFF.
6. Rule for every entry: **set data (and mode) first, then pulse: enter (or write) = 1, then = 0**.

### Part N: Task 1: Unlock, change password, lock again

1. data = **1234**. enter = **1**, then enter = **0**. **unlock = 1, LD3 ON**.
2. mode = **1**. data = **BEEF**.
3. write = **1**, then write = **0**. VIO **stored_pass = BEEF**.
4. mode = **0**.
5. enter = **1**, then enter = **0**. **unlock = 0, LD3 OFF** (locked again).
6. Screenshot the VIO after each check.

### Part O: Task 2: Old password fails, new password works

1. data = **1234**. enter = **1**, then **0**. **attempts = 1, LD0 ON**, unlock = 0.
2. data = **BEEF**. enter = **1**, then **0**. **unlock = 1, LD3 ON**, attempts = 0.
3. enter = **1**, then **0** to lock again (LD3 OFF).
4. Screenshot the VIO and photograph the board.

### Part P: Task 3: Lock-out and master override

1. data = **0001**. enter = **1**, then **0**. attempts = 1.
2. data = **0002**. enter = **1**, then **0**. attempts = 2.
3. data = **0003**. enter = **1**, then **0**. **attempts = 3, LD2 ON** (locked out; LD0 and LD1 also ON).
4. data = **BEEF**. enter = **1**, then **0**. Nothing happens: still locked, unlock = 0.
5. data = **FFFF**. enter = **1**, then **0**. **unlock = 1, LD3 ON, attempts = 0** (LD2 OFF).
6. Screenshot the VIO and photograph the board. **Stay unlocked (VERIFIED) for Task 4.**

### Part Q: Task 4: ILA combined trigger (state == 1 AND write_pulse == 1, position 100)

1. Check that VIO unlock = 1 (state is VERIFIED). If not, enter data = FFFF and pulse enter.
2. Open the **hw_ila_1** dashboard. ILA probe mapping: `probe0` = state, `probe1` = data, `probe2` = write_pulse, `probe3` = unlock.
3. In **Settings - hw_ila_1**, set **Trigger position in window = 100** (Data depth 1024).
4. In **Trigger Setup**, click **+**, choose **probe0 (state)**, click **OK**. Set Operator **==**, Radix **[U]**, Value **1**.
5. Click **+** again, choose **probe2 (write_pulse)**, click **OK**. Set Operator **==**, Value **1**.
6. Set the **Trigger Condition** dropdown (top of Trigger Setup) to **Global AND** ("Set trigger condition to 'Global AND'").
7. In the VIO: mode = **1**, data = **CAFE**.
8. Click **Run trigger** (▶). Status shows **Waiting for Trigger**.
9. In the VIO: write = **1**, then write = **0**.
10. The ILA triggers. Waveform: write_pulse = 1 at sample 100 while state = 1, and data = CAFE. VIO **stored_pass = CAFE** at the same time.
11. Set mode = **0**. Screenshot the waveform and the VIO.

### Part R: Expected results

| Task | Expected |
|---|---|
| 1 | 1234 unlocks (LD3); stored_pass becomes BEEF; enter re-locks (LD3 OFF) |
| 2 | 1234 fails (attempts 1, LD0); BEEF unlocks (LD3) |
| 3 | 3 wrong attempts, LD2 ON, BEEF ignored; FFFF unlocks (LD3, attempts 0) |
| 4 | ILA triggers on state == 1 AND write_pulse == 1 at sample 100; stored_pass changes |

---

<a id="q5"></a>

## Q5: Drone Return-to-Home Advisor

**Files:** `q05/drone_rth.v`, `q05/q5_drone.py`

**Board connections:** Vivado part: none. Python part: PYNQ-Z2 with the PYNQ SD card, boot jumper on **SD**, Ethernet cable to the PC, micro-USB for power.

### Part A: Create the Vivado project

1. Open **Vivado**. On the Quick Start page click **Create Project**, then **Next**.
2. Project name: `q5_drone`. Project location: a folder with no spaces, e.g. `C:/fpga_exam`. Keep **Create project subdirectory** ticked. Click **Next**.
3. Select **RTL Project** and tick **Do not specify sources at this time**. Click **Next**.
4. Click the **Boards** tab, type `pynq` in **Search** and select **PYNQ-Z2**. Click **Next**. If PYNQ-Z2 isn't listed, click the **Parts** tab, search `xc7z020clg400-1`, and select it.
5. Click **Finish**. The project opens.

### Part B: Add the Verilog file

1. Flow Navigator → PROJECT MANAGER → **Add Sources**.
2. Select **Add or create design sources** and click **Next**.
3. Click **Add Files**, browse to `q05/drone_rth.v`, and click **OK**.
4. Tick **Copy sources into project** and click **Finish**.
5. Check that `drone_rth` appears under **Sources → Design Sources**.

### Part C: Create the block design

1. Flow Navigator → IP INTEGRATOR → **Create Block Design**.
2. Design name: `design_1`. Click **OK**. An empty **Diagram** opens.

### Part D: Add the Zynq PS and set the 50 MHz PL clock

1. **How to add an IP:** click **+** (Add IP) on the Diagram toolbar or press **Ctrl+I**, type the name in **Search**, and double-click the result. **How to rename:** click the block, change **Name** in the **Block Properties** panel, and press **Enter**. **How to configure:** double-click the block.
2. Add **ZYNQ7 Processing System** (`processing_system7_0`).
3. Click **Run Block Automation** (green banner). Tick `processing_system7_0`, keep **Apply Board Preset** ticked, and click **OK**. DDR and FIXED_IO become external.
4. Double-click `processing_system7_0`. Go to **Clock Configuration → PL Fabric Clocks** and set **FCLK_CLK0 Requested Frequency (MHz) = 50**.
5. Check **PS-PL Configuration → AXI Non Secure Enablement → GP Master AXI Interface → M AXI GP0 interface** is ticked (the board preset ticks it). Click **OK**.

### Part E: Add the AXI GPIO IPs

1. Add **AXI GPIO**. Keep the name `axi_gpio_0`. Double-click it. In the **IP Configuration** tab:
   - **GPIO:** tick **All Outputs**, GPIO Width = **1**  (en)
   - Tick **Enable Dual Channel**
   - **GPIO 2:** tick **All Outputs**, GPIO Width = **32**  (battery_mv)
   - **Board** tab: GPIO (and GPIO2) = **Custom**. Click **OK**.
2. Add **AXI GPIO**. Keep the name `axi_gpio_1`. Double-click it. In the **IP Configuration** tab:
   - **GPIO:** tick **All Outputs**, GPIO Width = **32**  (distance_m)
   - Leave **Enable Dual Channel** unticked
   - **Board** tab: GPIO (and GPIO2) = **Custom**. Click **OK**.
3. Add **AXI GPIO**. Keep the name `axi_gpio_2`. Double-click it. In the **IP Configuration** tab:
   - **GPIO:** tick **All Inputs**, GPIO Width = **32**  (range_m)
   - Tick **Enable Dual Channel**
   - **GPIO 2:** tick **All Inputs**, GPIO Width = **1**  (rth)
   - **Board** tab: GPIO (and GPIO2) = **Custom**. Click **OK**.

### Part F: Add the AXI BRAM Controller

1. Add **AXI BRAM Controller**. Keep the name `axi_bram_ctrl_0`. Double-click it.
2. AXI Protocol **AXI4**, Data Width **32**, **Number of BRAM interfaces = 1**. Click **OK**.

### Part G: Add the Block Memory Generator

1. Add **Block Memory Generator**. Keep the name `blk_mem_gen_0`. Double-click it.
2. **Basic** tab: Mode = **BRAM Controller**, Memory Type = **True Dual Port RAM**.
3. **Port B Options** tab: **untick Primitives Output Register** (gives a 1-cycle read latency, which the Verilog expects). Enable Port Type = **Use ENB Pin**.
4. Click **OK**. Leave Mode as **BRAM Controller**, not Stand Alone, because the Verilog uses byte addresses (word i at 4 × i).

### Part H: Connect BRAM controller to BRAM port A

1. Drag from `axi_bram_ctrl_0/BRAM_PORTA` to `blk_mem_gen_0/BRAM_PORTA` (interface-to-interface connection).

### Part I: Run Connection Automation (AXI only)

1. Click **Run Connection Automation** (green banner).
2. Tick **only**: `axi_bram_ctrl_0/S_AXI`, `axi_gpio_0/S_AXI`, `axi_gpio_1/S_AXI`, `axi_gpio_2/S_AXI`.
3. **Untick** every `GPIO` / `GPIO2` entry and `blk_mem_gen_0/BRAM_PORTB`. If those are ticked, Vivado makes them board pins or adds a second controller.
4. Click **OK**. Vivado adds `ps7_0_axi_periph` (AXI Interconnect) and `rst_ps7_0_50M`, and connects the AXI clocks and resets.

### Part J: Add constants for BRAM port B control pins

1. Add **Constant**. Rename it `const_rstb`. Configure: Const Width **1**, Const Val **0**. Click **OK**.
2. Add **Constant**. Rename it `const_web`. Configure: Const Width **4**, Const Val **0**. Click **OK**.

### Part K: Add the RTL module

1. Right-click an empty area of the Diagram and choose **Add Module**.
2. Select **drone_rth** and click **OK**. Block `drone_rth_0` appears.

### Part L: Connect BRAM port B to the module

1. On `blk_mem_gen_0`, click the **+** next to `BRAM_PORTB` to show its pins.
2. `drone_rth_0/bram_addr[31:0]` → `blk_mem_gen_0/addrb[31:0]`
3. `drone_rth_0/bram_en` → `blk_mem_gen_0/enb`
4. `blk_mem_gen_0/doutb[31:0]` → `drone_rth_0/bram_dout[31:0]`
5. `const_rstb/dout` → `blk_mem_gen_0/rstb`
6. `const_web/dout` → `blk_mem_gen_0/web[3:0]`
7. `processing_system7_0/FCLK_CLK0` → `blk_mem_gen_0/clkb` (drag `clkb` onto the FCLK_CLK0 wire).
8. Leave `dinb` unconnected.

### Part M: Connect the AXI GPIOs and clock to the module

1. On `axi_gpio_0`, click **+** next to `GPIO` and next to `GPIO2` to show the pins.
2. `axi_gpio_0/gpio_io_o` → `drone_rth_0/en`  (PS → PL, 1-bit)
3. `axi_gpio_0/gpio2_io_o` → `drone_rth_0/battery_mv`  (PS → PL, 32-bit)
4. On `axi_gpio_1`, click **+** next to `GPIO` to show the pins.
5. `axi_gpio_1/gpio_io_o` → `drone_rth_0/distance_m`  (PS → PL, 32-bit)
6. On `axi_gpio_2`, click **+** next to `GPIO` and next to `GPIO2` to show the pins.
7. `drone_rth_0/range_m` → `axi_gpio_2/gpio_io_i`  (PL → PS, 32-bit)
8. `drone_rth_0/rth` → `axi_gpio_2/gpio2_io_i`  (PL → PS, 1-bit)
9. `processing_system7_0/FCLK_CLK0` → `drone_rth_0/clk` (drag `clk` onto the FCLK_CLK0 wire).
10. Click **Regenerate Layout** (⟳), then **Ctrl+S**.

### Part N: Check addresses

1. Open the **Address Editor** tab (next to Diagram).
2. If anything is under **Unassigned**, right-click it and choose **Assign All**.
3. Note the base address of `axi_bram_ctrl_0` (usually **0x4000_0000**, range 8K). The Python script finds it automatically; this is the fallback value.

### Part O: Validate and create the HDL wrapper

1. Click **Validate Design** (the ✓ icon on the Diagram toolbar, or press **F6**). Expect **Validation successful**. Warnings are fine; errors must be fixed.
2. Press **Ctrl+S** to save the block design.
3. In **Sources** (Hierarchy tab), right-click **design_1 (design_1.bd)** and choose **Create HDL Wrapper**.
4. Choose **Let Vivado manage wrapper and auto-update** and click **OK**. `design_1_wrapper` becomes the top module (shown in bold).

### Part P: Generate the bitstream (no constraints file needed: the clock comes from the PS)

1. Flow Navigator → PROGRAM AND DEBUG → **Generate Bitstream**.
2. If asked to save, click **Save**. When asked to run synthesis and implementation, click **Yes**, then **OK**.
3. Wait for **Bitstream Generation successfully completed**. Choose **View Reports** (or Cancel) and click **OK**.

### Part Q: Confirm BRAM is used (utilization report)

1. Flow Navigator → IMPLEMENTATION → **Open Implemented Design**.
2. Menu **Reports → Report Utilization** → **OK**.
3. In the report, open **Memory**. **Block RAM Tile** must be **≥ 0.5** (1 RAMB18 or 1 RAMB36). Screenshot it.

### Part R: Collect the .bit and .hwh files

1. Make a folder, e.g. `C:/fpga_exam/q5_upload`.
2. Copy `C:/fpga_exam/q5_drone/q5_drone.runs/impl_1/design_1_wrapper.bit` into it and rename the copy **`q5.bit`**.
3. Copy `C:/fpga_exam/q5_drone/q5_drone.gen/sources_1/bd/design_1/hw_handoff/design_1.hwh` into it and rename the copy **`q5.hwh`**. In Vivado 2020.1 or older, the path uses `.srcs` instead of `.gen`.
4. Copy `q05/q5_drone.py` into the same folder.
5. The names must match: `q5.bit` + `q5.hwh`. The script loads `Overlay("q5.bit")`.

### Part S: Boot the PYNQ board and open Jupyter

1. Insert the PYNQ SD card. Set the boot-mode jumper to **SD**. Connect the Ethernet cable from the board to the PC (or the lab switch). Connect the micro-USB for power.
2. Switch the board **ON** and wait about 1 minute until boot finishes (the LEDs flash when it's ready).
3. If the cable goes straight into the PC, set the PC's Ethernet IPv4 to **192.168.2.1**, mask **255.255.255.0**: Control Panel → Network Connections → Ethernet → Properties → IPv4.
4. Open a browser at **http://192.168.2.99:9090** (use the lab's board IP if different). Password: **xilinx**.

### Part T: Upload and run the Python script

1. In Jupyter: **New → Folder**. Tick **Untitled Folder** → **Rename** → `q5` → open the folder.
2. Click **Upload**, select `q5.bit`, `q5.hwh` and `q5_drone.py`, then click the blue **Upload** button next to each file. In JupyterLab, use the ↑ upload icon in the file browser.
3. **New → Python 3**. A notebook opens inside the same folder.
4. In the first cell type `%run q5_drone.py` and press **Shift+Enter**. Alternatively, open `q5_drone.py` and paste its contents into the cell.
5. The script loads the overlay, writes [0, 200, 500, 900, 1400, 2000, 2700, 3500] to BRAM words 0–7, sets en = 1, runs the 3 tests and prints `Test | Expected | Observed | PASS/FAIL`.
6. Check that all 3 lines say **PASS**. Screenshot the output.

### Part U: Expected results

| Test | Inputs | Expected |
|---|---|---|
| 1 Fresh Battery, Close to Home | battery_mv = 4150, distance_m = 1200 | range_m = 3500, rth = 0 |
| 2 Half Battery, Far Away | battery_mv = 3720, distance_m = 1800 | range_m = 1400, rth = 1 |
| 3 Nearly Empty | battery_mv = 3250, distance_m = 50 | range_m = 0, rth = 1 |

**If it fails:**
- All outputs 0: en not written, FCLK_CLK0 not wired to `clk`/`clkb`, or BRAM port A not connected.
- Values from the wrong word: BMG in Stand Alone mode, or *Primitives Output Register* still ticked.
- `KeyError: axi_bram_ctrl_0`: run `ol.ip_dict.keys()` and `ol.mem_dict.keys()` in a cell, or replace the `bram = MMIO(base, rng)` line with `bram = MMIO(0x40000000, 0x2000)` (your Address Editor value).
- `Overlay` error: the `.bit`/`.hwh` names differ, or they're not in the notebook's folder.

---

<a id="q6"></a>

## Q6: Digital Audio Volume Control with Clipping

**Files:** `q06/audio_vol.v`, `q06/q6_audio.py`

**Board connections:** Vivado part: none. Python part: PYNQ-Z2 with the PYNQ SD card, boot jumper on **SD**, Ethernet cable to the PC, micro-USB for power.

### Part A: Create the Vivado project

1. Open **Vivado**. On the Quick Start page click **Create Project**, then **Next**.
2. Project name: `q6_audio`. Project location: a folder with no spaces, e.g. `C:/fpga_exam`. Keep **Create project subdirectory** ticked. Click **Next**.
3. Select **RTL Project** and tick **Do not specify sources at this time**. Click **Next**.
4. Click the **Boards** tab, type `pynq` in **Search** and select **PYNQ-Z2**. Click **Next**. If PYNQ-Z2 isn't listed, click the **Parts** tab, search `xc7z020clg400-1`, and select it.
5. Click **Finish**. The project opens.

### Part B: Add the Verilog file

1. Flow Navigator → PROJECT MANAGER → **Add Sources**.
2. Select **Add or create design sources** and click **Next**.
3. Click **Add Files**, browse to `q06/audio_vol.v`, and click **OK**.
4. Tick **Copy sources into project** and click **Finish**.
5. Check that `audio_vol` appears under **Sources → Design Sources**.

### Part C: Create the block design

1. Flow Navigator → IP INTEGRATOR → **Create Block Design**.
2. Design name: `design_1`. Click **OK**. An empty **Diagram** opens.

### Part D: Add the Zynq PS and set the 50 MHz PL clock

1. **How to add an IP:** click **+** (Add IP) on the Diagram toolbar or press **Ctrl+I**, type the name in **Search**, and double-click the result. **How to rename:** click the block, change **Name** in the **Block Properties** panel, and press **Enter**. **How to configure:** double-click the block.
2. Add **ZYNQ7 Processing System** (`processing_system7_0`).
3. Click **Run Block Automation** (green banner). Tick `processing_system7_0`, keep **Apply Board Preset** ticked, and click **OK**. DDR and FIXED_IO become external.
4. Double-click `processing_system7_0`. Go to **Clock Configuration → PL Fabric Clocks** and set **FCLK_CLK0 Requested Frequency (MHz) = 50**.
5. Check **PS-PL Configuration → AXI Non Secure Enablement → GP Master AXI Interface → M AXI GP0 interface** is ticked (the board preset ticks it). Click **OK**.

### Part E: Add the AXI GPIO IPs

1. Add **AXI GPIO**. Keep the name `axi_gpio_0`. Double-click it. In the **IP Configuration** tab:
   - **GPIO:** tick **All Outputs**, GPIO Width = **1**  (en)
   - Tick **Enable Dual Channel**
   - **GPIO 2:** tick **All Outputs**, GPIO Width = **32**  (sample)
   - **Board** tab: GPIO (and GPIO2) = **Custom**. Click **OK**.
2. Add **AXI GPIO**. Keep the name `axi_gpio_1`. Double-click it. In the **IP Configuration** tab:
   - **GPIO:** tick **All Outputs**, GPIO Width = **3**  (vol)
   - Leave **Enable Dual Channel** unticked
   - **Board** tab: GPIO (and GPIO2) = **Custom**. Click **OK**.
3. Add **AXI GPIO**. Keep the name `axi_gpio_2`. Double-click it. In the **IP Configuration** tab:
   - **GPIO:** tick **All Inputs**, GPIO Width = **32**  (gain)
   - Tick **Enable Dual Channel**
   - **GPIO 2:** tick **All Inputs**, GPIO Width = **32**  (out)
   - **Board** tab: GPIO (and GPIO2) = **Custom**. Click **OK**.

### Part F: Add the AXI BRAM Controller

1. Add **AXI BRAM Controller**. Keep the name `axi_bram_ctrl_0`. Double-click it.
2. AXI Protocol **AXI4**, Data Width **32**, **Number of BRAM interfaces = 1**. Click **OK**.

### Part G: Add the Block Memory Generator

1. Add **Block Memory Generator**. Keep the name `blk_mem_gen_0`. Double-click it.
2. **Basic** tab: Mode = **BRAM Controller**, Memory Type = **True Dual Port RAM**.
3. **Port B Options** tab: **untick Primitives Output Register** (gives a 1-cycle read latency, which the Verilog expects). Enable Port Type = **Use ENB Pin**.
4. Click **OK**. Leave Mode as **BRAM Controller**, not Stand Alone, because the Verilog uses byte addresses (word i at 4 × i).

### Part H: Connect BRAM controller to BRAM port A

1. Drag from `axi_bram_ctrl_0/BRAM_PORTA` to `blk_mem_gen_0/BRAM_PORTA` (interface-to-interface connection).

### Part I: Run Connection Automation (AXI only)

1. Click **Run Connection Automation** (green banner).
2. Tick **only**: `axi_bram_ctrl_0/S_AXI`, `axi_gpio_0/S_AXI`, `axi_gpio_1/S_AXI`, `axi_gpio_2/S_AXI`.
3. **Untick** every `GPIO` / `GPIO2` entry and `blk_mem_gen_0/BRAM_PORTB`. If those are ticked, Vivado makes them board pins or adds a second controller.
4. Click **OK**. Vivado adds `ps7_0_axi_periph` (AXI Interconnect) and `rst_ps7_0_50M`, and connects the AXI clocks and resets.

### Part J: Add constants for BRAM port B control pins

1. Add **Constant**. Rename it `const_rstb`. Configure: Const Width **1**, Const Val **0**. Click **OK**.
2. Add **Constant**. Rename it `const_web`. Configure: Const Width **4**, Const Val **0**. Click **OK**.

### Part K: Add the RTL module

1. Right-click an empty area of the Diagram and choose **Add Module**.
2. Select **audio_vol** and click **OK**. Block `audio_vol_0` appears.

### Part L: Connect BRAM port B to the module

1. On `blk_mem_gen_0`, click the **+** next to `BRAM_PORTB` to show its pins.
2. `audio_vol_0/bram_addr[31:0]` → `blk_mem_gen_0/addrb[31:0]`
3. `audio_vol_0/bram_en` → `blk_mem_gen_0/enb`
4. `blk_mem_gen_0/doutb[31:0]` → `audio_vol_0/bram_dout[31:0]`
5. `const_rstb/dout` → `blk_mem_gen_0/rstb`
6. `const_web/dout` → `blk_mem_gen_0/web[3:0]`
7. `processing_system7_0/FCLK_CLK0` → `blk_mem_gen_0/clkb` (drag `clkb` onto the FCLK_CLK0 wire).
8. Leave `dinb` unconnected.

### Part M: Connect the AXI GPIOs and clock to the module

1. On `axi_gpio_0`, click **+** next to `GPIO` and next to `GPIO2` to show the pins.
2. `axi_gpio_0/gpio_io_o` → `audio_vol_0/en`  (PS → PL, 1-bit)
3. `axi_gpio_0/gpio2_io_o` → `audio_vol_0/sample`  (PS → PL, 32-bit)
4. On `axi_gpio_1`, click **+** next to `GPIO` to show the pins.
5. `axi_gpio_1/gpio_io_o` → `audio_vol_0/vol`  (PS → PL, 3-bit)
6. On `axi_gpio_2`, click **+** next to `GPIO` and next to `GPIO2` to show the pins.
7. `audio_vol_0/gain` → `axi_gpio_2/gpio_io_i`  (PL → PS, 32-bit)
8. `audio_vol_0/out` → `axi_gpio_2/gpio2_io_i`  (PL → PS, 32-bit)
9. `processing_system7_0/FCLK_CLK0` → `audio_vol_0/clk` (drag `clk` onto the FCLK_CLK0 wire).
10. Click **Regenerate Layout** (⟳), then **Ctrl+S**.

### Part N: Check addresses

1. Open the **Address Editor** tab (next to Diagram).
2. If anything is under **Unassigned**, right-click it and choose **Assign All**.
3. Note the base address of `axi_bram_ctrl_0` (usually **0x4000_0000**, range 8K). The Python script finds it automatically; this is the fallback value.

### Part O: Validate and create the HDL wrapper

1. Click **Validate Design** (the ✓ icon on the Diagram toolbar, or press **F6**). Expect **Validation successful**. Warnings are fine; errors must be fixed.
2. Press **Ctrl+S** to save the block design.
3. In **Sources** (Hierarchy tab), right-click **design_1 (design_1.bd)** and choose **Create HDL Wrapper**.
4. Choose **Let Vivado manage wrapper and auto-update** and click **OK**. `design_1_wrapper` becomes the top module (shown in bold).

### Part P: Generate the bitstream (no constraints file needed: the clock comes from the PS)

1. Flow Navigator → PROGRAM AND DEBUG → **Generate Bitstream**.
2. If asked to save, click **Save**. When asked to run synthesis and implementation, click **Yes**, then **OK**.
3. Wait for **Bitstream Generation successfully completed**. Choose **View Reports** (or Cancel) and click **OK**.

### Part Q: Confirm BRAM is used (utilization report)

1. Flow Navigator → IMPLEMENTATION → **Open Implemented Design**.
2. Menu **Reports → Report Utilization** → **OK**.
3. In the report, open **Memory**. **Block RAM Tile** must be **≥ 0.5** (1 RAMB18 or 1 RAMB36). Screenshot it.

### Part R: Collect the .bit and .hwh files

1. Make a folder, e.g. `C:/fpga_exam/q6_upload`.
2. Copy `C:/fpga_exam/q6_audio/q6_audio.runs/impl_1/design_1_wrapper.bit` into it and rename the copy **`q6.bit`**.
3. Copy `C:/fpga_exam/q6_audio/q6_audio.gen/sources_1/bd/design_1/hw_handoff/design_1.hwh` into it and rename the copy **`q6.hwh`**. In Vivado 2020.1 or older, the path uses `.srcs` instead of `.gen`.
4. Copy `q06/q6_audio.py` into the same folder.
5. The names must match: `q6.bit` + `q6.hwh`. The script loads `Overlay("q6.bit")`.

### Part S: Boot the PYNQ board and open Jupyter

1. Insert the PYNQ SD card. Set the boot-mode jumper to **SD**. Connect the Ethernet cable from the board to the PC (or the lab switch). Connect the micro-USB for power.
2. Switch the board **ON** and wait about 1 minute until boot finishes (the LEDs flash when it's ready).
3. If the cable goes straight into the PC, set the PC's Ethernet IPv4 to **192.168.2.1**, mask **255.255.255.0**: Control Panel → Network Connections → Ethernet → Properties → IPv4.
4. Open a browser at **http://192.168.2.99:9090** (use the lab's board IP if different). Password: **xilinx**.

### Part T: Upload and run the Python script

1. In Jupyter: **New → Folder**. Tick **Untitled Folder** → **Rename** → `q6` → open the folder.
2. Click **Upload**, select `q6.bit`, `q6.hwh` and `q6_audio.py`, then click the blue **Upload** button next to each file. In JupyterLab, use the ↑ upload icon in the file browser.
3. **New → Python 3**. A notebook opens inside the same folder.
4. In the first cell type `%run q6_audio.py` and press **Shift+Enter**. Alternatively, open `q6_audio.py` and paste its contents into the cell.
5. The script loads the overlay, writes [0, 32, 64, 128, 192, 256, 384, 512] to BRAM words 0–7, sets en = 1, runs the 3 tests and prints `Test | Expected | Observed | PASS/FAIL`.
6. Check that all 3 lines say **PASS**. Screenshot the output.

### Part U: Expected results

| Test | Inputs | Expected |
|---|---|---|
| 1 Normal Playback | sample = 1000, vol = 4 | gain = 192, out = 750 |
| 2 Low Volume | sample = 2000, vol = 2 | gain = 64, out = 500 |
| 3 Loud Boost, Clipped | sample = 30000, vol = 7 | gain = 512, out = 32767 |

**If it fails:**
- All outputs 0: en not written, FCLK_CLK0 not wired to `clk`/`clkb`, or BRAM port A not connected.
- Values from the wrong word: BMG in Stand Alone mode, or *Primitives Output Register* still ticked.
- `KeyError: axi_bram_ctrl_0`: run `ol.ip_dict.keys()` and `ol.mem_dict.keys()` in a cell, or replace the `bram = MMIO(base, rng)` line with `bram = MMIO(0x40000000, 0x2000)` (your Address Editor value).
- `Overlay` error: the `.bit`/`.hwh` names differ, or they're not in the notebook's folder.

---

<a id="q7"></a>

## Q7: Threshold Event Counter over a BRAM Window

**Files:** `q07/thresh_count.v`, `q07/q7_threshold.py`

**Board connections:** Vivado part: none. Python part: PYNQ-Z2 with the PYNQ SD card, boot jumper on **SD**, Ethernet cable to the PC, micro-USB for power.

### Part A: Create the Vivado project

1. Open **Vivado**. On the Quick Start page click **Create Project**, then **Next**.
2. Project name: `q7_threshold`. Project location: a folder with no spaces, e.g. `C:/fpga_exam`. Keep **Create project subdirectory** ticked. Click **Next**.
3. Select **RTL Project** and tick **Do not specify sources at this time**. Click **Next**.
4. Click the **Boards** tab, type `pynq` in **Search** and select **PYNQ-Z2**. Click **Next**. If PYNQ-Z2 isn't listed, click the **Parts** tab, search `xc7z020clg400-1`, and select it.
5. Click **Finish**. The project opens.

### Part B: Add the Verilog file

1. Flow Navigator → PROJECT MANAGER → **Add Sources**.
2. Select **Add or create design sources** and click **Next**.
3. Click **Add Files**, browse to `q07/thresh_count.v`, and click **OK**.
4. Tick **Copy sources into project** and click **Finish**.
5. Check that `thresh_count` appears under **Sources → Design Sources**.

### Part C: Create the block design

1. Flow Navigator → IP INTEGRATOR → **Create Block Design**.
2. Design name: `design_1`. Click **OK**. An empty **Diagram** opens.

### Part D: Add the Zynq PS and set the 50 MHz PL clock

1. **How to add an IP:** click **+** (Add IP) on the Diagram toolbar or press **Ctrl+I**, type the name in **Search**, and double-click the result. **How to rename:** click the block, change **Name** in the **Block Properties** panel, and press **Enter**. **How to configure:** double-click the block.
2. Add **ZYNQ7 Processing System** (`processing_system7_0`).
3. Click **Run Block Automation** (green banner). Tick `processing_system7_0`, keep **Apply Board Preset** ticked, and click **OK**. DDR and FIXED_IO become external.
4. Double-click `processing_system7_0`. Go to **Clock Configuration → PL Fabric Clocks** and set **FCLK_CLK0 Requested Frequency (MHz) = 50**.
5. Check **PS-PL Configuration → AXI Non Secure Enablement → GP Master AXI Interface → M AXI GP0 interface** is ticked (the board preset ticks it). Click **OK**.

### Part E: Add the AXI GPIO IPs

1. Add **AXI GPIO**. Keep the name `axi_gpio_0`. Double-click it. In the **IP Configuration** tab:
   - **GPIO:** tick **All Outputs**, GPIO Width = **1**  (en)
   - Tick **Enable Dual Channel**
   - **GPIO 2:** tick **All Outputs**, GPIO Width = **32**  (threshold)
   - **Board** tab: GPIO (and GPIO2) = **Custom**. Click **OK**.
2. Add **AXI GPIO**. Keep the name `axi_gpio_1`. Double-click it. In the **IP Configuration** tab:
   - **GPIO:** tick **All Inputs**, GPIO Width = **32**  (count_above)
   - Tick **Enable Dual Channel**
   - **GPIO 2:** tick **All Inputs**, GPIO Width = **32**  (sum)
   - **Board** tab: GPIO (and GPIO2) = **Custom**. Click **OK**.

### Part F: Add the AXI BRAM Controller

1. Add **AXI BRAM Controller**. Keep the name `axi_bram_ctrl_0`. Double-click it.
2. AXI Protocol **AXI4**, Data Width **32**, **Number of BRAM interfaces = 1**. Click **OK**.

### Part G: Add the Block Memory Generator

1. Add **Block Memory Generator**. Keep the name `blk_mem_gen_0`. Double-click it.
2. **Basic** tab: Mode = **BRAM Controller**, Memory Type = **True Dual Port RAM**.
3. **Port B Options** tab: **untick Primitives Output Register** (gives a 1-cycle read latency, which the Verilog expects). Enable Port Type = **Use ENB Pin**.
4. Click **OK**. Leave Mode as **BRAM Controller**, not Stand Alone, because the Verilog uses byte addresses (word i at 4 × i).

### Part H: Connect BRAM controller to BRAM port A

1. Drag from `axi_bram_ctrl_0/BRAM_PORTA` to `blk_mem_gen_0/BRAM_PORTA` (interface-to-interface connection).

### Part I: Run Connection Automation (AXI only)

1. Click **Run Connection Automation** (green banner).
2. Tick **only**: `axi_bram_ctrl_0/S_AXI`, `axi_gpio_0/S_AXI`, `axi_gpio_1/S_AXI`.
3. **Untick** every `GPIO` / `GPIO2` entry and `blk_mem_gen_0/BRAM_PORTB`. If those are ticked, Vivado makes them board pins or adds a second controller.
4. Click **OK**. Vivado adds `ps7_0_axi_periph` (AXI Interconnect) and `rst_ps7_0_50M`, and connects the AXI clocks and resets.

### Part J: Add constants for BRAM port B control pins

1. Add **Constant**. Rename it `const_rstb`. Configure: Const Width **1**, Const Val **0**. Click **OK**.
2. Add **Constant**. Rename it `const_web`. Configure: Const Width **4**, Const Val **0**. Click **OK**.

### Part K: Add the RTL module

1. Right-click an empty area of the Diagram and choose **Add Module**.
2. Select **thresh_count** and click **OK**. Block `thresh_count_0` appears.

### Part L: Connect BRAM port B to the module

1. On `blk_mem_gen_0`, click the **+** next to `BRAM_PORTB` to show its pins.
2. `thresh_count_0/bram_addr[31:0]` → `blk_mem_gen_0/addrb[31:0]`
3. `thresh_count_0/bram_en` → `blk_mem_gen_0/enb`
4. `blk_mem_gen_0/doutb[31:0]` → `thresh_count_0/bram_dout[31:0]`
5. `const_rstb/dout` → `blk_mem_gen_0/rstb`
6. `const_web/dout` → `blk_mem_gen_0/web[3:0]`
7. `processing_system7_0/FCLK_CLK0` → `blk_mem_gen_0/clkb` (drag `clkb` onto the FCLK_CLK0 wire).
8. Leave `dinb` unconnected.

### Part M: Connect the AXI GPIOs and clock to the module

1. On `axi_gpio_0`, click **+** next to `GPIO` and next to `GPIO2` to show the pins.
2. `axi_gpio_0/gpio_io_o` → `thresh_count_0/en`  (PS → PL, 1-bit)
3. `axi_gpio_0/gpio2_io_o` → `thresh_count_0/threshold`  (PS → PL, 32-bit)
4. On `axi_gpio_1`, click **+** next to `GPIO` and next to `GPIO2` to show the pins.
5. `thresh_count_0/count_above` → `axi_gpio_1/gpio_io_i`  (PL → PS, 32-bit)
6. `thresh_count_0/sum` → `axi_gpio_1/gpio2_io_i`  (PL → PS, 32-bit)
7. `processing_system7_0/FCLK_CLK0` → `thresh_count_0/clk` (drag `clk` onto the FCLK_CLK0 wire).
8. Click **Regenerate Layout** (⟳), then **Ctrl+S**.

### Part N: Check addresses

1. Open the **Address Editor** tab (next to Diagram).
2. If anything is under **Unassigned**, right-click it and choose **Assign All**.
3. Note the base address of `axi_bram_ctrl_0` (usually **0x4000_0000**, range 8K). The Python script finds it automatically; this is the fallback value.

### Part O: Validate and create the HDL wrapper

1. Click **Validate Design** (the ✓ icon on the Diagram toolbar, or press **F6**). Expect **Validation successful**. Warnings are fine; errors must be fixed.
2. Press **Ctrl+S** to save the block design.
3. In **Sources** (Hierarchy tab), right-click **design_1 (design_1.bd)** and choose **Create HDL Wrapper**.
4. Choose **Let Vivado manage wrapper and auto-update** and click **OK**. `design_1_wrapper` becomes the top module (shown in bold).

### Part P: Generate the bitstream (no constraints file needed: the clock comes from the PS)

1. Flow Navigator → PROGRAM AND DEBUG → **Generate Bitstream**.
2. If asked to save, click **Save**. When asked to run synthesis and implementation, click **Yes**, then **OK**.
3. Wait for **Bitstream Generation successfully completed**. Choose **View Reports** (or Cancel) and click **OK**.

### Part Q: Confirm BRAM is used (utilization report)

1. Flow Navigator → IMPLEMENTATION → **Open Implemented Design**.
2. Menu **Reports → Report Utilization** → **OK**.
3. In the report, open **Memory**. **Block RAM Tile** must be **≥ 0.5** (1 RAMB18 or 1 RAMB36). Screenshot it.

### Part R: Collect the .bit and .hwh files

1. Make a folder, e.g. `C:/fpga_exam/q7_upload`.
2. Copy `C:/fpga_exam/q7_threshold/q7_threshold.runs/impl_1/design_1_wrapper.bit` into it and rename the copy **`q7.bit`**.
3. Copy `C:/fpga_exam/q7_threshold/q7_threshold.gen/sources_1/bd/design_1/hw_handoff/design_1.hwh` into it and rename the copy **`q7.hwh`**. In Vivado 2020.1 or older, the path uses `.srcs` instead of `.gen`.
4. Copy `q07/q7_threshold.py` into the same folder.
5. The names must match: `q7.bit` + `q7.hwh`. The script loads `Overlay("q7.bit")`.

### Part S: Boot the PYNQ board and open Jupyter

1. Insert the PYNQ SD card. Set the boot-mode jumper to **SD**. Connect the Ethernet cable from the board to the PC (or the lab switch). Connect the micro-USB for power.
2. Switch the board **ON** and wait about 1 minute until boot finishes (the LEDs flash when it's ready).
3. If the cable goes straight into the PC, set the PC's Ethernet IPv4 to **192.168.2.1**, mask **255.255.255.0**: Control Panel → Network Connections → Ethernet → Properties → IPv4.
4. Open a browser at **http://192.168.2.99:9090** (use the lab's board IP if different). Password: **xilinx**.

### Part T: Upload and run the Python script

1. In Jupyter: **New → Folder**. Tick **Untitled Folder** → **Rename** → `q7` → open the folder.
2. Click **Upload**, select `q7.bit`, `q7.hwh` and `q7_threshold.py`, then click the blue **Upload** button next to each file. In JupyterLab, use the ↑ upload icon in the file browser.
3. **New → Python 3**. A notebook opens inside the same folder.
4. In the first cell type `%run q7_threshold.py` and press **Shift+Enter**. Alternatively, open `q7_threshold.py` and paste its contents into the cell.
5. The script loads the overlay, writes the 8 samples of each test case to BRAM words 0–7, sets en = 1, runs the 3 tests and prints `Test | Expected | Observed | PASS/FAIL`.
6. Check that all 3 lines say **PASS**. Screenshot the output.

### Part U: Expected results

| Test | Inputs | Expected |
|---|---|---|
| 1 Large Values | [5, 12, 7, 20, 15, 3, 18, 9], threshold = 10 | count_above = 4, sum = 89 |
| 2 Small Values with a Zero | [4, 9, 2, 1, 15, 6, 0, 3], threshold = 2 | count_above = 5, sum = 40 |
| 3 Equal to Threshold | [100 × 8], threshold = 100 | count_above = 0, sum = 800 |

**If it fails:**
- All outputs 0: en not written, FCLK_CLK0 not wired to `clk`/`clkb`, or BRAM port A not connected.
- Values from the wrong word: BMG in Stand Alone mode, or *Primitives Output Register* still ticked.
- `KeyError: axi_bram_ctrl_0`: run `ol.ip_dict.keys()` and `ol.mem_dict.keys()` in a cell, or replace the `bram = MMIO(base, rng)` line with `bram = MMIO(0x40000000, 0x2000)` (your Address Editor value).
- `Overlay` error: the `.bit`/`.hwh` names differ, or they're not in the notebook's folder.

---

<a id="q8"></a>

## Q8: Nibble Substitution Engine

**Files:** `q08/nibble_sub.v`, `q08/q8_nibble.py`

**Board connections:** Vivado part: none. Python part: PYNQ-Z2 with the PYNQ SD card, boot jumper on **SD**, Ethernet cable to the PC, micro-USB for power.

### Part A: Create the Vivado project

1. Open **Vivado**. On the Quick Start page click **Create Project**, then **Next**.
2. Project name: `q8_nibble`. Project location: a folder with no spaces, e.g. `C:/fpga_exam`. Keep **Create project subdirectory** ticked. Click **Next**.
3. Select **RTL Project** and tick **Do not specify sources at this time**. Click **Next**.
4. Click the **Boards** tab, type `pynq` in **Search** and select **PYNQ-Z2**. Click **Next**. If PYNQ-Z2 isn't listed, click the **Parts** tab, search `xc7z020clg400-1`, and select it.
5. Click **Finish**. The project opens.

### Part B: Add the Verilog file

1. Flow Navigator → PROJECT MANAGER → **Add Sources**.
2. Select **Add or create design sources** and click **Next**.
3. Click **Add Files**, browse to `q08/nibble_sub.v`, and click **OK**.
4. Tick **Copy sources into project** and click **Finish**.
5. Check that `nibble_sub` appears under **Sources → Design Sources**.

### Part C: Create the block design

1. Flow Navigator → IP INTEGRATOR → **Create Block Design**.
2. Design name: `design_1`. Click **OK**. An empty **Diagram** opens.

### Part D: Add the Zynq PS and set the 50 MHz PL clock

1. **How to add an IP:** click **+** (Add IP) on the Diagram toolbar or press **Ctrl+I**, type the name in **Search**, and double-click the result. **How to rename:** click the block, change **Name** in the **Block Properties** panel, and press **Enter**. **How to configure:** double-click the block.
2. Add **ZYNQ7 Processing System** (`processing_system7_0`).
3. Click **Run Block Automation** (green banner). Tick `processing_system7_0`, keep **Apply Board Preset** ticked, and click **OK**. DDR and FIXED_IO become external.
4. Double-click `processing_system7_0`. Go to **Clock Configuration → PL Fabric Clocks** and set **FCLK_CLK0 Requested Frequency (MHz) = 50**.
5. Check **PS-PL Configuration → AXI Non Secure Enablement → GP Master AXI Interface → M AXI GP0 interface** is ticked (the board preset ticks it). Click **OK**.

### Part E: Add the AXI GPIO IPs

1. Add **AXI GPIO**. Keep the name `axi_gpio_0`. Double-click it. In the **IP Configuration** tab:
   - **GPIO:** tick **All Outputs**, GPIO Width = **1**  (en)
   - Tick **Enable Dual Channel**
   - **GPIO 2:** tick **All Outputs**, GPIO Width = **32**  (x)
   - **Board** tab: GPIO (and GPIO2) = **Custom**. Click **OK**.
2. Add **AXI GPIO**. Keep the name `axi_gpio_1`. Double-click it. In the **IP Configuration** tab:
   - **GPIO:** tick **All Inputs**, GPIO Width = **32**  (y)
   - Leave **Enable Dual Channel** unticked
   - **Board** tab: GPIO (and GPIO2) = **Custom**. Click **OK**.

### Part F: Add the AXI BRAM Controller

1. Add **AXI BRAM Controller**. Keep the name `axi_bram_ctrl_0`. Double-click it.
2. AXI Protocol **AXI4**, Data Width **32**, **Number of BRAM interfaces = 1**. Click **OK**.

### Part G: Add the Block Memory Generator

1. Add **Block Memory Generator**. Keep the name `blk_mem_gen_0`. Double-click it.
2. **Basic** tab: Mode = **BRAM Controller**, Memory Type = **True Dual Port RAM**.
3. **Port B Options** tab: **untick Primitives Output Register** (gives a 1-cycle read latency, which the Verilog expects). Enable Port Type = **Use ENB Pin**.
4. Click **OK**. Leave Mode as **BRAM Controller**, not Stand Alone, because the Verilog uses byte addresses (word i at 4 × i).

### Part H: Connect BRAM controller to BRAM port A

1. Drag from `axi_bram_ctrl_0/BRAM_PORTA` to `blk_mem_gen_0/BRAM_PORTA` (interface-to-interface connection).

### Part I: Run Connection Automation (AXI only)

1. Click **Run Connection Automation** (green banner).
2. Tick **only**: `axi_bram_ctrl_0/S_AXI`, `axi_gpio_0/S_AXI`, `axi_gpio_1/S_AXI`.
3. **Untick** every `GPIO` / `GPIO2` entry and `blk_mem_gen_0/BRAM_PORTB`. If those are ticked, Vivado makes them board pins or adds a second controller.
4. Click **OK**. Vivado adds `ps7_0_axi_periph` (AXI Interconnect) and `rst_ps7_0_50M`, and connects the AXI clocks and resets.

### Part J: Add constants for BRAM port B control pins

1. Add **Constant**. Rename it `const_rstb`. Configure: Const Width **1**, Const Val **0**. Click **OK**.
2. Add **Constant**. Rename it `const_web`. Configure: Const Width **4**, Const Val **0**. Click **OK**.

### Part K: Add the RTL module

1. Right-click an empty area of the Diagram and choose **Add Module**.
2. Select **nibble_sub** and click **OK**. Block `nibble_sub_0` appears.

### Part L: Connect BRAM port B to the module

1. On `blk_mem_gen_0`, click the **+** next to `BRAM_PORTB` to show its pins.
2. `nibble_sub_0/bram_addr[31:0]` → `blk_mem_gen_0/addrb[31:0]`
3. `nibble_sub_0/bram_en` → `blk_mem_gen_0/enb`
4. `blk_mem_gen_0/doutb[31:0]` → `nibble_sub_0/bram_dout[31:0]`
5. `const_rstb/dout` → `blk_mem_gen_0/rstb`
6. `const_web/dout` → `blk_mem_gen_0/web[3:0]`
7. `processing_system7_0/FCLK_CLK0` → `blk_mem_gen_0/clkb` (drag `clkb` onto the FCLK_CLK0 wire).
8. Leave `dinb` unconnected.

### Part M: Connect the AXI GPIOs and clock to the module

1. On `axi_gpio_0`, click **+** next to `GPIO` and next to `GPIO2` to show the pins.
2. `axi_gpio_0/gpio_io_o` → `nibble_sub_0/en`  (PS → PL, 1-bit)
3. `axi_gpio_0/gpio2_io_o` → `nibble_sub_0/x`  (PS → PL, 32-bit)
4. On `axi_gpio_1`, click **+** next to `GPIO` to show the pins.
5. `nibble_sub_0/y` → `axi_gpio_1/gpio_io_i`  (PL → PS, 32-bit)
6. `processing_system7_0/FCLK_CLK0` → `nibble_sub_0/clk` (drag `clk` onto the FCLK_CLK0 wire).
7. Click **Regenerate Layout** (⟳), then **Ctrl+S**.

### Part N: Check addresses

1. Open the **Address Editor** tab (next to Diagram).
2. If anything is under **Unassigned**, right-click it and choose **Assign All**.
3. Note the base address of `axi_bram_ctrl_0` (usually **0x4000_0000**, range 8K). The Python script finds it automatically; this is the fallback value.

### Part O: Validate and create the HDL wrapper

1. Click **Validate Design** (the ✓ icon on the Diagram toolbar, or press **F6**). Expect **Validation successful**. Warnings are fine; errors must be fixed.
2. Press **Ctrl+S** to save the block design.
3. In **Sources** (Hierarchy tab), right-click **design_1 (design_1.bd)** and choose **Create HDL Wrapper**.
4. Choose **Let Vivado manage wrapper and auto-update** and click **OK**. `design_1_wrapper` becomes the top module (shown in bold).

### Part P: Generate the bitstream (no constraints file needed: the clock comes from the PS)

1. Flow Navigator → PROGRAM AND DEBUG → **Generate Bitstream**.
2. If asked to save, click **Save**. When asked to run synthesis and implementation, click **Yes**, then **OK**.
3. Wait for **Bitstream Generation successfully completed**. Choose **View Reports** (or Cancel) and click **OK**.

### Part Q: Confirm BRAM is used (utilization report)

1. Flow Navigator → IMPLEMENTATION → **Open Implemented Design**.
2. Menu **Reports → Report Utilization** → **OK**.
3. In the report, open **Memory**. **Block RAM Tile** must be **≥ 0.5** (1 RAMB18 or 1 RAMB36). Screenshot it.

### Part R: Collect the .bit and .hwh files

1. Make a folder, e.g. `C:/fpga_exam/q8_upload`.
2. Copy `C:/fpga_exam/q8_nibble/q8_nibble.runs/impl_1/design_1_wrapper.bit` into it and rename the copy **`q8.bit`**.
3. Copy `C:/fpga_exam/q8_nibble/q8_nibble.gen/sources_1/bd/design_1/hw_handoff/design_1.hwh` into it and rename the copy **`q8.hwh`**. In Vivado 2020.1 or older, the path uses `.srcs` instead of `.gen`.
4. Copy `q08/q8_nibble.py` into the same folder.
5. The names must match: `q8.bit` + `q8.hwh`. The script loads `Overlay("q8.bit")`.

### Part S: Boot the PYNQ board and open Jupyter

1. Insert the PYNQ SD card. Set the boot-mode jumper to **SD**. Connect the Ethernet cable from the board to the PC (or the lab switch). Connect the micro-USB for power.
2. Switch the board **ON** and wait about 1 minute until boot finishes (the LEDs flash when it's ready).
3. If the cable goes straight into the PC, set the PC's Ethernet IPv4 to **192.168.2.1**, mask **255.255.255.0**: Control Panel → Network Connections → Ethernet → Properties → IPv4.
4. Open a browser at **http://192.168.2.99:9090** (use the lab's board IP if different). Password: **xilinx**.

### Part T: Upload and run the Python script

1. In Jupyter: **New → Folder**. Tick **Untitled Folder** → **Rename** → `q8` → open the folder.
2. Click **Upload**, select `q8.bit`, `q8.hwh` and `q8_nibble.py`, then click the blue **Upload** button next to each file. In JupyterLab, use the ↑ upload icon in the file browser.
3. **New → Python 3**. A notebook opens inside the same folder.
4. In the first cell type `%run q8_nibble.py` and press **Shift+Enter**. Alternatively, open `q8_nibble.py` and paste its contents into the cell.
5. The script loads the overlay, writes [12, 5, 6, 11, 9, 0, 10, 13, 3, 14, 15, 8, 4, 7, 1, 2] to BRAM words 0–15, sets en = 1, runs the 3 tests and prints `Test | Expected | Observed | PASS/FAIL`.
6. Check that all 3 lines say **PASS**. Screenshot the output.

### Part U: Expected results

| Test | Inputs | Expected |
|---|---|---|
| 1 All Zeros | x = 0x00000000 | y = 0xCCCCCCCC |
| 2 Ascending Nibbles | x = 0x12345678 | y = 0x56B90AD3 |
| 3 Descending Nibbles | x = 0xFEDCBA98 | y = 0x21748FE3 |

**If it fails:**
- All outputs 0: en not written, FCLK_CLK0 not wired to `clk`/`clkb`, or BRAM port A not connected.
- Values from the wrong word: BMG in Stand Alone mode, or *Primitives Output Register* still ticked.
- `KeyError: axi_bram_ctrl_0`: run `ol.ip_dict.keys()` and `ol.mem_dict.keys()` in a cell, or replace the `bram = MMIO(base, rng)` line with `bram = MMIO(0x40000000, 0x2000)` (your Address Editor value).
- `Overlay` error: the `.bit`/`.hwh` names differ, or they're not in the notebook's folder.

---

<a id="q9"></a>

## Q9: Crash & Fall Detection Coprocessor

**Files:** `q09/crash_detect.m` (MATLAB function code), `q09/build_q9_model.m` + `q09/make_hdl_model.m` (make the Simulink file `q9_crash.slx`), `q09/q9_main.c` (Vitis C code)

**Board connections:** PYNQ-Z2, micro-USB cable to the PC, boot jumper on **JTAG**.

### Part A: Get the Simulink model ready (MATLAB)

1. Copy the folder `q09` to the lab PC, e.g. `C:/fpga_exam/q09` (no spaces in the path).
2. Open **MATLAB**. In the **Current Folder** bar, browse to `C:/fpga_exam/q09`.
3. If `q9_crash.slx` is **not** in the folder yet: in the Command Window type `build_q9_model` and press **Enter**. Wait for `Saved q9_crash.slx`. (Warnings saying `... not set` are OK; set those items by hand in Part C.)
4. If MATLAB can't find Vivado later (Part C), run this once in the Command Window, with your Vivado version and path:
   ```
   hdlsetuptoolpath('ToolName','Xilinx Vivado','ToolPath','C:\Xilinx\Vivado\2022.2\bin\vivado.bat')
   ```
5. Double-click `q9_crash.slx` to open it. You'll see Constant blocks (Gx_in, Gy_in) → subsystem **`q9_dut`** → Display blocks.
6. Double-click `q9_dut` to look inside. It contains Inports → **one MATLAB Function block** (`crash_detect`) → Outports. Double-click the MATLAB Function block to see the code (the same as `crash_detect.m`). Go back to the top level.

### Part B: Simulate in Simulink

1. On the **Simulation** tab, click **Run** (green ▶).
2. The Display blocks show **impact = 800, alert = 0** (test case 1).
3. Optional: double-click the Constant blocks to try the other cases, then click Run: Gx_in = -1800, Gy_in = 1200 gives impact = 3000, alert = 1; Gx_in = 5200, Gy_in = -3400 gives impact = 8600, alert = 2. Put the values back to test case 1 afterwards.
4. Press **Ctrl+S** to save the model.

### Part C: Generate the AXI4-Lite IP core (HDL Workflow Advisor)

1. Right-click the subsystem **`q9_dut`** → **HDL Code → HDL Workflow Advisor**.
2. Select **1.1 Set Target Device and Synthesis Tool** and check (fix if different):
   - Target workflow **IP Core Generation**
   - Target platform **Generic Xilinx Platform**
   - Synthesis tool **Xilinx Vivado**
   - Family **Zynq**, Device **xc7z020**, Package **clg400**, Speed **-1**
   - Project folder `hdl_prj`
   Click **Run This Task**.
3. If **1.2 Set Target Reference Design** appears, keep the defaults and click **Run This Task**.
4. Select **1.3 Set Target Interface**. In the table, every port's **Target Platform Interfaces** = **AXI4-Lite**, with addresses **Gx = 0x100, Gy = 0x104, impact = 0x108, alert = 0x10C**. Click **Run This Task**.
5. Select **1.4 Set Target Frequency**: Target Frequency (MHz) = **50**. Click **Run This Task**.
6. Right-click **3.2 Generate RTL Code and IP Core** and choose **Run to Selected Task**. It runs steps 2.x, 3.1 and 3.2 in order.
7. If a step in folder 2 fails and shows a **Modify Settings** button, click it, then right-click 3.2 → **Run to Selected Task** again.
8. When 3.2 passes, click the **IP Core Report** link. In **Register Address Mapping**, check Gx = 0x100, Gy = 0x104, impact = 0x108, alert = 0x10C. These must match the `#define`s at the top of `q9_main.c`.
9. The IP is in `C:/fpga_exam/q09/hdl_prj/ipcore/q9_dut_ip_v1_0`.

### Part D: Create the Vivado project

1. Open **Vivado**. On the Quick Start page click **Create Project**, then **Next**.
2. Project name: `q9_crash_vivado`. Project location: `C:/fpga_exam/q09`. Keep **Create project subdirectory** ticked. Click **Next**.
3. Select **RTL Project** and tick **Do not specify sources at this time**. Click **Next**.
4. Click the **Boards** tab, search `pynq` and select **PYNQ-Z2**. Click **Next**. If it isn't listed: **Parts** tab → `xc7z020clg400-1`.
5. Click **Finish**.

### Part E: Add the HDL Coder IP repository

1. Flow Navigator → PROJECT MANAGER → **Settings**.
2. Go to **Project Settings → IP → Repository** and click **+**.
3. Browse to `C:/fpga_exam/q09/hdl_prj/ipcore` and click **Select**. A message lists **1 IP** (`q9_dut_ip`). Click **OK**, then **OK** again.

### Part F: Block design: Zynq PS + your IP

1. Flow Navigator → IP INTEGRATOR → **Create Block Design**. Name it `design_1` and click **OK**.
2. **How to add an IP:** click **+** (Add IP) on the Diagram toolbar or press **Ctrl+I**, type the name in **Search**, and double-click the result. **How to rename:** click the block, change **Name** in the **Block Properties** panel, and press **Enter**. **How to configure:** double-click the block.
3. Add **ZYNQ7 Processing System**. Click **Run Block Automation**, keep **Apply Board Preset** ticked, and click **OK**.
4. Double-click `processing_system7_0`. Go to **Clock Configuration → PL Fabric Clocks** and set **FCLK_CLK0 = 50** MHz. Click **OK**.
5. Add IP: search **q9_dut_ip** and double-click it. Block `q9_dut_ip_0` appears.
6. Click **Run Connection Automation**. Tick **All Automation** and click **OK**. This adds the AXI Interconnect and Processor System Reset and connects `AXI4_Lite` and its clock/reset.
7. Check the remaining pins on `q9_dut_ip_0`. If **IPCORE_CLK** is unconnected, drag it onto the **FCLK_CLK0** wire. If **IPCORE_RESETN** is unconnected, drag it onto `rst_ps7_0_50M/peripheral_aresetn`.
8. Click **Regenerate Layout** (⟳), then **Validate Design** (F6). Expect **Validation successful**. Press **Ctrl+S**.
9. Open the **Address Editor** tab and note the IP's base address (usually **0x43C0_0000**).

### Part G: Wrapper and bitstream

1. In **Sources**, right-click **design_1 (design_1.bd)** → **Create HDL Wrapper** → **Let Vivado manage wrapper and auto-update** → **OK**.
2. Flow Navigator → PROGRAM AND DEBUG → **Generate Bitstream**. Click **Yes** (run synthesis and implementation), then **OK**. No constraints file is needed: the clock comes from the PS.
3. Wait for **Bitstream Generation successfully completed**, then click **Cancel** on the dialog.

### Part H: Export the hardware (.xsa)

1. Menu **File → Export → Export Hardware** → **Next**.
2. Select **Include bitstream** and click **Next**.
3. XSA file name: `q9_crash_hw`. Export to: `C:/fpga_exam/q09`. Click **Next**, then **Finish**.

### Part I: Connect the board and open MobaXterm

1. Set the PYNQ-Z2 boot-mode jumper to **JTAG**. Connect the micro-USB (PROG/UART) to the PC. Switch the board **ON**.
2. Find the COM port: Windows **Device Manager → Ports (COM & LPT) → USB Serial Port (COMx)**.
3. Open **MobaXterm**: **Session → Serial**. Serial port = **COMx**, Speed = **115200**. Click **OK**. Leave this window open.

### Part J: Vitis: create the application

1. Open **Vitis IDE** (Windows Start, or Vivado **Tools → Launch Vitis IDE**). Workspace: `C:/fpga_exam/q09/vitis_ws` → **Launch**.
2. **File → New → Application Project** → **Next**.
3. Choose the **Create a new platform from hardware (XSA)** tab → **Browse** → `C:/fpga_exam/q09/q9_crash_hw.xsa` → **Next**.
4. Application project name: `q9_crash_app` → **Next**.
5. Domain: Operating System **standalone**, Processor **ps7_cortexa9_0** → **Next**.
6. Template: **Hello World** → **Finish**. Wait for the platform to build.
7. **If you have the new unified Vitis (2023.2 or later):** use **File → New Component → Platform** (pick the .xsa, standalone, ps7_cortexa9_0) and Build it. Then use **Examples → Hello World → Create Application Component from Template** with that platform.

### Part K: Put in the C code and build

1. In the Explorer, expand `q9_crash_app → src` and double-click **helloworld.c**.
2. Press **Ctrl+A**, then **Delete**. Open `q09/q9_main.c` in Notepad, copy everything, paste it into helloworld.c, and press **Ctrl+S**.
3. Check the `#define` offsets at the top match the IP Core Report (Gx = 0x100, Gy = 0x104, impact = 0x108, alert = 0x10C). `IP_BASE` uses `XPAR_Q9_DUT_IP_0_BASEADDR` if Vitis defines it; otherwise it uses `0x43C00000`. If your Address Editor showed a different base, change that number.
4. Right-click `q9_crash_app` → **Build Project**. The Console must end with **Build Finished** and no errors.

### Part L: Run on the board and check MobaXterm

1. Right-click `q9_crash_app` → **Run As → Launch Hardware** (older versions: *Launch on Hardware (Single Application Debug)*). This programs the FPGA, runs ps7_init and starts the program.
2. Look at MobaXterm. It prints one line per test: `Test | Inputs | Expected | Observed | PASS`, then **3/3 PASS**.
3. If nothing prints: check the COM port/baud, then use Vitis **Xilinx → Program Device → Program** and Run again.
4. If values print but are 0 or wrong: the base address or offsets don't match (check the Address Editor and the IP Core Report).
5. Screenshot MobaXterm.

### Part M: Expected results

| Test | Inputs | Expected |
|---|---|---|
| 1 Routine Motion | Gx = 350, Gy = −450 | impact = 800, alert = 0 |
| 2 Trip & Hard Fall | Gx = −1800, Gy = 1200 | impact = 3000, alert = 1 |
| 3 High-G Car Crash | Gx = 5200, Gy = −3400 | impact = 8600, alert = 2 |

**Fallback if the `.slx` can't be used (build it by hand):** new blank model → add a Subsystem named `q9_dut`. Inside it, delete the In1→Out1 line and add one **MATLAB Function** block. Paste the contents of `crash_detect.m` into it. Name the Inports Gx, Gy and set each one's data type to **int32**. Name the Outports impact, alert. Wire Inports → function → Outports. At the top level, add int32 **Constant** blocks (Sample time 1) on the inputs and **Display** blocks on the outputs. Model Settings → Solver → **Fixed-step, discrete**. Run `hdlsetup('<model name>')` in the Command Window, save, and continue with Part B.

---

<a id="q10"></a>

## Q10: PCIe/CXL SerDes Link Health & Retimer Engine

**Files:** `q10/link_health.m` (MATLAB function code), `q10/build_q10_model.m` + `q10/make_hdl_model.m` (make the Simulink file `q10_link.slx`), `q10/q10_main.c` (Vitis C code)

**Board connections:** PYNQ-Z2, micro-USB cable to the PC, boot jumper on **JTAG**.

### Part A: Get the Simulink model ready (MATLAB)

1. Copy the folder `q10` to the lab PC, e.g. `C:/fpga_exam/q10` (no spaces in the path).
2. Open **MATLAB**. In the **Current Folder** bar, browse to `C:/fpga_exam/q10`.
3. If `q10_link.slx` is **not** in the folder yet: in the Command Window type `build_q10_model` and press **Enter**. Wait for `Saved q10_link.slx`. (Warnings saying `... not set` are OK; set those items by hand in Part C.)
4. If MATLAB can't find Vivado later (Part C), run this once in the Command Window, with your Vivado version and path:
   ```
   hdlsetuptoolpath('ToolName','Xilinx Vivado','ToolPath','C:\Xilinx\Vivado\2022.2\bin\vivado.bat')
   ```
5. Double-click `q10_link.slx` to open it. You'll see Constant blocks (eye_margin_in, error_count_in) → subsystem **`q10_dut`** → Display blocks.
6. Double-click `q10_dut` to look inside. It contains Inports → **one MATLAB Function block** (`link_health`) → Outports. Double-click the MATLAB Function block to see the code (the same as `link_health.m`). Go back to the top level.

### Part B: Simulate in Simulink

1. On the **Simulation** tab, click **Run** (green ▶).
2. The Display blocks show **link_score = 340, retimer_action = 0** (test case 1).
3. Optional: double-click the Constant blocks to try the other cases, then click Run: eye_margin_in = 220, error_count_in = 18 gives 130, 1; eye_margin_in = 120, error_count_in = 50 gives 0, 2. Put the values back to test case 1 afterwards.
4. Press **Ctrl+S** to save the model.

### Part C: Generate the AXI4-Lite IP core (HDL Workflow Advisor)

1. Right-click the subsystem **`q10_dut`** → **HDL Code → HDL Workflow Advisor**.
2. Select **1.1 Set Target Device and Synthesis Tool** and check (fix if different):
   - Target workflow **IP Core Generation**
   - Target platform **Generic Xilinx Platform**
   - Synthesis tool **Xilinx Vivado**
   - Family **Zynq**, Device **xc7z020**, Package **clg400**, Speed **-1**
   - Project folder `hdl_prj`
   Click **Run This Task**.
3. If **1.2 Set Target Reference Design** appears, keep the defaults and click **Run This Task**.
4. Select **1.3 Set Target Interface**. In the table, every port's **Target Platform Interfaces** = **AXI4-Lite**, with addresses **eye_margin = 0x100, error_count = 0x104, link_score = 0x108, retimer_action = 0x10C**. Click **Run This Task**.
5. Select **1.4 Set Target Frequency**: Target Frequency (MHz) = **50**. Click **Run This Task**.
6. Right-click **3.2 Generate RTL Code and IP Core** and choose **Run to Selected Task**. It runs steps 2.x, 3.1 and 3.2 in order.
7. If a step in folder 2 fails and shows a **Modify Settings** button, click it, then right-click 3.2 → **Run to Selected Task** again.
8. When 3.2 passes, click the **IP Core Report** link. In **Register Address Mapping**, check eye_margin = 0x100, error_count = 0x104, link_score = 0x108, retimer_action = 0x10C. These must match the `#define`s at the top of `q10_main.c`.
9. The IP is in `C:/fpga_exam/q10/hdl_prj/ipcore/q10_dut_ip_v1_0`.

### Part D: Create the Vivado project

1. Open **Vivado**. On the Quick Start page click **Create Project**, then **Next**.
2. Project name: `q10_link_vivado`. Project location: `C:/fpga_exam/q10`. Keep **Create project subdirectory** ticked. Click **Next**.
3. Select **RTL Project** and tick **Do not specify sources at this time**. Click **Next**.
4. Click the **Boards** tab, search `pynq` and select **PYNQ-Z2**. Click **Next**. If it isn't listed: **Parts** tab → `xc7z020clg400-1`.
5. Click **Finish**.

### Part E: Add the HDL Coder IP repository

1. Flow Navigator → PROJECT MANAGER → **Settings**.
2. Go to **Project Settings → IP → Repository** and click **+**.
3. Browse to `C:/fpga_exam/q10/hdl_prj/ipcore` and click **Select**. A message lists **1 IP** (`q10_dut_ip`). Click **OK**, then **OK** again.

### Part F: Block design: Zynq PS + your IP

1. Flow Navigator → IP INTEGRATOR → **Create Block Design**. Name it `design_1` and click **OK**.
2. **How to add an IP:** click **+** (Add IP) on the Diagram toolbar or press **Ctrl+I**, type the name in **Search**, and double-click the result. **How to rename:** click the block, change **Name** in the **Block Properties** panel, and press **Enter**. **How to configure:** double-click the block.
3. Add **ZYNQ7 Processing System**. Click **Run Block Automation**, keep **Apply Board Preset** ticked, and click **OK**.
4. Double-click `processing_system7_0`. Go to **Clock Configuration → PL Fabric Clocks** and set **FCLK_CLK0 = 50** MHz. Click **OK**.
5. Add IP: search **q10_dut_ip** and double-click it. Block `q10_dut_ip_0` appears.
6. Click **Run Connection Automation**. Tick **All Automation** and click **OK**. This adds the AXI Interconnect and Processor System Reset and connects `AXI4_Lite` and its clock/reset.
7. Check the remaining pins on `q10_dut_ip_0`. If **IPCORE_CLK** is unconnected, drag it onto the **FCLK_CLK0** wire. If **IPCORE_RESETN** is unconnected, drag it onto `rst_ps7_0_50M/peripheral_aresetn`.
8. Click **Regenerate Layout** (⟳), then **Validate Design** (F6). Expect **Validation successful**. Press **Ctrl+S**.
9. Open the **Address Editor** tab and note the IP's base address (usually **0x43C0_0000**).

### Part G: Wrapper and bitstream

1. In **Sources**, right-click **design_1 (design_1.bd)** → **Create HDL Wrapper** → **Let Vivado manage wrapper and auto-update** → **OK**.
2. Flow Navigator → PROGRAM AND DEBUG → **Generate Bitstream**. Click **Yes** (run synthesis and implementation), then **OK**. No constraints file is needed: the clock comes from the PS.
3. Wait for **Bitstream Generation successfully completed**, then click **Cancel** on the dialog.

### Part H: Export the hardware (.xsa)

1. Menu **File → Export → Export Hardware** → **Next**.
2. Select **Include bitstream** and click **Next**.
3. XSA file name: `q10_link_hw`. Export to: `C:/fpga_exam/q10`. Click **Next**, then **Finish**.

### Part I: Connect the board and open MobaXterm

1. Set the PYNQ-Z2 boot-mode jumper to **JTAG**. Connect the micro-USB (PROG/UART) to the PC. Switch the board **ON**.
2. Find the COM port: Windows **Device Manager → Ports (COM & LPT) → USB Serial Port (COMx)**.
3. Open **MobaXterm**: **Session → Serial**. Serial port = **COMx**, Speed = **115200**. Click **OK**. Leave this window open.

### Part J: Vitis: create the application

1. Open **Vitis IDE** (Windows Start, or Vivado **Tools → Launch Vitis IDE**). Workspace: `C:/fpga_exam/q10/vitis_ws` → **Launch**.
2. **File → New → Application Project** → **Next**.
3. Choose the **Create a new platform from hardware (XSA)** tab → **Browse** → `C:/fpga_exam/q10/q10_link_hw.xsa` → **Next**.
4. Application project name: `q10_link_app` → **Next**.
5. Domain: Operating System **standalone**, Processor **ps7_cortexa9_0** → **Next**.
6. Template: **Hello World** → **Finish**. Wait for the platform to build.
7. **If you have the new unified Vitis (2023.2 or later):** use **File → New Component → Platform** (pick the .xsa, standalone, ps7_cortexa9_0) and Build it. Then use **Examples → Hello World → Create Application Component from Template** with that platform.

### Part K: Put in the C code and build

1. In the Explorer, expand `q10_link_app → src` and double-click **helloworld.c**.
2. Press **Ctrl+A**, then **Delete**. Open `q10/q10_main.c` in Notepad, copy everything, paste it into helloworld.c, and press **Ctrl+S**.
3. Check the `#define` offsets at the top match the IP Core Report (eye_margin = 0x100, error_count = 0x104, link_score = 0x108, retimer_action = 0x10C). `IP_BASE` uses `XPAR_Q10_DUT_IP_0_BASEADDR` if Vitis defines it; otherwise it uses `0x43C00000`. If your Address Editor showed a different base, change that number.
4. Right-click `q10_link_app` → **Build Project**. The Console must end with **Build Finished** and no errors.

### Part L: Run on the board and check MobaXterm

1. Right-click `q10_link_app` → **Run As → Launch Hardware** (older versions: *Launch on Hardware (Single Application Debug)*). This programs the FPGA, runs ps7_init and starts the program.
2. Look at MobaXterm. It prints one line per test: `Test | Inputs | Expected | Observed | PASS`, then **3/3 PASS**.
3. If nothing prints: check the COM port/baud, then use Vitis **Xilinx → Program Device → Program** and Run again.
4. If values print but are 0 or wrong: the base address or offsets don't match (check the Address Editor and the IP Core Report).
5. Screenshot MobaXterm.

### Part M: Expected results

| Test | Inputs | Expected |
|---|---|---|
| 1 Pristine Gen5 Link | eye_margin = 350, error_count = 2 | Q = 340, action = 0 |
| 2 Thermal Jitter / Lossy Trace | eye_margin = 220, error_count = 18 | Q = 130, action = 1 |
| 3 Severe Cable Fault | eye_margin = 120, error_count = 50 | Q = 0, action = 2 |

**Fallback if the `.slx` can't be used (build it by hand):** new blank model → add a Subsystem named `q10_dut`. Inside it, delete the In1→Out1 line and add one **MATLAB Function** block. Paste the contents of `link_health.m` into it. Name the Inports eye_margin, error_count and set each one's data type to **int32**. Name the Outports link_score, retimer_action. Wire Inports → function → Outports. At the top level, add int32 **Constant** blocks (Sample time 1) on the inputs and **Display** blocks on the outputs. Model Settings → Solver → **Fixed-step, discrete**. Run `hdlsetup('<model name>')` in the Command Window, save, and continue with Part B.

---

<a id="q11"></a>

## Q11: 3-Point Median Filter

**Files:** `q11/median3.m` (MATLAB function code), `q11/build_q11_model.m` + `q11/make_hdl_model.m` (make the Simulink file `q11_median.slx`), `q11/q11_main.c` (Vitis C code)

**Board connections:** PYNQ-Z2, micro-USB cable to the PC, boot jumper on **JTAG**.

### Part A: Get the Simulink model ready (MATLAB)

1. Copy the folder `q11` to the lab PC, e.g. `C:/fpga_exam/q11` (no spaces in the path).
2. Open **MATLAB**. In the **Current Folder** bar, browse to `C:/fpga_exam/q11`.
3. If `q11_median.slx` is **not** in the folder yet: in the Command Window type `build_q11_model` and press **Enter**. Wait for `Saved q11_median.slx`. (Warnings saying `... not set` are OK; set those items by hand in Part C.)
4. If MATLAB can't find Vivado later (Part C), run this once in the Command Window, with your Vivado version and path:
   ```
   hdlsetuptoolpath('ToolName','Xilinx Vivado','ToolPath','C:\Xilinx\Vivado\2022.2\bin\vivado.bat')
   ```
5. Double-click `q11_median.slx` to open it. You'll see Constant blocks (A_in, B_in, C_in) → subsystem **`q11_dut`** → Display blocks.
6. Double-click `q11_dut` to look inside. It contains Inports → **one MATLAB Function block** (`median3`) → Outports. Double-click the MATLAB Function block to see the code (the same as `median3.m`). Go back to the top level.

### Part B: Simulate in Simulink

1. On the **Simulation** tab, click **Run** (green ▶).
2. The Display blocks show **med = 40** (test case 1).
3. Optional: double-click the Constant blocks to try the other cases, then click Run: A_in = -50, B_in = -10, C_in = -80 gives med = -50; A_in = 3300, B_in = 9999, C_in = 3320 gives med = 3320. Put the values back to test case 1 afterwards.
4. Press **Ctrl+S** to save the model.

### Part C: Generate the AXI4-Lite IP core (HDL Workflow Advisor)

1. Right-click the subsystem **`q11_dut`** → **HDL Code → HDL Workflow Advisor**.
2. Select **1.1 Set Target Device and Synthesis Tool** and check (fix if different):
   - Target workflow **IP Core Generation**
   - Target platform **Generic Xilinx Platform**
   - Synthesis tool **Xilinx Vivado**
   - Family **Zynq**, Device **xc7z020**, Package **clg400**, Speed **-1**
   - Project folder `hdl_prj`
   Click **Run This Task**.
3. If **1.2 Set Target Reference Design** appears, keep the defaults and click **Run This Task**.
4. Select **1.3 Set Target Interface**. In the table, every port's **Target Platform Interfaces** = **AXI4-Lite**, with addresses **A = 0x100, B = 0x104, C = 0x108, med = 0x10C**. Click **Run This Task**.
5. Select **1.4 Set Target Frequency**: Target Frequency (MHz) = **50**. Click **Run This Task**.
6. Right-click **3.2 Generate RTL Code and IP Core** and choose **Run to Selected Task**. It runs steps 2.x, 3.1 and 3.2 in order.
7. If a step in folder 2 fails and shows a **Modify Settings** button, click it, then right-click 3.2 → **Run to Selected Task** again.
8. When 3.2 passes, click the **IP Core Report** link. In **Register Address Mapping**, check A = 0x100, B = 0x104, C = 0x108, med = 0x10C. These must match the `#define`s at the top of `q11_main.c`.
9. The IP is in `C:/fpga_exam/q11/hdl_prj/ipcore/q11_dut_ip_v1_0`.

### Part D: Create the Vivado project

1. Open **Vivado**. On the Quick Start page click **Create Project**, then **Next**.
2. Project name: `q11_median_vivado`. Project location: `C:/fpga_exam/q11`. Keep **Create project subdirectory** ticked. Click **Next**.
3. Select **RTL Project** and tick **Do not specify sources at this time**. Click **Next**.
4. Click the **Boards** tab, search `pynq` and select **PYNQ-Z2**. Click **Next**. If it isn't listed: **Parts** tab → `xc7z020clg400-1`.
5. Click **Finish**.

### Part E: Add the HDL Coder IP repository

1. Flow Navigator → PROJECT MANAGER → **Settings**.
2. Go to **Project Settings → IP → Repository** and click **+**.
3. Browse to `C:/fpga_exam/q11/hdl_prj/ipcore` and click **Select**. A message lists **1 IP** (`q11_dut_ip`). Click **OK**, then **OK** again.

### Part F: Block design: Zynq PS + your IP

1. Flow Navigator → IP INTEGRATOR → **Create Block Design**. Name it `design_1` and click **OK**.
2. **How to add an IP:** click **+** (Add IP) on the Diagram toolbar or press **Ctrl+I**, type the name in **Search**, and double-click the result. **How to rename:** click the block, change **Name** in the **Block Properties** panel, and press **Enter**. **How to configure:** double-click the block.
3. Add **ZYNQ7 Processing System**. Click **Run Block Automation**, keep **Apply Board Preset** ticked, and click **OK**.
4. Double-click `processing_system7_0`. Go to **Clock Configuration → PL Fabric Clocks** and set **FCLK_CLK0 = 50** MHz. Click **OK**.
5. Add IP: search **q11_dut_ip** and double-click it. Block `q11_dut_ip_0` appears.
6. Click **Run Connection Automation**. Tick **All Automation** and click **OK**. This adds the AXI Interconnect and Processor System Reset and connects `AXI4_Lite` and its clock/reset.
7. Check the remaining pins on `q11_dut_ip_0`. If **IPCORE_CLK** is unconnected, drag it onto the **FCLK_CLK0** wire. If **IPCORE_RESETN** is unconnected, drag it onto `rst_ps7_0_50M/peripheral_aresetn`.
8. Click **Regenerate Layout** (⟳), then **Validate Design** (F6). Expect **Validation successful**. Press **Ctrl+S**.
9. Open the **Address Editor** tab and note the IP's base address (usually **0x43C0_0000**).

### Part G: Wrapper and bitstream

1. In **Sources**, right-click **design_1 (design_1.bd)** → **Create HDL Wrapper** → **Let Vivado manage wrapper and auto-update** → **OK**.
2. Flow Navigator → PROGRAM AND DEBUG → **Generate Bitstream**. Click **Yes** (run synthesis and implementation), then **OK**. No constraints file is needed: the clock comes from the PS.
3. Wait for **Bitstream Generation successfully completed**, then click **Cancel** on the dialog.

### Part H: Export the hardware (.xsa)

1. Menu **File → Export → Export Hardware** → **Next**.
2. Select **Include bitstream** and click **Next**.
3. XSA file name: `q11_median_hw`. Export to: `C:/fpga_exam/q11`. Click **Next**, then **Finish**.

### Part I: Connect the board and open MobaXterm

1. Set the PYNQ-Z2 boot-mode jumper to **JTAG**. Connect the micro-USB (PROG/UART) to the PC. Switch the board **ON**.
2. Find the COM port: Windows **Device Manager → Ports (COM & LPT) → USB Serial Port (COMx)**.
3. Open **MobaXterm**: **Session → Serial**. Serial port = **COMx**, Speed = **115200**. Click **OK**. Leave this window open.

### Part J: Vitis: create the application

1. Open **Vitis IDE** (Windows Start, or Vivado **Tools → Launch Vitis IDE**). Workspace: `C:/fpga_exam/q11/vitis_ws` → **Launch**.
2. **File → New → Application Project** → **Next**.
3. Choose the **Create a new platform from hardware (XSA)** tab → **Browse** → `C:/fpga_exam/q11/q11_median_hw.xsa` → **Next**.
4. Application project name: `q11_median_app` → **Next**.
5. Domain: Operating System **standalone**, Processor **ps7_cortexa9_0** → **Next**.
6. Template: **Hello World** → **Finish**. Wait for the platform to build.
7. **If you have the new unified Vitis (2023.2 or later):** use **File → New Component → Platform** (pick the .xsa, standalone, ps7_cortexa9_0) and Build it. Then use **Examples → Hello World → Create Application Component from Template** with that platform.

### Part K: Put in the C code and build

1. In the Explorer, expand `q11_median_app → src` and double-click **helloworld.c**.
2. Press **Ctrl+A**, then **Delete**. Open `q11/q11_main.c` in Notepad, copy everything, paste it into helloworld.c, and press **Ctrl+S**.
3. Check the `#define` offsets at the top match the IP Core Report (A = 0x100, B = 0x104, C = 0x108, med = 0x10C). `IP_BASE` uses `XPAR_Q11_DUT_IP_0_BASEADDR` if Vitis defines it; otherwise it uses `0x43C00000`. If your Address Editor showed a different base, change that number.
4. Right-click `q11_median_app` → **Build Project**. The Console must end with **Build Finished** and no errors.

### Part L: Run on the board and check MobaXterm

1. Right-click `q11_median_app` → **Run As → Launch Hardware** (older versions: *Launch on Hardware (Single Application Debug)*). This programs the FPGA, runs ps7_init and starts the program.
2. Look at MobaXterm. It prints one line per test: `Test | Inputs | Expected | Observed | PASS`, then **3/3 PASS**.
3. If nothing prints: check the COM port/baud, then use Vitis **Xilinx → Program Device → Program** and Run again.
4. If values print but are 0 or wrong: the base address or offsets don't match (check the Address Editor and the IP Core Report).
5. Screenshot MobaXterm.

### Part M: Expected results

| Test | Inputs | Expected |
|---|---|---|
| 1 Unordered Sequence | A = 15, B = 90, C = 40 | med = 40 |
| 2 Negative Quantities | A = −50, B = −10, C = −80 | med = −50 |
| 3 ADC Spike Rejection | A = 3300, B = 9999, C = 3320 | med = 3320 |

**Fallback if the `.slx` can't be used (build it by hand):** new blank model → add a Subsystem named `q11_dut`. Inside it, delete the In1→Out1 line and add one **MATLAB Function** block. Paste the contents of `median3.m` into it. Name the Inports A, B, C and set each one's data type to **int32**. Name the Outports med. Wire Inports → function → Outports. At the top level, add int32 **Constant** blocks (Sample time 1) on the inputs and **Display** blocks on the outputs. Model Settings → Solver → **Fixed-step, discrete**. Run `hdlsetup('<model name>')` in the Command Window, save, and continue with Part B.

---

<a id="q12"></a>

## Q12: Saturated Adder with Dynamic Clamping

**Files:** `q12/sat_add.m` (MATLAB function code), `q12/build_q12_model.m` + `q12/make_hdl_model.m` (make the Simulink file `q12_satadd.slx`), `q12/q12_main.c` (Vitis C code)

**Board connections:** PYNQ-Z2, micro-USB cable to the PC, boot jumper on **JTAG**.

### Part A: Get the Simulink model ready (MATLAB)

1. Copy the folder `q12` to the lab PC, e.g. `C:/fpga_exam/q12` (no spaces in the path).
2. Open **MATLAB**. In the **Current Folder** bar, browse to `C:/fpga_exam/q12`.
3. If `q12_satadd.slx` is **not** in the folder yet: in the Command Window type `build_q12_model` and press **Enter**. Wait for `Saved q12_satadd.slx`. (Warnings saying `... not set` are OK; set those items by hand in Part C.)
4. If MATLAB can't find Vivado later (Part C), run this once in the Command Window, with your Vivado version and path:
   ```
   hdlsetuptoolpath('ToolName','Xilinx Vivado','ToolPath','C:\Xilinx\Vivado\2022.2\bin\vivado.bat')
   ```
5. Double-click `q12_satadd.slx` to open it. You'll see Constant blocks (A_in, B_in) → subsystem **`q12_dut`** → Display blocks.
6. Double-click `q12_dut` to look inside. It contains Inports → **one MATLAB Function block** (`sat_add`) → Outports. Double-click the MATLAB Function block to see the code (the same as `sat_add.m`). Go back to the top level.

### Part B: Simulate in Simulink

1. On the **Simulation** tab, click **Run** (green ▶).
2. The Display blocks show **Y = 500** (test case 1).
3. Optional: double-click the Constant blocks to try the other cases, then click Run: A_in = 800, B_in = 500 gives Y = 1000; A_in = -700, B_in = -600 gives Y = -1000. Put the values back to test case 1 afterwards.
4. Press **Ctrl+S** to save the model.

### Part C: Generate the AXI4-Lite IP core (HDL Workflow Advisor)

1. Right-click the subsystem **`q12_dut`** → **HDL Code → HDL Workflow Advisor**.
2. Select **1.1 Set Target Device and Synthesis Tool** and check (fix if different):
   - Target workflow **IP Core Generation**
   - Target platform **Generic Xilinx Platform**
   - Synthesis tool **Xilinx Vivado**
   - Family **Zynq**, Device **xc7z020**, Package **clg400**, Speed **-1**
   - Project folder `hdl_prj`
   Click **Run This Task**.
3. If **1.2 Set Target Reference Design** appears, keep the defaults and click **Run This Task**.
4. Select **1.3 Set Target Interface**. In the table, every port's **Target Platform Interfaces** = **AXI4-Lite**, with addresses **A = 0x100, B = 0x104, Y = 0x108**. Click **Run This Task**.
5. Select **1.4 Set Target Frequency**: Target Frequency (MHz) = **50**. Click **Run This Task**.
6. Right-click **3.2 Generate RTL Code and IP Core** and choose **Run to Selected Task**. It runs steps 2.x, 3.1 and 3.2 in order.
7. If a step in folder 2 fails and shows a **Modify Settings** button, click it, then right-click 3.2 → **Run to Selected Task** again.
8. When 3.2 passes, click the **IP Core Report** link. In **Register Address Mapping**, check A = 0x100, B = 0x104, Y = 0x108. These must match the `#define`s at the top of `q12_main.c`.
9. The IP is in `C:/fpga_exam/q12/hdl_prj/ipcore/q12_dut_ip_v1_0`.

### Part D: Create the Vivado project

1. Open **Vivado**. On the Quick Start page click **Create Project**, then **Next**.
2. Project name: `q12_satadd_vivado`. Project location: `C:/fpga_exam/q12`. Keep **Create project subdirectory** ticked. Click **Next**.
3. Select **RTL Project** and tick **Do not specify sources at this time**. Click **Next**.
4. Click the **Boards** tab, search `pynq` and select **PYNQ-Z2**. Click **Next**. If it isn't listed: **Parts** tab → `xc7z020clg400-1`.
5. Click **Finish**.

### Part E: Add the HDL Coder IP repository

1. Flow Navigator → PROJECT MANAGER → **Settings**.
2. Go to **Project Settings → IP → Repository** and click **+**.
3. Browse to `C:/fpga_exam/q12/hdl_prj/ipcore` and click **Select**. A message lists **1 IP** (`q12_dut_ip`). Click **OK**, then **OK** again.

### Part F: Block design: Zynq PS + your IP

1. Flow Navigator → IP INTEGRATOR → **Create Block Design**. Name it `design_1` and click **OK**.
2. **How to add an IP:** click **+** (Add IP) on the Diagram toolbar or press **Ctrl+I**, type the name in **Search**, and double-click the result. **How to rename:** click the block, change **Name** in the **Block Properties** panel, and press **Enter**. **How to configure:** double-click the block.
3. Add **ZYNQ7 Processing System**. Click **Run Block Automation**, keep **Apply Board Preset** ticked, and click **OK**.
4. Double-click `processing_system7_0`. Go to **Clock Configuration → PL Fabric Clocks** and set **FCLK_CLK0 = 50** MHz. Click **OK**.
5. Add IP: search **q12_dut_ip** and double-click it. Block `q12_dut_ip_0` appears.
6. Click **Run Connection Automation**. Tick **All Automation** and click **OK**. This adds the AXI Interconnect and Processor System Reset and connects `AXI4_Lite` and its clock/reset.
7. Check the remaining pins on `q12_dut_ip_0`. If **IPCORE_CLK** is unconnected, drag it onto the **FCLK_CLK0** wire. If **IPCORE_RESETN** is unconnected, drag it onto `rst_ps7_0_50M/peripheral_aresetn`.
8. Click **Regenerate Layout** (⟳), then **Validate Design** (F6). Expect **Validation successful**. Press **Ctrl+S**.
9. Open the **Address Editor** tab and note the IP's base address (usually **0x43C0_0000**).

### Part G: Wrapper and bitstream

1. In **Sources**, right-click **design_1 (design_1.bd)** → **Create HDL Wrapper** → **Let Vivado manage wrapper and auto-update** → **OK**.
2. Flow Navigator → PROGRAM AND DEBUG → **Generate Bitstream**. Click **Yes** (run synthesis and implementation), then **OK**. No constraints file is needed: the clock comes from the PS.
3. Wait for **Bitstream Generation successfully completed**, then click **Cancel** on the dialog.

### Part H: Export the hardware (.xsa)

1. Menu **File → Export → Export Hardware** → **Next**.
2. Select **Include bitstream** and click **Next**.
3. XSA file name: `q12_satadd_hw`. Export to: `C:/fpga_exam/q12`. Click **Next**, then **Finish**.

### Part I: Connect the board and open MobaXterm

1. Set the PYNQ-Z2 boot-mode jumper to **JTAG**. Connect the micro-USB (PROG/UART) to the PC. Switch the board **ON**.
2. Find the COM port: Windows **Device Manager → Ports (COM & LPT) → USB Serial Port (COMx)**.
3. Open **MobaXterm**: **Session → Serial**. Serial port = **COMx**, Speed = **115200**. Click **OK**. Leave this window open.

### Part J: Vitis: create the application

1. Open **Vitis IDE** (Windows Start, or Vivado **Tools → Launch Vitis IDE**). Workspace: `C:/fpga_exam/q12/vitis_ws` → **Launch**.
2. **File → New → Application Project** → **Next**.
3. Choose the **Create a new platform from hardware (XSA)** tab → **Browse** → `C:/fpga_exam/q12/q12_satadd_hw.xsa` → **Next**.
4. Application project name: `q12_satadd_app` → **Next**.
5. Domain: Operating System **standalone**, Processor **ps7_cortexa9_0** → **Next**.
6. Template: **Hello World** → **Finish**. Wait for the platform to build.
7. **If you have the new unified Vitis (2023.2 or later):** use **File → New Component → Platform** (pick the .xsa, standalone, ps7_cortexa9_0) and Build it. Then use **Examples → Hello World → Create Application Component from Template** with that platform.

### Part K: Put in the C code and build

1. In the Explorer, expand `q12_satadd_app → src` and double-click **helloworld.c**.
2. Press **Ctrl+A**, then **Delete**. Open `q12/q12_main.c` in Notepad, copy everything, paste it into helloworld.c, and press **Ctrl+S**.
3. Check the `#define` offsets at the top match the IP Core Report (A = 0x100, B = 0x104, Y = 0x108). `IP_BASE` uses `XPAR_Q12_DUT_IP_0_BASEADDR` if Vitis defines it; otherwise it uses `0x43C00000`. If your Address Editor showed a different base, change that number.
4. Right-click `q12_satadd_app` → **Build Project**. The Console must end with **Build Finished** and no errors.

### Part L: Run on the board and check MobaXterm

1. Right-click `q12_satadd_app` → **Run As → Launch Hardware** (older versions: *Launch on Hardware (Single Application Debug)*). This programs the FPGA, runs ps7_init and starts the program.
2. Look at MobaXterm. It prints one line per test: `Test | Inputs | Expected | Observed | PASS`, then **3/3 PASS**.
3. If nothing prints: check the COM port/baud, then use Vitis **Xilinx → Program Device → Program** and Run again.
4. If values print but are 0 or wrong: the base address or offsets don't match (check the Address Editor and the IP Core Report).
5. Screenshot MobaXterm.

### Part M: Expected results

| Test | Inputs | Expected |
|---|---|---|
| 1 Within bounds | A = 200, B = 300 | Y = 500 |
| 2 Positive saturation | A = 800, B = 500 | Y = 1000 |
| 3 Negative saturation | A = −700, B = −600 | Y = −1000 |

**Fallback if the `.slx` can't be used (build it by hand):** new blank model → add a Subsystem named `q12_dut`. Inside it, delete the In1→Out1 line and add one **MATLAB Function** block. Paste the contents of `sat_add.m` into it. Name the Inports A, B and set each one's data type to **int32**. Name the Outports Y. Wire Inports → function → Outports. At the top level, add int32 **Constant** blocks (Sample time 1) on the inputs and **Display** blocks on the outputs. Model Settings → Solver → **Fixed-step, discrete**. Run `hdlsetup('<model name>')` in the Command Window, save, and continue with Part B.

