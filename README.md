# Verilog-A Models for MAGIC Memristive Crossbars

Verilog-A memristor models I use to simulate **MAGIC (Memristor-Aided loGIC)** NOR/NOT gates, adders and multipliers on memristive crossbars in **Cadence Spectre/Virtuoso** and **ngspice (OSDI)**.

## Files
| File | What it is |
|---|---|
| `veriloga/vteam_fixed.va` | VTEAM memristor (Biolek window, p = 2) with the state integrated by the simulator. **Use this one.** |
| `veriloga/vteam_technion_original.va` | Original VTEAM Verilog-A model from the Technion (Kvatinsky group), unmodified, kept for reference |

## Why `vteam_fixed.va`
The original model updates the state as `x = x_last + dt*dxdt` once per model evaluation. Spectre evaluates a model several times per timestep (Newton iterations, rejected steps), so the switching time depends on `maxstep`/`reltol`.

`vteam_fixed.va` uses the same equations (VTEAM, Biolek window, linear R–x relation) but stores the normalised state x/D on an internal node and lets the simulator integrate it with `ddt()`. The result is timestep-independent.

## Device parameters (defaults)
| Parameter | Value | Parameter | Value |
|---|---|---|---|
| `Ron` | 1 kΩ | `v_on` | −1.5 V |
| `Roff` | 300 kΩ | `v_off` | 0.3 V |
| `k_on` | −216.2 m/s | `k_off` | 0.091 m/s |
| `alpha_on` | 4 | `alpha_off` | 4 |
| `D` | 3 nm | `init_state` | 1 (Roff) |

**Conventions**
- Logic 1 = Ron (x/D = 0), and logic 0 = Roff (x/D = 1).
- Connect the p-terminal to the **row** and the n-terminal to the **column**.
- `V(p,n) > v_off` → RESET (towards Roff). `V(p,n) < v_on` → SET (towards Ron).
- Pin `w` outputs x/D as a voltage (0–1 V), so you can plot the state directly.

## How to implement

**Cadence Virtuoso / Spectre**
1. In your library, create a cell `vteam_fixed` → **File → New → Cellview**, type **VerilogA**. Paste `vteam_fixed.va` and save. Virtuoso compiles it and offers to create a symbol; accept.
2. Place the memristor symbol in your schematic. Set `init_state` per instance: 0 for logic 1 (Ron), 1 for logic 0 (Roff).
3. ADE → **Transient**. For MAGIC steps of a few ns, a `maxstep` of 10 ps is a good start.
4. Plot `w` to watch each memristor switch, and the branch current to measure energy.

**Spectre netlist (text)**
```
ahdl_include "veriloga/vteam_fixed.va"
M1 (row1 col1 w1) vteam_fixed init_state=1
```

**ngspice (open source)**
1. Compile the model with [OpenVAF](https://openvaf.semimod.de): `openvaf veriloga/vteam_fixed.va` → `vteam_fixed.osdi`
2. In the netlist:
```
.control
pre_osdi vteam_fixed.osdi
.endc
N1 row1 col1 w1 vteam_model
.model vteam_model vteam_fixed init_state=1
```
3. Run a `.tran` analysis and plot `v(w1)`.

**Quick check: a MAGIC NOT gate**
1. Put an input memristor and an output memristor in series, in opposite orientation. The output starts at Ron (`init_state=0`).
2. Apply V0 ≈ 1.4 V across the pair for a few ns.
3. Input at Ron → the output switches to Roff (NOT 1 = 0). Input at Roff → the output stays at Ron (NOT 0 = 1).

## References
1. S. Kvatinsky, M. Ramadan, E. G. Friedman, A. Kolodny, "VTEAM: A General Model for Voltage-Controlled Memristors," *IEEE TCAS-II*, 2015.
2. S. Kvatinsky et al., "MAGIC—Memristor-Aided Logic," *IEEE TCAS-II*, 2014.
3. Z. Biolek, D. Biolek, V. Biolková, "SPICE Model of Memristor with Nonlinear Dopant Drift," *Radioengineering*, 2009.

`vteam_technion_original.va` is the Technion's model, redistributed unmodified with its original header. `vteam_fixed.va` re-implements the same equations.

## Contributing
I'm open to open-source contributions and collaboration. Issues and pull requests are welcome.
You can reach me at **vipulatluri98@gmail.com**.
