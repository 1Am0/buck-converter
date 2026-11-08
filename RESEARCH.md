# Designing and Building a 12 V to 5 V Buck Converter

Research and build notes for the buck converter in this repo.

## Overview

I built a 12 V to 5 V, 1 A buck converter around the LM2596T-ADJ on a 7 x 9 cm perfboard. After tuning the feedback divider it measured 4.98 V with no load and 4.97 V at a full 1 A load.

The design started in LTspice, moved to a Fusion Electronics schematic (ERC clean), and was built by hand from through-hole parts. The final feedback divider needed a combination of four common resistors, because the exact values the math asked for were not in my parts bin.

| Spec | Target | Result |
|---|---|---|
| Input | 12 V adapter | 14.5 V measured with no load |
| Output | 5.00 V | 4.98 V no load, 4.97 V at 1 A |
| Load current | 1 A | 1 A (5 ohm, 25 W load resistor) |
| Regulator | LM2596T-ADJ | TO-220-5, 150 kHz switching |
| Build | Perfboard | 7 x 9 cm, wires on the solder side |

## How the Converter Works and Why the LM2596

A buck converter steps voltage down efficiently by switching the input on and off about 150,000 times a second and smoothing the result, instead of burning the extra voltage as heat the way a linear regulator does.

While U1's internal switch is on, current flows from the input through L1 into the output and charges C_OUT. When the switch turns off, L1 keeps pushing current, and D1 (a Schottky diode) gives that current a path back around, so the output never sees a gap. The ratio of on-time to total time sets the output: roughly Vout = D x Vin, so 5 V from 12 V means the switch is on about 42% of the time. The regulator adjusts that duty cycle on its own to hold the output steady.

I chose the LM2596T-ADJ because it contains the switch, oscillator, reference and error amplifier in one TO-220 package, handles 3 A, and needs only an inductor, a diode, two capacitors and a resistor divider around it. It is also a through-hole part, which suits perfboard.

The pins mattered in practice, and mixing them up was one of the early confusions:

1. Pin 1, VIN: the input supply.
2. Pin 2, OUTPUT: this is the switch node, not the regulated output. It feeds L1 and the cathode of D1.
3. Pin 3, GND.
4. Pin 4, FB: feedback, which the chip regulates to 1.23 V.
5. Pin 5, ON/OFF: tied to ground to keep the chip enabled.

## How We Selected the Parts

Each part was picked to match a job in the switching loop, and a few choices changed once I looked at what was physically available.

| Ref | Part | Why this one |
|---|---|---|
| U1 | LM2596T-ADJ | Integrated 3 A switcher, through-hole TO-220, adjustable output |
| L1 | 68 uH toroid (external coil) | The inductor sets ripple current; it must be rated above 1 A and not saturate. I had to search for the coiled through-hole type rather than a surface-mount one |
| D1 | 1N5822 Schottky | Low forward drop and fast recovery; a normal diode would waste more power and be too slow at 150 kHz |
| C_IN | 470 uF, 25 V or more | Bulk capacitor that supplies the input current pulses; voltage rating needs margin over the 14.5 V seen at the input |
| C_BYP | 100 nF ceramic | Handles the fast, high-frequency spikes that the large electrolytic is too slow for |
| C_OUT | 220 uF, 10 V or more | Smooths the output ripple; 220 uF covers the 5 V rail with margin |
| R1 to R4 | 1% axial resistors | Set the output voltage (see the divider section) |
| J_IN, J_OUT | 2-pin screw terminals | Easy connection for the adapter and load |

Two details are worth remembering. Electrolytic capacitors are polarized, so the long lead or the side without the stripe goes to the positive net, while ceramics have no polarity. The stripe on the diode marks the cathode, which connects to the switch node.

The two input capacitors are in parallel on purpose: the big electrolytic stores charge but responds slowly, and the small ceramic responds instantly to the switching edges. Together they cover the full range of frequencies the regulator draws current at.

The screw terminals I received turned out to be the angled two-part type rather than simple pins. That did not affect the electronics, only the footprint, so I noted it in the BOM as a footprint to check.

## The Feedback Divider and the Math

The output voltage is set by a resistor divider, and the formula comes from one idea: the LM2596 adjusts its output until the voltage at the FB pin equals its 1.23 V reference.

The divider is a series chain from the output, through the top resistors, to the FB node, then through R1 to ground. FB taps the chain just above R1, so R1 carries the fraction of the output that the chip compares against 1.23 V.

    Vout = Vref x (1 + Rtop / R1),   Vref = 1.23 V

Solving for the top resistance when R1 = 1 kohm and the target is 5.00 V:

    Rtop = R1 x (Vout / Vref - 1) = 1000 x (5.00 / 1.23 - 1) = about 3065 ohm

The first plan used 3 kohm plus 68 ohm in series (3068 ohm), which gives 1.23 x (1 + 3.068) = 5.00 V. That clicked once I walked through the current in the chain: the same current flows through every resistor in series, so with 4068 ohm total and 5 V across it, the current is 5 / 4068 = 1.23 mA, and that current times R1 (1 kohm) is 1.23 V. The FB pin sits at exactly the reference, which is why the regulator stops adjusting.

Two misconceptions came up along the way and are worth recording. First, the chain is not a path that dumps current to ground before reaching R1; it is a single series loop where R1 is simply the last element. Second, the reference is 1.23 **volts**, not amps; the 1.23 mA figure above is a coincidence of using 1 kohm for R1.

The 5 ohm load resistor is not part of the divider at all. It sits across the output and only draws about 1 A so the converter can be tested at full current.

## Problems We Hit and How We Solved Them

The hardest problems were not in the theory; they were getting a clean schematic and then matching the divider to resistors I actually owned. In order:

| # | Problem | Cause | Fix |
|---|---|---|---|
| 1 | Fusion was slow and laggy | Cloud syncing of the design | Looked at alternatives, then stayed in Fusion and kept going |
| 2 | Needed through-hole R and C parts | Not obvious where they live in the library | Used the `rcl` library for axial resistors and a ceramic disc from the Tinkercad library |
| 3 | Screw terminals were a different type than planned | Received angled two-part terminals | Footprint noted as a placeholder; electrically identical |
| 4 | ERC said U1's GND pin was on a net named `n$2 1` | The ground wires were drawn as separate nets | Renamed the net to `GND` so every ground connects; ERC then showed zero errors |
| 5 | Unsure what ON/OFF and FB connect to | Pins easy to confuse with OUTPUT | ON/OFF to ground, FB to the divider tap, and pin 2 is the switch node, not Vout |
| 6 | 3 kohm and 68 ohm were not in my parts bin | I only owned common E-series values | Searched combinations of what I had (see next section) |
| 7 | First reading was 5.4 V | A resistor in the chain was skipped in the wiring | Rewired the chain and re-measured |
| 8 | Readings of 5.2 V, then 4.77 V, then 4.78 V | A mix of a wiring slip, partial chain, and meter effects | Rebuilt the chain one resistor at a time, checking FB each time |
| 9 | A 2.2 kohm resistor read 1.6 kohm | I measured it while it was still in the circuit | In-circuit readings include parallel paths, so measure resistors out of the circuit |
| 10 | U1 got very hot at 1 A | A TO-220 with no heatsink dissipates about 1 to 1.5 W here | Unplugged it, planned a heatsink and a lighter test load |

Problems 7 and 8 are the ones I am least able to fully explain. The pattern was that each wiring correction moved the output closer to the prediction, and the FB pin read 1.23 V to 1.27 V throughout, which fits the datasheet reference range. I cannot say with certainty which single mistake caused each intermediate number, only that the final chain matched the calculation once it was wired exactly as designed.

## The Final Resistor Combination and Results

The winning divider is R1 = 1 kohm to ground with 2.2 kohm + 680 ohm + 220 ohm in series on top, which totals 3100 ohm and predicts 5.04 V; the board measured 4.98 V.

Because 1% resistors of the exact values were not available, the approach was to keep R1 at 1 kohm and build the top half out of series combinations of values I owned, then compare each prediction to a measurement.

    Vout = 1.23 x (1 + (2200 + 680 + 220) / 1000) = 1.23 x 4.1 = 5.04 V

| Top chain | Total | Predicted | Measured |
|---|---|---|---|
| 3 kohm + 68 ohm (original plan, not on hand) | 3068 ohm | 5.00 V | not built |
| 2.2 kohm + 680 ohm | 2880 ohm | 4.77 V | not built as a stage |
| 2.2 kohm + 680 ohm + 47 ohm | 2927 ohm | 4.83 V | 4.77 to 4.78 V |
| 2.2 kohm + 680 ohm + 220 ohm (final) | 3100 ohm | 5.04 V | 4.98 V no load, 4.97 V at 1 A |

The final result is about 1.2% below the prediction. That gap is inside what I would expect: the divider resistors are 1%, and the LM2596's reference is specified over a range (about 1.19 V to 1.27 V), so a real chip rarely sits at exactly 1.23 V. The FB pin read 1.23 V on the finished board, which confirms the chip was regulating to the value used in the formula.

The more important result is that the output held at 4.97 V with the full 5 ohm load, a drop of only 0.01 V from no load.

## Heat, Lessons Learned, and the Move to a PCB

The perfboard version works, but U1 runs hot at full load, and that is the main thing the PCB version should fix.

At 1 A with the input near 14.5 V, the LM2596 dissipates roughly 1 to 1.5 W in its switch and control circuitry, which is a lot for a bare TO-220. The fixes are a heatsink on the tab (after checking by continuity whether the tab is grounded), airflow, and testing with a 10 ohm load (about 0.5 A) for longer runs. A PCB can also spread heat into a copper pour under the tab.

What I would carry forward:

- Measure resistors out of the circuit, and measure FB first when the output is wrong; it tells you whether the divider or the chip is at fault.
- Name the ground net so every ground connects, and run ERC after every wiring change.
- Keep the switch-node loop (U1, D1, ground) short, and keep the FB wire short and away from L1, since L1 radiates noise.
- Check the real footprints of L1 and C_OUT before ordering, because the schematic ones are placeholders.

Next step is the PCB. The schematic is already in Fusion Electronics, so the board work starts by generating the board from it, then placing U1, D1, L1 and the capacitors close together, routing the high-current switching loop first with wide traces, and adding a ground pour and thermal relief for U1. The perfboard measurements (4.98 V no load, 4.97 V at 1 A) are the baseline the PCB should match or beat.
