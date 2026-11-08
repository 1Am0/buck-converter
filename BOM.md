# Bill of Materials

12 V to 5 V, 1 A buck converter on perfboard (LM2596T-ADJ). Part names match the Fusion schematic.

| Ref | Value | Part | Footprint in schematic |
|---|---|---|---|
| U1 | LM2596T-ADJ | Step-down regulator, TO-220-5 | TO170P1015X457X1845-5 |
| D1 | 1N5822 | 3 A Schottky diode, DO-201AD | DIOAD1990W125L900D530 |
| L1 | 68 uH | Toroid inductor, through-hole | IND_HCTI-68-7.7 (check against real part) |
| C_IN | 470 uF, 25 V or more | Radial electrolytic | CAPPRD500W60D1000H1250 |
| C_OUT | 220 uF, 10 V or more | Radial electrolytic | CAPPRD500W50D500H1100 (check can size) |
| C_BYP | 100 nF | Ceramic, through-hole | CERAMIC-2.54MM |
| R1 | 1 kohm | 1/4 W axial, 1% | 0207/10 |
| R2 | 2.2 kohm | 1/4 W axial, 1% | 0207/10 |
| R3 | 680 ohm | 1/4 W axial, 1% | 0207/10 |
| R4 | 220 ohm | 1/4 W axial, 1% | 0207/10 |
| J_IN | TB007-508-02BE | 2-pin 5.08 mm terminal | CUI_TB007-508-02BE |
| J_OUT | TB007-508-02BE | 2-pin 5.08 mm terminal | CUI_TB007-508-02BE |

Off board: 12 V 2 A wall adapter, inline 2 A fuse, 5 ohm 25 W load resistor, TO-220 heatsink for U1.

Output voltage: Vout = 1.23 V x (1 + (R2 + R3 + R4) / R1) = 1.23 x (1 + 3100 / 1000) = 5.04 V predicted.

Measured on the built board: 4.98 V with no load, 4.97 V with the 5 ohm load, 1.23 V at the FB pin.