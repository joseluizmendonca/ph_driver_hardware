## Peak-and-Hold / Saturation Injector Driver (Hardware)

This repository contains the hardware for an automotive injector driver board that supports both Peak-and-Hold (PH) and Saturation drive modes. It's intended for flexible use across different injector types and engine setups.

The project is a simplified/ downtuned version of the sytem that I designed for my bachelors thesis for driving methanol injectors which would be overkill for most aplications, and is not yet public

Design files are in `src/`.

## Photos / Renders

Below are images from the board.

![Side view render](./pics/render4k_sidehigh.png)

![Top view render](./pics/render4k_top.png)

## Features

- Supports both injector drive strategies:
	- Peak-and-Hold (PH) for low-impedance injectors
	- Saturation for high-impedance injectors
- Works with both high-impedance and low-impedance injectors
- Stackable: multiple boards can be placed one on top of another to drive more injectors
- KiCad project files included (`.kicad_sch`, `.kicad_pcb`) under `src/`
- Max switching frequency: under test (TBD)
- Proven setup: tested on a Weber 2-cylinder engine with ~12 Ω injectors, and also on low impedance injectors on a testbench
- Cost-conscious design: uses a common N‑channel MOSFET with discrete flyback/clamp diodes to manage inductive kickback, avoiding expensive automotive‑rated ICs where practical
- MCP1406 MOSFET gate driver for fast gate charge delivery and sharp turn‑on/turn‑off
- Adjustable flyback clamp: injector shutoff speed can be tuned by changing the output‑side Zener diode value

## Peak-and-Hold vs. Saturation (Quick primer)

Automotive injectors are solenoid valves. To open quickly and control heating, there are two common drive strategies:

- Peak-and-Hold (PH):
	- Typically used with low-impedance injectors (e.g., 1–3 Ω).
	- A high initial current “peak” is applied to open the injector rapidly, then the current is reduced to a lower “hold” level to keep it open while minimizing power dissipation and coil heating.
	- Requires current control (sensing + limiting) and a fast driver.

- Saturation:
	- Typically used with high-impedance injectors (e.g., 12–16 Ω).
	- The driver applies battery voltage and lets the current rise to a safe level dictated by the injector’s resistance. Simpler electronics, slower opening than PH for low-impedance coils but appropriate for high-impedance parts.
	- Current limiting is less complex; thermal stress is inherently lower due to higher coil resistance.

This board is designed to accommodate both approaches so you can pair the drive mode with your injector type. Use PH for low-impedance injectors to achieve fast opening while controlling heat, and use saturation mode for high-impedance injectors for a simpler setup.

## Design goals (cost and availability)

The main goal was to build a robust injector driver without relying on expensive, automotive‑rated monolithic components. Instead, the design:

- Uses a common N‑channel power MOSFET as the switching element
- Protects it using discrete diodes and a Zener clamp against the solenoid’s inductive kickback
- Adds a dedicated MOSFET gate driver (MCP1406) to ensure fast and repeatable switching

This keeps the BOM accessible and cost‑effective while maintaining performance suitable for practical engine control setups.

## Inductive kickback, clamping, and shutoff time

When turning the injector off, the coil’s stored energy forces current to continue flowing. A clamp network provides a safe path and defines the voltage across the coil, which sets the current decay rate. In simple terms, di/dt ≈ Vclamp / L, so a higher clamp voltage yields faster current fall and a quicker injector shutoff.

- Adjustable clamp via Zener: You can change the Zener diode on the output side to set the clamp voltage.
- Trade‑offs:
	- Higher clamp voltage → faster shutoff time, but the MOSFET must withstand a higher VDS during the event and the clamp network must absorb more energy.
	- Lower clamp voltage → reduced MOSFET voltage stress and a “safer” environment for the switch, but slower shutoff time (longer tail current).

Choose the Zener rating considering MOSFET VDS limits, injector L/R, energy dissipation in the clamp path, and thermal margins. The included simulations and schematics in `src/` show the intended topology. The BOM references common Zener parts (for example, 1N5338B family) that can be swapped to suit your target clamp level.

## Falstad demos (kickback concept)

Small interactive simulations demonstrating the kickback clamp concept are available on Falstad:

- [Single diode vs Zener clamp](https://www.falstad.com/circuit/circuitjs.html?ctz=CQAgjCAMB0l3BWK0BsB2AnADgEwI5gMw6SFZYQAsOISlhtApgLRhgBQANiMeGDb140YkCMwzQ0bBCkKEMOQmkUJCLElHYATPjQwpdISlhA0tjAGYBDAK6cALu0KUQFgO5aATgHsADgB0ARxgsSAMIMEkwegQcCgUKeBwWNChTWHhMrMgsFw0OAC9DZl42ARoady8-IJCw9jceGjLDFsgGw31W-k1G3mZjYsH2xpaBkzHeEcNSnuH2ADcQZhQXQZWXEuE0ykhwbb2YBHYAc2XVoxMNkAvDjuut8835xofB-vmAZyaZit20iDWTifRinH44f5CchpdqeH6CGjjAE4dhw+jlIwfEx7fhg9Gmf74uLYzTffEtIn-HGuKzA0F9QgGJH4pHTORMwSM5bzM7sy48Lm3TRLPnvLmPQ5GHEHZDHRrknoIHptbS0HoXJXNAxmSy2BxcTHNOYI0zIUS0aCUagYMCxFAIfDkDAIFhgDC9Q3ckyar0en0tH23Dr+gw+1nsbw8U17XYYd3-SIGPYCHgR0wVHaQOOmHGocCmnACdhAA)

- [FET ringing mitigation](https://www.falstad.com/circuit/circuitjs.html?ctz=CQAgjCAMB0l3BWEDoBYEE4AcYFwMz5iqoBMqIZyl+yApgLRhgBQAbuBqSIQGyfdSkLJWq9I4bhOnQELAGYhSWEUJH5eq4T25hZUWKRYAnAUu3KtIiRlSQWAYyUrwAdn6XJIimEbMQTKSw2LjMkBiuWBjRrgZwEDCsADY8ml6p3tZxEAww8HAYkORwJFh2GGDYUiwAJpSoIszcpSC8FNw1dPIAhgCuSQAuLABe9SJM-C0NSiCdPf0DDMN0AHZ0xiwA7s5WO2789tueanvFUCwA5mZ81whS51ca6kIZPHdQ59tPSuSv+C+HPZgZSvYHWLagkEtMGfMbgYFwmGA74Mf5w1HVbYtDHosAHCFTbzTNrnDikVzcEnk7gTD4SOySOkGORXamtdoUkBYaSwtm0vlowEC7gowUmV4477TCTAiHfSoitJIlgAe3AtAo9PCGHMBiyulo+BY+FUIAAYgkPmB4AEIAAlOgAZwAlo6Bt0Vg46CwgA)

## Scaling up (stacking boards)

The board is stackable: you can assemble and stack multiple boards to drive a higher number of injectors. Ensure proper power distribution, grounding, and cooling when scaling up, and verify your controller’s channel count and timing strategy.

## Assembly notes

Due to the size of the power planes attached to the main MOSFET, hand soldering those components is not recommended—the planes wick heat and make it difficult to achieve proper wetting.

Recommended approaches:

- Hot plate or reflow oven for consistent thermal profile
- Fabricator assembly (e.g., JLCPCB). Assembly files are included in `src/` to support turnkey assembly

If you must rework by hand, preheat the board, use ample flux, and use a proper soldering iron.

## SMD MOSFET thermal note

The latest revision uses surface‑mount MOSFETs. Thermal vias were added under the main drain pad to spread heat into inner and bottom copper. In bench tests so far, device temperatures remained fully acceptable, so external heatsinks are not required for typical operating conditions.

## Status

- Max switching frequency: still under test.
- Tested on a Weber two-cylinder engine using injectors of approximately 12 Ω impedance.

## Directory

- `src/` – KiCad schematics/PCB, libraries, production assets.
- `3d_render/` – Blender/KiCad 3D assets for visuals.
- `pics/` – Screenshots and renders used in this README.
