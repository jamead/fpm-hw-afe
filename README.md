# NSLS-II Fill Pattern Monitor Analog Front End

Analog front-end (AFE) board for the NSLS-II Fill Pattern Monitor.

![NSLS-II Fill Pattern Monitor analog front-end board](fpm-afe.png)

| Item | Description |
|---|---|
| Drawing number | E-SK-2604 |
| Revision | 1 |
| Date | September 20, 2026 |
| Design software | KiCad |
| PCB | Six layers, 1.6 mm nominal thickness |

## Overview

The board accepts a broadband RF signal and produces two simultaneous 50-ohm outputs:

- **Low-gain output:** a direct, filtered monitor of the input signal.
- **High-gain output:** a filtered and amplified signal with digitally programmable attenuation.

The two outputs provide sufficient dynamic range to observe both large and small signals without changing the input connection. The programmable attenuator in the high-gain path allows its level to be adjusted for the acquisition system while reducing the risk of amplifier or ADC saturation.

## RF signal path

The signal enters the board through **J2** and first passes through **U8 (RCAT-01+)**, a fixed attenuator used to improve the input match and provide isolation. It then passes through **U4 (XLF-172H+)**, which limits the RF bandwidth before the signal reaches **U7 (RPS-2-30+)**, a two-way power splitter.

```text
J2 RF input
  -> U8 fixed attenuator
  -> U4 input filter
  -> U7 two-way splitter
       |-> J5 low-gain output
       |
       `-> U2 digital step attenuator
           -> U3 RF gain block
           -> U5 output filter
           -> J3 high-gain output
```

The RF input and both outputs are AC-coupled. Board-level RF interconnects are designed as 50-ohm transmission lines.

### Low-gain output

One output of the splitter is AC-coupled directly to **J5**. This path contains no active gain stage and provides a conditioned copy of the input for larger signals and diagnostic measurements.

### Programmable high-gain output

The second splitter output is AC-coupled to **U2 (PE43713B-Z)**, a digitally controlled RF step attenuator. U2 sets the signal level applied to **U3 (ADL5613)**, a broadband RF gain block. The amplifier supply is delivered through bias inductor **L2** and is locally bypassed by the surrounding capacitors.

After amplification, the signal passes through **U5 (XLF-172H+)** and is AC-coupled to **J3**, the high-gain output. The resulting gain depends on the fixed losses of the input network and splitter, the programmed attenuation of U2, the gain of U3, and the insertion loss of the output filter.

## Digital control

The PE43713 attenuator is controlled through **J4** using a three-wire serial interface:

- Serial data
- Serial clock
- Latch enable

The three control lines pass through 22-ohm series resistors **R5-R7**. Resistors **R1** and **R2** establish the required logic state for the attenuator's power-up/select input.

## Power supply

External DC power enters through **J1**. **U1**, a low-noise linear regulator, generates the local **+5.0 V** supply used by the RF circuitry. Bulk and high-frequency bypass capacitors are placed around the regulator and RF devices to keep the supply impedance low across the operating bandwidth.

## Connector summary

| Reference | Function |
|---|---|
| J1 | DC power input |
| J2 | RF input |
| J3 | High-gain RF output |
| J4 | Digital attenuator control |
| J5 | Low-gain RF output |

## PCB construction

The board uses the PCBWay standard six-layer stackup with solid ground reference planes on **L2** and **L5**. Top-layer RF traces are routed as 50-ohm grounded coplanar waveguide referenced to L2. Ground-stitching vias along the RF paths and around the connector transitions maintain a short return path and reduce unwanted coupling.

The RF components are placed in signal-flow order to minimize interconnect length. The four plated mounting holes are connected to ground and can bond the PCB ground to the enclosure.

Via-in-pad vias are specified to be filled with nonconductive epoxy, planarized, copper capped, and plated over in accordance with **IPC-4761 Type VII**.

## Design and fabrication notes

- The low-gain and high-gain outputs operate simultaneously.
- The high-gain output level depends on the programmed PE43713 attenuation.
- Preserve the specified 50-ohm geometry through RF pads and connector launches.
- Keep the L2 reference plane continuous beneath all top-layer RF traces.
- Fabricator changes to the stackup, dielectric materials, trace width, or coplanar gap require engineering approval.
- Review the schematic, PCB, bill of materials, and fabrication outputs together before ordering boards.

## Repository contents

The repository contains the KiCad schematic and PCB layout for the AFE. Depending on the release, it may also include project-specific symbols and footprints, the bill of materials, fabrication files, assembly information, and mechanical drawings.

## Organization

Brookhaven National Laboratory  
National Synchrotron Light Source II (NSLS-II)
