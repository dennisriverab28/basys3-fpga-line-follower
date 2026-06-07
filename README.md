# Line-Following Color-Sensing Robot — Basys 3 FPGA

**Course:** RPL1 — Robotics Project Lab 1, Texas Tech University  
**Team:** Donte G · Dennis Rivera  
**Hardware:** Digilent Basys 3 (Artix-7 FPGA) · Verilog HDL

---

## Overview

A Verilog-implemented line-following robot controlled by a Basys 3 FPGA. The robot follows a line using IR sensors and can identify colored targets using a TCS230/TCS3200 color sensor, adjusting its behavior based on color detection. The design integrates PWM motor control, servo actuation, overcurrent protection, a frequency measurement unit, and a 7-segment status display — all implemented as synthesizable Verilog modules.

---

## Features

| Module | Description |
|--------|-------------|
| **Color Sensor FSM** | State machine that cycles through R/G/B photodiode selection and captures frequency readings for each channel |
| **Frequency → Color Converter** | Converts raw frequency measurements to identified color (RGB threshold comparison) |
| **IR Emitter / Receiver** | PWM-driven IR LED + receiver for line detection |
| **Line Following** | Motor direction logic based on IR sensor pair readings; multiple speed variation profiles |
| **Motor Control** | Top-level integration: takes IR + color inputs, outputs left/right PWM and direction signals |
| **PWM Generator** | Configurable duty-cycle PWM for motor speed control |
| **Servo Control** | Servo position control module (claw/arm mechanism) |
| **Overcurrent Protection** | Hardware protection module with delay; flips protection switch and signals fault via LED |
| **Seven Segment Display** | Status/debug output on Basys 3 onboard 7-segment display |
| **Testbenches** | Simulation testbenches for color FSM, frequency measurement, motor control, servo, and 7-segment |

---

## Project Structure

All files are Verilog HDL (`.v`) — uploaded to the repo without file extensions.

```
Basys_Board_FPGA_Project/
│
├── Color Sensor Top Module + Emitter Implementation + INDEXING + DESIRED CONDITION
├── Color State Machine              # ColorSensorStateMachine — R/G/B FSM with freq_meas
├── Color Sensor Constraints         # Basys 3 XDC pin constraints for color sensor
├── Frequency to Color Converter     # Maps measured frequencies to color ID
├── freq_meas Module                 # Frequency measurement via clock counting
│
├── IR_emitter                       # PWM-driven IR LED emitter
├── IR_emitter_top                   # Top-level IR emitter with constraints
├── cons_IR_emitter                  # XDC constraints for IR emitter
├── IPSxIR                           # IR sensor + line following integration
├── IPSxIR_const                     # Constraints for IPS x IR integration
├── simple_pwm_IR                    # Simple PWM for IR timing
├── PWM SIMPLE                       # Basic PWM generator
├── PWM_simple for ir                # PWM variant for IR emitter
│
├── motor_control                    # Motor control FSM (IR + color inputs → PWM + direction)
├── final_motor_control              # Final integrated motor control top module
├── Line Following 75 to 50 Variation  # Line follow: speed ramp 75%→50%
├── Line Following 60 to 50 Variation  # Line follow: speed ramp 60%→50%
├── Line Following 50 to 30 Variation  # Line follow: speed ramp 50%→30%
├── Line Following Constraints         # XDC constraints for line following
│
├── Servo Control Module             # Servo PWM position control
├── Servo Top Module                 # Servo top-level integration
├── Servo Constraints                # XDC constraints for servo
│
├── OverCurrent Protection Top Module              # OC protection with LM324AN comparator
├── Overcurrent Protection LM324AN Top Module      # Alternate OC module
├── Finalized OC protection with Delay             # Final OC with delay before fault
├── Overcurrent_Protection (switch and LED testing)# OC test with LED indicator
├── OC Test                                        # OC simulation
├── Overcurrent Constraints                        # XDC constraints for OC protection
│
├── Seven Segment                    # 7-segment display driver
│
├── Integration Task 1               # Task 1 integrated design
├── Task Integration 2               # Task 2 integrated design
│
├── Servo tb                         # Servo testbench
├── IPSxIR_tb                        # IR sensor integration testbench
├── Overcurrent Protection tb        # OC protection testbench
├── tb_ColorSensorStateMachine       # Color FSM testbench (WIP)
├── tb_freq_meas_optimized           # Frequency measurement testbench
├── tb_seven_segment_display         # 7-segment testbench
│
└── temp                             # Scratch/work-in-progress module
```

---

## System Integration

```
Basys 3 FPGA
    │
    ├── IR Sensor Pair ──────────────► Line Following Logic
    │                                        │
    ├── TCS230 Color Sensor ─► Color FSM ──► Motor Control FSM ──► L298N ──► Drive Motors
    │       (R/G/B freq)        (freq_meas)         │
    │                                         Servo Control ──────────────► Servo (claw)
    │
    ├── INA/LM324AN ──────────► Overcurrent Protection ──► Fault LED + Motor Shutoff
    │
    └── 7-Segment Display ──────────────────────────────► Status Output
```

---

## Build & Program

### Requirements
- [Vivado Design Suite](https://www.xilinx.com/products/design-tools/vivado.html) (Xilinx/AMD)
- Digilent Basys 3 board

### Steps
1. Open Vivado → `Create Project` → Add all Verilog files as sources
2. Set top module to `final_motor_control` (or the relevant top-level for your task)
3. Add the appropriate constraints file (`.xdc`) for your target integration
4. **Simulate:** Run behavioral simulation with the relevant testbench (`tb_*` files)
5. **Synthesize & Implement:** `Flow > Run Implementation`
6. **Program:** `Flow > Generate Bitstream` → `Open Hardware Manager` → Program Basys 3

### Note on File Extensions
All Verilog source files in this repo were committed without `.v` extensions. When importing into Vivado, add them as Verilog sources — Vivado will parse them correctly regardless of extension.

---

## Key Modules

### `Color State Machine`
FSM that controls the TCS230 S2/S3 photodiode select pins to cycle through red, green, and blue measurements. Each state captures the output frequency from `freq_meas` and stores R/G/B values. Asserts `done_color` when all three channels are captured.

### `motor_control` / `final_motor_control`
Top-level integration module. Takes IR sensor readings (`IR_sensor[1:0]`), color ID (`rgb_desired`, `rgb_output`), and proximity sensor (`psensor`) as inputs. Outputs left/right motor direction pins (`LIN1`, `LIN2`, `RIN1`, `RIN2`) and PWM signals. Controls a claw servo output as well.

### `Overcurrent Protection`
Uses an LM324AN comparator to detect overcurrent conditions on motor drivers. The "Finalized OC protection with Delay" version adds a time delay before triggering the fault to avoid false positives from motor startup inrush current.
