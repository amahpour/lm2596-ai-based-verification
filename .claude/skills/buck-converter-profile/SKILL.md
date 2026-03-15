---
name: buck-converter-profile
description: Profile the LM2596 buck converter using Rigol DP832A power supply, DL3021A electronic load, and MSO5074 oscilloscope via MCP servers. Use when running load sweeps, efficiency measurements, line regulation tests, or ripple characterization.
argument-hint: <test-type> [input-voltage] [max-current]
---

# Buck Converter Profiling

Profile the LM2596 DC-DC buck converter module using three Rigol instrument MCP servers.

## Arguments
- `$ARGUMENTS`: Optional test parameters. Examples:
  - `load-sweep 12 3` — Load regulation sweep at 12V input, 0 to 3A
  - `line-sweep 3` — Line regulation sweep at 3A load (varies input voltage)
  - `efficiency 12 3` — Efficiency curve at 12V input, 0 to 3A
  - `ripple 12 3` — Output ripple measurement at 12V input, 3A load
  - `full 12 3` — Run all tests

If no arguments provided, run a full profile at 12V input, 0 to 3A.

## Test Equipment & Wiring

- **DP832A Channel 2** → DUT input (IN+/IN-)
- **Scope Channel 1** → DUT input (monitors input voltage)
- **DL3021A** → DUT output (OUT+/OUT-)
- **Scope Channel 2** → DUT output (monitors output voltage/ripple)

## Safety Limits

These MUST be enforced at all times:
- **Input voltage:** Never exceed 35V (module max). Stay at or below 30V.
- **Supply current limit:** Set DP832A current limit to 2A for normal testing. Only raise to 3A if explicitly needed and confirmed by user.
- **Load current:** Never exceed 3A (module max rated).
- **Sustained load:** For loads above 2.5A, warn about thermal limits and keep test duration short.
- **Minimum headroom:** Input must be at least 1.5V above the set output voltage.
- **Startup sequence:** Always configure instruments BEFORE enabling outputs. Always disable eload BEFORE disabling power supply.
- **Shutdown sequence:** Disable eload first, then disable power supply output.

## MCP Servers

These three MCP servers must be connected:
- `rigol-dp832` — Power supply control (set voltage, current limit, enable/disable, measure)
- `rigol-dl3021` — Electronic load control (CC mode, set current, enable/disable, measure)
- `rigol-mso5000` — Oscilloscope (channel scale, timebase, waveform summary)

## Test Procedures

### Load Regulation Sweep
1. Connect to all three instruments
2. Verify all outputs/inputs are OFF
3. Set DP832A CH2 to specified input voltage with 2A current limit
4. Set scope CH1 and CH2 scales appropriate for the voltage levels (2V/div is a good starting point)
5. Set scope timebase to 10us/div (shows several switching cycles at 65kHz)
6. Enable DP832A CH2 output
7. Record no-load output voltage from eload
8. For each current step (0.25A increments from 0 to max):
   a. Set DL3021A CC mode to target current (enable on first step)
   b. Wait briefly for settling
   c. Record from DP832A: input voltage, input current, input power
   d. Record from DL3021A: output voltage, output current, output power
   e. Record from scope: CH1 mean/Vpp (input), CH2 mean/Vpp (output ripple)
   f. Calculate efficiency: (output power / input power) * 100%
9. Disable eload
10. Disable power supply
11. Present results as a markdown table

### Line Regulation Sweep
1. Set DL3021A to a fixed load current
2. Sweep DP832A CH2 input voltage from (VOUT + 2V) to 30V in 2V steps
3. At each step record: VIN, IIN, VOUT, IOUT, efficiency
4. Present results as a markdown table

### Ripple Measurement
1. Set scope to AC coupling or use tighter V/div scale (50mV or 100mV/div) on CH2
2. Set timebase to capture several switching cycles
3. Measure Vpp on CH2 at specified load current
4. Report ripple as absolute mV and as percentage of VOUT

### Full Profile
Run all of the above in sequence, then produce a summary report.

## Output Format

Present all results as markdown tables. Include:
- Test conditions (VIN, VOUT setpoint, temperature if available)
- Raw measurements at each operating point
- Calculated efficiency at each point
- Summary of key findings (regulation accuracy, max efficiency point, ripple)
