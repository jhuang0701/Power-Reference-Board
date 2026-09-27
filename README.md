# 3-Phase FOC BLDC Motor Driver with Onboard IMU

A custom 4-layer PCB for field-oriented control (FOC) of a brushless DC motor, with an onboard IMU for real-time orientation feedback. Designed in KiCad, targeting a self-balancing single-wheel robot or reaction-wheel inverted pendulum as the demo application.

This is a design-only project: schematic and layout are complete and DRC-clean. Fabrication is a separate decision, not yet made.

## What it does

The board takes a 6S battery input (22.2V nominal, up to 25.2V) and drives a 3-phase BLDC motor using field-oriented control, with current feedback and IMU data available for a real-time stabilization loop. It's built around three main subsystems: power delivery and protection, the 3-phase gate drive and half-bridge power stage, and the sensing/control electronics.

## Core components

- **MCU:** STM32G474RET6 (LQFP-64). Chosen specifically for its onboard op-amps, which are used directly for current-sense amplification, and for TIM1's hardware dead-time generation on complementary PWM channels.
- **Gate driver:** DRV8353HRTAT, a 102V-rated 3-phase smart gate driver in hardware-interface mode (no SPI on this variant), configured via resistor-divider pins for 6x independent PWM mode.
- **Phase MOSFETs:** 6x CSD18540Q5BT (60V, 2.2mΩ RDS(on)), two per phase leg.
- **IMU:** ICM-42688-P, 6-axis accelerometer/gyroscope, connected over SPI.
- **Buck converter:** TPS54331DR, steps the battery rail down to a clean 3.3V supply for the MCU and IMU.
- **Reverse-polarity protection:** a 7th CSD18540Q5BT, wired low-side in the ground return path, with a Zener-clamped gate bias network. (An earlier design used a small P-channel MOSFET for this; it was replaced after checking its actual gate-voltage rating against the battery voltage and finding it wouldn't survive.)

## Design decisions worth knowing about

**Dual current-sensing paths.** The DRV8353HRTAT's built-in shunt amplifiers are used only for the driver's own hardware overcurrent protection, independent of the MCU. The actual current feedback used for the FOC control loop comes from the STM32G474's internal op-amps, wired directly to the same shunt resistors. Two separate paths, one fast and hardware-only for protection, one precise and MCU-driven for control.

**Hardware dead-time via TIM1.** Rather than relying on the gate driver's default switching behavior, all six gate signals (INHx/INLx) are driven independently from the MCU's TIM1 timer, which generates dead-time insertion in hardware. This gives direct control over switching behavior rather than trusting a fixed driver-side default.

**Mixed-signal supply isolation.** VDDA (the MCU's analog supply, feeding the ADC and current-sense op-amps) is isolated from the main 3.3V digital rail with a series resistor and its own local decoupling, keeping switching noise out of the current measurements.

## Specs

| | |
|---|---|
| Input | 6S Li-ion, 22.2V nominal / 25.2V max |
| Phase current | 15A continuous, 30A peak |
| Board | 4-layer, 60x60mm, 2oz outer copper |
| MCU | STM32G474RET6 |
| Gate driver | DRV8353HRTAT |
| IMU | ICM-42688-P (SPI) |

## Repo contents

- `/schematic` - full schematic PDF export
- `/gerbers` - fabrication-ready Gerber files
- `/bom` - bill of materials
- `/renders` - 3D board renders and layout screenshots

## Status

Schematic and layout complete, DRC-clean. Fabrication not yet started.
