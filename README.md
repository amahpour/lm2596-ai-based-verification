# lm2596-ai-based-verification

AI-based bring-up and profiling of the LM2596 Buck Converter using Rigol instrument MCP servers controlled directly from Claude Code.

## Device Under Test

**LM2596 DC-DC Buck Converter Module** ([Amazon B076H3XHXP](https://www.amazon.com/dp/B076H3XHXP))

| Parameter | Value |
|---|---|
| Type | DC-DC non-isolated buck, non-synchronous rectification |
| Input Voltage | DC 3.2V - 35V |
| Output Voltage | DC 1.25V - 30V (adjustable) |
| Output Current | 3A (max) |
| Output Power | 15W (max) |
| Conversion Efficiency | 92% (max) |
| Switching Frequency | 65kHz (per listing; TI datasheet says 150kHz) |
| Load Regulation | +/- 0.5% |
| Voltage Regulation | +/- 2.5% |
| Operating Temperature | -45C to +85C |
| Size | 45 x 23 x 14mm |

### Board Components (identified from product photos)

| Component | Value | Notes |
|---|---|---|
| IC | LM2596 (adjustable), TO-263 | Possibly clone — 65kHz vs 150kHz TI spec |
| Inductor | 47uH shielded SMD | Marked "470", datasheet range: 33-68uH |
| Input Capacitor (CIN) | 100uF / 50V electrolytic | Undersized vs datasheet recommended 470uF |
| Output Capacitor (COUT) | 220uF / 35V electrolytic | Matches datasheet recommendation |
| Catch Diode | SS34 (3A/40V Schottky) | SMD equivalent of 1N5822 |
| Potentiometer (R2) | 10k ohm trimmer | Blue multi-turn, sets VOUT via feedback divider |
| R1 (feedback) | Unknown | Hidden on PCB, needs measurement |

## Test Equipment

| Instrument | Model | IP Address | Role |
|---|---|---|---|
| Power Supply | Rigol DP832A | 192.168.68.138 | Input power to DUT |
| Electronic Load | Rigol DL3021A | 192.168.68.131 | Output load on DUT |
| Oscilloscope | Rigol MSO5074 | 192.168.68.113 | Switching waveforms and ripple |

All instruments are controlled via custom Rigol MCP servers from Claude Code:

- [rigol_dp832_mcp_server](https://github.com/amahpour/rigol_dp832_mcp_server) — Power supply control
- [rigol_dl3021_mcp_server](https://github.com/amahpour/rigol_dl3021_mcp_server) — Electronic load control
- [pymso5000-mcp](https://github.com/amahpour/pymso5000-mcp) — Oscilloscope control

### Wiring Setup

```
                        DUT (LM2596 Module)
                     ┌──────────────────────┐
  DP832A CH2 ───────>│ IN+            OUT+ │───────> DL3021A (eload)
                     │                      │
             GND ───>│ IN-            OUT- │───────> DL3021A GND
                     └──────────────────────┘
                       │                  │
                  Scope CH1          Scope CH2
                  (input side)       (output side)
```

- **DP832A Channel 2** → DUT input (IN+/IN-)
- **Scope Channel 1** → DUT input (monitors input voltage)
- **DL3021A** → DUT output (OUT+/OUT-)
- **Scope Channel 2** → DUT output (monitors output voltage/ripple)

## Claude Code Skill

A `/buck-converter-profile` skill is included in `.claude/skills/` for automated profiling. Usage:

```
/buck-converter-profile load-sweep 12 3    # Load regulation sweep at 12V in, 0-3A
/buck-converter-profile line-sweep 3       # Line regulation sweep at 3A load
/buck-converter-profile efficiency 12 3    # Efficiency curve
/buck-converter-profile ripple 12 3        # Output ripple measurement
/buck-converter-profile full 12 3          # Run all tests
```

## Repository Structure

- `lm2596.pdf` — TI LM2596 datasheet
- `product_description.md` — Amazon listing details and board component analysis
- `product_photos/` — Product images from Amazon listing
- `datasheet_out/` — Extracted datasheet data (text, tables) via [datasheet-extractor](https://github.com/amahpour/datasheet-extractor)
- `.claude/skills/` — Claude Code skill for automated buck converter profiling
