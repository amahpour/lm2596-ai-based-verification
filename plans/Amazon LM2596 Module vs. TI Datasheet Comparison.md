# Plan: Amazon LM2596 Module vs. TI Datasheet Comparison

## Context

The DUT is a cheap Amazon LM2596 buck converter module (ASIN B076H3XHXP). Several red flags have already been identified from the product listing and board photos — most notably a 65kHz switching frequency (TI specifies 150kHz), an undersized input capacitor (100uF vs 470uF recommended), and a missing feedforward capacitor. This plan produces a structured comparison report that combines a static BOM analysis against TI's datasheet with real measurements from the lab instruments to validate each finding.

## Phase 1: Static Analysis (No Instruments)

Build a component-by-component scorecard by cross-referencing `product_description.md` against the extracted datasheet tables. Each row gets a PASS / MARGINAL / CONCERN / FAIL rating.

| # | Category | Amazon Module | TI Recommendation | Source Tables |
|---|----------|--------------|-------------------|---------------|
| 1 | **IC Authenticity** | 65kHz, 3.2-35V input | 150kHz (127-173kHz), 4.5-40V operating | `table_0012`, `table_0006` |
| 2 | **Input Cap (CIN)** | 100uF/50V generic electrolytic | 470uF low-ESR, RMS rated for 50% of load current | `document.json` sec 9.1.1 |
| 3 | **Output Cap (COUT)** | 220uF/35V electrolytic | 220uF acceptable; ESR is the critical unknown | `table_0019` |
| 4 | **Inductor** | 47uH shielded SMD | 22-68uH depending on VIN/VOUT/ILOAD; 47uH is mid-range | `table_0013`, `table_0016` |
| 5 | **Catch Diode** | SS34 (3A/40V Schottky) | SK34/1N5822 class; TI reference uses 5A diode | `table_0017` |
| 6 | **Feedback Network** | R2=10k pot, R1=unknown | R1 ~1k recommended (1%), determines VOUT range | `document.json` sec 9.2 |
| 7 | **Feedforward Cap (CFF)** | Not present | Required for VOUT > 10V (stability) | `table_0019`, sec 9.1.2 |
| 8 | **PCB Layout/Thermal** | 45x23mm (~1 sq in) | 3-9 sq in with 2oz copper recommended | `table_0020`, `table_0021` |

### Key files to read:
- `product_description.md` — DUT component list
- `datasheet_out/lm2596/tables/table_0012.json` — device parameters (freq, Vsat, Ilimit)
- `datasheet_out/lm2596/tables/table_0016.json` — complete design examples
- `datasheet_out/lm2596/tables/table_0019.json` — output cap / CFF selection
- `datasheet_out/lm2596/document.json` — CIN sizing (sec 9.1.1), CFF (sec 9.1.2), inductor/diode guidance

## Phase 2: Measurement-Based Verification

All tests use the existing `/buck-converter-profile` skill procedures where possible. Target VOUT: whatever the pot is currently set to (measure first).

### Test 0: Pre-flight
1. Instruments already connected
2. Set DP832A CH2 to 12V / 2A limit, enable, measure VOUT at eload terminals
3. Record baseline VOUT — this is our reference for all tests
4. Disable supply

### Test 1: Switching Frequency (Clone Verification) — MOST IMPORTANT
- **Conditions:** VIN=12V, ILOAD=1A
- **Method:** Scope CH2 at 50mV/div AC-coupled, timebase 5us/div. Capture waveform, measure period, calculate frequency.
- **Pass:** 127-173kHz (genuine TI). **Fail:** ~65kHz (clone confirmed).

### Test 2: Load Sweep (captures regulation + efficiency + ripple in one pass)
- **Conditions:** VIN=12V, ILOAD=0 to 2.5A in 0.25A steps
- **Method:** Use `/buck-converter-profile load-sweep 12 2.5`. At each step record VIN, IIN, VOUT, IOUT, CH1 Vpp (input ripple), CH2 Vpp (output ripple).
- **Derived metrics:**
  - Load regulation: (VOUT_noload - VOUT_fullload) / VOUT_noload × 100% → compare to claimed ±0.5%
  - Efficiency: POUT/PIN × 100% → compare to TI's ~80% at 5V/3A and Amazon's 92% max claim
  - Input ripple vs load → validates undersized CIN
  - Output ripple vs load → validates COUT adequacy

### Test 3: Line Regulation
- **Conditions:** ILOAD=1A fixed, VIN swept from (VOUT+2V) to 25V in 2V steps
- **Method:** Use `/buck-converter-profile line-sweep 1`
- **Pass:** VOUT variation ≤2.5% (Amazon claim). Compare to TI's ±4% system accuracy.

### Test 4: Thermal Soak (if time permits)
- **Conditions:** VIN=12V, ILOAD=2A sustained for 2-3 minutes
- **Method:** Record efficiency at start and end. Efficiency drop indicates thermal issues from the undersized PCB.
- **Pass:** <1% efficiency drop. **Concern:** >3% drop.

## Phase 3: Report Generation

Create `comparison_report.md` with this structure:

```
# LM2596 Amazon Module vs. TI Datasheet Comparison

## Executive Summary
- Clone determination and key deviations
- 3-5 bullet point findings

## Static Analysis Scorecard
[8-row table from Phase 1, with ratings]

## Measurement Results
### Switching Frequency — [result, clone Y/N]
### Load Regulation — [table + ±% result vs claimed ±0.5%]
### Input Ripple — [table, mVpp vs load, assessment of CIN adequacy]
### Output Ripple — [table, mVpp vs load, assessment of COUT adequacy]
### Efficiency Curve — [table + comparison to TI reference values]
### Line Regulation — [table + ±% result vs claimed ±2.5%]
### Thermal (if performed)

## Static vs. Measured Correlation
[Did measurements confirm each static finding?]

## Final Verdict Table
[Combined rating per category]

## Recommendations
[What would improve this design]
```

## Execution Order

1. Write static scorecard section (reading datasheet tables)
2. Test 0: Measure baseline VOUT
3. Test 1: Switching frequency (informs interpretation of everything else)
4. Test 2: Load sweep at 12V (captures regulation, efficiency, ripple in one pass)
5. Test 3: Line regulation sweep
6. Test 4: Thermal soak (optional, user can skip)
7. Compile report with all data

## Safety Reminders
- Startup: configure → enable PSU → enable eload
- Shutdown: disable eload → disable PSU
- Default 2A current limit; 3A only with user confirmation
- Loads >2.5A: warn, keep short
- VIN never exceeds 30V
