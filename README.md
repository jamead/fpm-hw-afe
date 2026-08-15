# Fill Pattern Monitor

Bunch-by-bunch **Fill Pattern Monitor** RF front end for the NSLS-II BPM system.

The design uses spare RFSoC ADC channels to digitize the same BPM button signal at two different effective gain ranges. A direct path provides large-signal headroom while a programmable high-gain path provides sensitivity for smaller bunch signals.

## RF Architecture

```text
BPM Button
    |
    v
RCAT-01+              1 dB broadband input pad
    |
    v
XLF-172H+             Reflectionless low-pass filter
    |
    v
RPS-2-30+             2-way RF power splitter
   / \
  /   \
 /     \
v       v

LOW-GAIN PATH                 HIGH-GAIN PATH

Direct                         PE43713B-Z
  |                         Programmable attenuator
  |                              |
  |                              v
  |                           ADL5611
  |                        Wideband amplifier
  |                              |
  |                              v
  |                          XLF-172H+
  |                       Reflectionless LPF
  |                              |
  v                              v
RFSoC ADC                     RFSoC ADC
```

The splitter is intentionally placed **before** the programmable attenuator so that the low-gain ADC always sees the largest available signal while the high-gain path can be adjusted independently.

## Design Goals

- Measure bunch-by-bunch fill pattern with a 5 GS/s RFSoC ADC.
- Preserve the time-domain BPM button pulse shape.
- Provide simultaneous low-gain and high-gain acquisition.
- Extend usable dynamic range without an RF bypass switch.
- Maintain a well-behaved 50 Ω environment at the BPM input.
- Reduce reflected out-of-band energy returning toward the BPM button.
- Provide programmable attenuation ahead of the high-gain amplifier.
- Limit out-of-band noise and harmonics before digitization.

## Key Components

| Function | Part | Notes |
|---|---|---|
| Input pad | **RCAT-01+** | 1 dB broadband fixed attenuator |
| Input low-pass filter | **XLF-172H+** | Reflectionless low-pass filter |
| 2-way splitter | **RPS-2-30+** | Splits the BPM signal into low/high gain paths |
| Programmable attenuator | **PE43713B-Z** | Controls high-gain path level |
| High-gain amplifier | **ADL5611** | Wideband gain block |
| High-gain output filter | **XLF-172H+** | Removes out-of-band amplifier noise/harmonics |
| ADC platform | **RFSoC** | 5 GS/s ADC acquisition |

## Why a Reflectionless Input Filter?

A conventional low-pass filter can present a poor impedance match in its stopband and reflect rejected high-frequency energy back toward the BPM button.

The XLF-series reflectionless filter absorbs much of this rejected energy instead. Placing it near the front of the chain helps maintain a better broadband termination for the BPM cable and reduces re-reflections.

The RCAT input pad provides additional broadband damping of downstream impedance errors.

## Low-Gain Path

The low-gain path is intended for large bunch signals and maximum headroom.

```text
RPS-2-30+ -> RFSoC ADC
```

An optional fixed attenuator footprint can be included in this path if additional ADC headroom or impedance isolation is required during testing.

## High-Gain Path

```text
RPS-2-30+
    |
PE43713B-Z
    |
ADL5611
    |
XLF-172H+
    |
RFSoC ADC
```

The programmable attenuator is placed **before the ADL5611** so that large signals can be reduced before reaching the amplifier.

This helps keep the amplifier out of compression while allowing the FPGA/processor to optimize the high-gain ADC level.

## ADL5611 Bias Network

The ADL5611 is operated from the +5 V rail.

Current schematic values:

```text
+5 V
 |
43 nH
 |
+---------------- RF OUT / bias
|
+-- 68 pF  -> GND
+-- 1.2 nF -> GND
+-- 1 uF   -> GND
```

Input and output DC-blocking capacitors are retained around the amplifier.

The bias components should be placed very close to the device and connected to a low-inductance ground structure.

## BPM Input

The BPM button is a capacitive pickup and does not provide a DC beam component. The passive front-end chain therefore does not require an additional AC-coupling capacitor solely for DC removal.

Preferred input chain:

```text
SMA -> RCAT-01+ -> XLF-172H+ -> RPS-2-30+
```

Avoid unnecessary series components in this section because their parasitics can affect the multi-GHz pulse response.

## PCB Layout Guidelines

The RF layout is as important as the schematic.

- Maintain **50 Ω controlled-impedance transmission lines** throughout the single-ended RF chain.
- Keep RF traces short and direct.
- Avoid stubs on RF nets.
- Place the RCAT-01+ and input XLF-172H+ close to the input connector.
- Place the RPS-2-30+ close to the point where the two signal paths diverge.
- Place the PE43713B-Z close to the ADL5611.
- Place the ADL5611 bias choke and bypass capacitors immediately adjacent to the amplifier.
- Use a continuous ground plane underneath the RF section.
- Use dense ground vias around filters, splitter, amplifier, connectors, and transmission-line transitions.
- Follow the manufacturer's recommended land patterns for RF components.
- Ground all pins that the XLF-172H+ datasheet specifies as externally grounded, including the exposed paddle.
- Keep digital attenuator control traces away from sensitive RF traces where practical.

## Dynamic Range Strategy

Both ADCs can be captured simultaneously.

Firmware can select which measurement to use based on signal level:

```text
High-gain ADC not near clipping
        |
        +---- yes ---> use high-gain data
        |
        +---- no ----> use low-gain data
```

The overlap between the two ranges can also be used for calibration and consistency checks.

## Calibration

Each RF path should be characterized independently because the two paths have different gain and frequency response.

Recommended calibration measurements include:

- Gain versus frequency
- Phase versus frequency
- Channel-to-channel delay
- Pulse response
- ADC full-scale level
- High-gain compression point
- PE43713 attenuation accuracy
- Low/high gain overlap
- Noise floor
- Return loss at the BPM input

Frequency-dependent correction can be applied in FPGA or software if required.

## Repository Notes

This repository contains the KiCad hardware design for the Fill Pattern Monitor front end.

KiCad-generated backup archives and other temporary files should not be committed to the repository. Keep source schematic, PCB, symbol, footprint, and project files under version control while excluding generated backup/temporary files through `.gitignore`.

## Status

**Design in progress.**

Current work includes:

- RF signal-chain optimization
- Final filter bandwidth selection
- RFSoC ADC interface design
- PCB footprint verification
- RF layout
- Dynamic-range and gain budgeting
- Bench validation of the low- and high-gain paths

## Project

**NSLS-II Bunch-by-Bunch Fill Pattern Monitor**  
RFSoC-based BPM diagnostics front end
