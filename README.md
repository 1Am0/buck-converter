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

U1 switches the input at about 150 kHz. L1 and C_OUT smooth the switched voltage into a steady output, and D1 keeps current flowing in L1 while the switch is off. R2 and R3 in series from the output, plus R1 to ground, scale the output down to the 1.23 V that U1's feedback pin regulates to. C_IN and C_BYP supply the input current pulses.

## Status

Schematic complete, ERC clean. Perfboard layout and build are next. The inductor and C_OUT footprints are placeholders to check against the real parts.