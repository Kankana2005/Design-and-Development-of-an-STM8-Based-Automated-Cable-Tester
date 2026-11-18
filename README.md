# Design and Development of an STM8-Based Automated Cable Tester

An STM8-microcontroller-driven automated cable/harness tester that detects opens, shorts, and miswires across multi-conductor cables by scanning each conductor with shift-register-based I/O and reporting the result on an onboard LCD.

## Overview

Testing multi-pin cables and wiring harnesses by hand (continuity-checking every pin combination with a multimeter) is slow and error-prone. This project replaces that manual process with a dedicated hardware tester built around an **STM8S005K6T6** microcontroller:

- One end of the cable under test is driven, pin by pin, through an output stage.
- The other end is scanned through an input stage.
- The MCU compares the expected wiring map against what it reads back and reports **good cable / open / short / miswire** on a 16x2 character LCD.
- A 4-button keypad lets the operator select test modes / cable profiles and step through results.

The electronics were designed in KiCad as a hierarchical schematic split into functional sheets (power, MCU, keypad, input, output), which keeps the high-channel-count I/O manageable.

## Hardware Architecture

| Sheet | Purpose | Key parts |
|---|---|---|
| **Power** | Converts 12 V input to a regulated 5 V rail | `TPS563200` synchronous buck converter, reverse-polarity protection diodes, status LED |
| **MCU** | Core control, timing, and user interface | `STM8S005K6T6CTR`, 12 MHz crystal oscillator, SWIM programming/debug header, `HY1602E` 16x2 character LCD |
| **Keypad** | User input | 4x pull-up momentary switches (`SW_MEC_5E`) |
| **Input** | Reads back cable conductors under test | 3x `74HC165` parallel-in/serial-out shift registers, cascaded (SIPO→SIPO), fed from a 15-pin and two auxiliary connectors supporting up to 23 test points |
| **Output** | Drives cable conductors under test | 4x `UBC3005G` (16-pin SOP) serial-in sink drivers, cascaded, driving a 26-pin high-density connector (`L77HDA26SD1CH4F`) |

The MCU communicates with the input and output shift-register chains over a shared synchronous serial bus (`SR_CLK`, `SR_DATA`/`SR_DATA_IN`, `SR_CE`, `SR_LOAD`/`SR_OE`), letting a handful of STM8 GPIO pins control dozens of test channels.

### Repository Contents

| File | Description |
|---|---|
| `Design and Development of an STM8-Based Automated Cable Tester (2).pdf` | Full project report: motivation, design methodology, schematic walk-through, firmware logic, test results, and conclusions |
| `Hierarchical Circuit Design.png` | Top-level KiCad schematic showing how the power, keypad, MCU, input, and output sheets interconnect |
| `STM8 Microcontroller Circuit Schematic.png` | MCU sheet — STM8S005K6T6, oscillator, SWIM header, LCD interface |
| `Power Supply.png` | 12 V → 5 V buck regulator and input protection circuit |
| `Keypad.png` | 4-button keypad input circuit |
| `Input.png` | Shift-register input stage used to read back cable conductors |
| `Output.png` | Shift-register output driver stage used to source test signals onto cable conductors |

## How It Works

1. The operator connects the cable under test between the **output** connector (driven side) and the **input** connector (read-back side).
2. The STM8 clocks a single active bit through the output shift-register chain, energizing one conductor at a time.
3. The STM8 simultaneously clocks in the state of every input channel through the input shift-register chain.
4. Firmware compares the observed pattern against the expected wiring map for that cable type:
   - **Exactly one matching input active** → that conductor is good.
   - **No input active** → open circuit on that conductor.
   - **Multiple inputs active** → short/cross-wire between conductors.
5. Results are shown on the 16x2 LCD, and the keypad is used to start a new test, cycle through per-pin results, or select a different cable profile.

## Getting Started

1. **Review the design**: start with the full report PDF and the hierarchical schematic image for the system overview, then drill into the individual sheet images for component-level detail.
2. **Build the hardware**: fabricate/assemble the circuit per the schematics (power, MCU, keypad, input, output sheets).
3. **Program the MCU**: flash the STM8S005K6T6 via the SWIM header using an ST-LINK programmer and your STM8 firmware toolchain (e.g., STVD/Cosmic, IAR, or SDCC).
4. **Connect a cable under test** across the output and input connectors and run a test cycle.

## Status

This repository currently holds the hardware design artifacts (schematics and project report) for the cable tester. Firmware source is not yet included in this repo.
