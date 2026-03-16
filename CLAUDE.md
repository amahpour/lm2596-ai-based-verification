# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI-driven bring-up and profiling of an LM2596 DC-DC buck converter module using three Rigol instrument MCP servers. There is no traditional software build system — work consists of controlling lab instruments, collecting measurements, and analyzing results.

## Instrument MCP Servers

All testing is done through these MCP servers (must be connected):

| Server | Instrument | Purpose |
|---|---|---|
| `rigol-dp832` | Rigol DP832A | Power supply — drives DUT input via **Channel 2** |
| `rigol-dl3021` | Rigol DL3021A | Electronic load — sinks DUT output current |
| `rigol-mso5000` | Rigol MSO5074 | Oscilloscope — captures switching waveforms/ripple |

**Always connect without specifying an IP address** — the MCP servers use auto-discovery (SCPI broadcast) to find instruments on the local network. IPs are assigned by DHCP and change frequently. Do NOT hardcode IP addresses.

### Wiring

- DP832A CH2 → DUT IN+/IN−
- DL3021A → DUT OUT+/OUT−
- Scope CH1 → DUT input side
- Scope CH2 → DUT output side

## Safety Limits (MUST enforce)

- **Input voltage:** ≤30V (absolute max 35V)
- **Supply current limit:** Default 2A on DP832A; only raise to 3A with explicit user confirmation
- **Load current:** ≤3A
- **Sustained loads >2.5A:** Warn about thermal limits, keep tests short
- **Minimum headroom:** VIN must be at least 1.5V above VOUT
- **Startup order:** Configure all instruments → enable power supply → enable eload
- **Shutdown order:** Disable eload first → then disable power supply

## DUT Key Specs

| Parameter | Value | Notes |
|---|---|---|
| Input range | 3.2V – 35V | |
| Output range | 1.25V – 30V | Adjustable via trimmer potentiometer |
| Max output current | 3A | Derate to 2.5A for sustained use |
| Switching frequency | 65kHz | Per Amazon listing; TI datasheet says 150kHz — likely clone IC |
| Input cap (CIN) | 100µF / 50V | Undersized vs TI-recommended 470µF |
| Output cap (COUT) | 220µF / 35V | Matches datasheet |

## Profiling Skill

Use `/buck-converter-profile` for automated test sequences:

```
/buck-converter-profile load-sweep 12 3    # Load regulation sweep
/buck-converter-profile line-sweep 3       # Line regulation sweep
/buck-converter-profile efficiency 12 3    # Efficiency curve
/buck-converter-profile ripple 12 3        # Output ripple measurement
/buck-converter-profile full 12 3          # Run all tests
```

See `.claude/skills/buck-converter-profile/SKILL.md` for full test procedures and measurement steps.

## Reference Data

- `lm2596.pdf` — TI LM2596 datasheet (full PDF)
- `datasheet_out/lm2596/tables/` — Extracted datasheet tables in JSON, CSV, and Markdown (26 tables). Use `index.json` to locate specific tables.
- `datasheet_out/lm2596/document.json` — Full extracted text from the datasheet
- `product_description.md` — Amazon listing specs and identified board components
- `product_photos/` — Product images showing board layout and components
