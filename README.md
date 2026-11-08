# Buck Converter

Schematic for a 12 V to 5 V, 1 A step-down converter built around the LM2596T-ADJ, designed and simulated in LTspice, drawn in Fusion Electronics, and planned for a perfboard build.

## Files

| File | What it is |
|---|---|
| schematics/buck-converter.pdf | Schematic, one page |
| schematics/buck-converter.png | Same schematic as an image |
| schematics/buck-converter.sch | EAGLE-format schematic exported from Fusion |
| BOM.md | Parts list with values and footprints |

## How it works

U1 switches the input at about 150 kHz. L1 and C_OUT smooth the switched voltage into a steady output, and D1 keeps current flowing in L1 while the switch is off. R4, R2 and R3 in series from the output (3.1 kohm total), plus R1 (1 kohm) to ground, scale the output down to the 1.23 V that U1's feedback pin regulates to. C_IN and C_BYP supply the input current pulses.

## Results

Built on perfboard and tested: 4.98 V with no load, 4.97 V at full load (5 ohm, about 1 A), FB pin at 1.23 V. U1 runs hot at 1 A without a heatsink, so a TO-220 heatsink is on the BOM.

## Status

Schematic complete, ERC clean, and matches the built board. The inductor and C_OUT footprints are placeholders to check against the real parts.
