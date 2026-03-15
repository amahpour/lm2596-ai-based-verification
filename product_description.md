# LM2596 DC-DC Buck Converter Module

**Amazon Link:** https://www.amazon.com/dp/B076H3XHXP

## Product Description

### Features
- DC to DC step-down module
- Thickened circuit boards
- Advanced solid-state capacitor
- Short circuit and over-temperature protection
- 100% tested, sealed in anti-static packaging

### Important Notes
1. Input voltage must be 1.5V higher than the output voltage, no boost
2. Do not reverse the input and output interface
3. Keep output under 2.5A (or 10W) and use a heat sink when working for a long time
4. The factory settings stay a high voltage, you may need to contrarotate the screw on blue potentiometer about 7-15 circles before the output voltage changes

### Specifications
| Parameter | Value |
|---|---|
| Type | DC-DC buck converter |
| Input Voltage | DC 3.2V - 35V |
| Output Voltage | DC 1.25V - 30V (adjustable) |
| Output Current | 3A (max) |
| Output Power | 15W (max) |
| Conversion Efficiency | 92% (max) |
| Switching Frequency | 65kHz |
| Rectification | Non-synchronous rectification |
| Module Property | Non-isolated buck module |
| Operating Temperature | -45C to +85C |
| Load Regulation | +/- 0.5% |
| Voltage Regulation | +/- 2.5% |
| Size (L x W x H) | 45 x 23 x 14mm |
| Weight | 14g |

### Package Contents
6x DC to DC Buck Converter Module

## Identified Board Components (from product photos)

| Component | Value | Notes |
|---|---|---|
| IC | LM2596 (adjustable), TO-263 | Possibly clone - 65kHz listed vs 150kHz TI datasheet |
| Inductor | 47uH shielded SMD | Marked "470", datasheet range: 33-68uH |
| Input Capacitor (CIN) | 100uF / 50V electrolytic | Undersized vs datasheet recommended 470uF |
| Output Capacitor (COUT) | 220uF / 35V electrolytic | Matches datasheet recommendation |
| Catch Diode | SS34 (3A/40V Schottky) | SMD equivalent of 1N5822, matches datasheet rec |
| Potentiometer (R2) | 10k ohm trimmer | Blue multi-turn, sets VOUT via feedback divider |
| R1 (feedback) | Unknown | Hidden on PCB, needs measurement |
