# CMOS Digital Cell Design — Cadence Virtuoso (UMC 90nm)

**Tool:** Cadence Virtuoso 6.1.x · **PDK:** UMC 90nm (SP) · **Verification:** Assura DRC/LVS/QRC

Full custom design flow for a CMOS inverter, tristate inverter, and D flip-flop — from schematic through layout, parasitic extraction, and post-layout simulation.

---

## Design Flow

```
Schematic (Virtuoso SE)
      ↓
  Symbol Creation
      ↓
  Layout (Virtuoso Layout Suite GXL)
      ↓
  DRC → LVS (Assura)
      ↓
  Parasitic Extraction — av_extracted view (QRC)
      ↓
  Post-Layout Simulation (ADE)
```

---

## Cells Designed

### 1. CMOS Inverter

| Parameter | PMOS | NMOS |
|-----------|------|------|
| Model | `p_10_sp` | `n_10_sp` |
| W | 800 nm | 480 nm |
| L | 80 nm | 80 nm |
| Fingers | 1 | 1 |

- Schematic + symbol created in Virtuoso Schematic Editor
- Full-custom layout completed using UMC 90nm design rules
- Assura DRC and LVS clean
- `av_extracted` view generated via QRC parasitic extraction
- Post-layout transient simulation verified correct inverting behaviour

---

### 2. Tristate Inverter

| Parameter | PMOS (×2) | NMOS (×2) |
|-----------|-----------|-----------|
| Model | `p_10_sp` | `n_10_sp` |
| W | 1.6 µm | 800 nm |
| L | 80 nm | 80 nm |

- Control signal `ne` (active-low enable) gates output to Hi-Z when deasserted
- Topology: stacked PMOS pair (VDD → ne̅ → in) and stacked NMOS pair (in → ne → GND)
- Symbol exposes pins: `x` (in), `ne` (enable), `y` (out)
- Layout, DRC/LVS, QRC extraction and post-layout sim completed
- Verified three-state behaviour: output follows `¬x` when `ne=0`; Hi-Z when `ne=1`

---

### 3. D Flip-Flop (Master–Slave)

Built hierarchically from the inverter and tristate inverter cells above.

**Circuit composition:**
- 4 × tristate inverter instances
- 5 × inverter instances
- Master latch: tristate inverter (CKN-gated) + inverter feedback loop
- Slave latch: tristate inverter (CK-gated) + inverter feedback loop
- Outputs: `Q`, `nQ`

**Ports:** `D`, `CLK` → `Q`, `nQ`

- Schematic assembled hierarchically in Virtuoso Schematic Editor
- Layout completed in Virtuoso Layout Suite GXL
- Post-layout transient simulation confirmed correct D-FF sampling behaviour on rising clock edge

---

### 4. Fanout Buffer Chain

- 1-to-N inverter fanout schematic built in Virtuoso Schematic Editor
- Transient simulation used to observe signal integrity degradation with increasing fanout
- Both NMOS discharge current (`V0/MINUS`) and PMOS charge current (`V0/PLUS`) probed

---

### 5. Wire Delay (Interconnect Analysis)

- Short-length and long-length wire delay schematics constructed
- Two inverters separated by a distributed RC wire segment
- Parametric sweep across supply voltage (`Vdd`) with delay measurement (`delayMeasure`)
- Energy-delay product plotted to identify optimal operating point

---

## Verification Summary

| Cell | DRC | LVS | QRC Extraction | Post-Layout Sim |
|------|-----|-----|----------------|-----------------|
| Inverter | ✅ Clean | ✅ Clean | ✅ av_extracted | ✅ Verified |
| Tristate Inverter | ✅ Clean | ✅ Clean | ✅ av_extracted | ✅ Verified |
| D Flip-Flop | ✅ Clean | ✅ Clean | — | ✅ Verified |
| Fanout Chain | — | — | — | ✅ Verified |
| Wire Delay | — | — | — | ✅ Verified |

---

## Repository Structure

```
.
├── inverter/
│   ├── schematic/
│   ├── symbol/
│   ├── layout/
│   └── av_extracted/
├── tristate_inverter/
│   ├── schematic/
│   ├── symbol/
│   ├── layout/
│   └── av_extracted/
├── d_flip_flop/
│   ├── schematic/
│   └── layout/
├── fanout/
│   └── schematic/
├── wire_delay/
│   └── schematic/
└── README.md
```

> **Note:** Cadence Virtuoso library files (`.cdsinit`, `cds.lib`, cell directories) should be placed inside each cell folder. PDK files are not included — UMC 90nm SP PDK must be licensed separately.

---

## Tools & Environment

| Tool | Version / Node |
|------|---------------|
| Cadence Virtuoso | 6.1.8 |
| Assura (DRC/LVS) | Included with Virtuoso |
| QRC Extractor | For `av_extracted` parasitic netlists |
| ADE (Analog Design Environment) | For transient / parametric simulation |
| Process Node | UMC 90nm SP |

---

## Key Simulation Results

- Inverter: correct logic inversion confirmed in transient response
- Tristate inverter: Hi-Z state observed when enable deasserted; logic inversion when enabled
- D flip-flop: Q samples D on active clock edge; nQ follows complement
- Wire delay: delay increases with wire length; energy-delay trade-off confirmed with supply sweep
- Fanout: signal quality degrades as fanout increases; buffering strategy evaluated
