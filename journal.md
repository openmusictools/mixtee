# MIXTEE Development Journal

## 2026-03-01 — Daughter/Output Board Trace Routing

### What was done
Added programmatic trace routing to `hardware/kicad/gen_daughter_output.py`. The board has 6 nets (AIN1–4, +5VA, GND) across 10 components on an 80×20mm 2-layer PCB.

### Routing strategy
- **AIN1**: All F.Cu — L-route from J1 to J2.T, vertical down to D1.K
- **AIN2–4**: F.Cu stub from J1 → via → B.Cu hop → via → F.Cu fan-out to jack T pad and diode K pad
- **+5VA**: F.Cu L-route from J1 to C1
- **GND**: Short F.Cu stubs from SMD diode/cap pads to vias connecting to B.Cu ground zone

Helper functions added: `trace_seg()` and `via_pad()` for generating KiCad S-expression segments and vias.

### Bugs found and fixed

**1. J1 rotation convention (rotation=90 → 270)**

KiCad uses **clockwise** rotation for positive angles in PCB coordinates. With `rotation=90`, J1's pins were mirrored — pin 1 (AIN1) ended up at y=115 (bottom) instead of y=105 (top). Every trace shorted to the wrong J1 pin.

Fix: changed J1 rotation from 90° to 270°. With 270°, the pin-to-absolute transform is `(px, py) → (-py, px)` giving pin 1 at (104, 105) as intended.

**2. Jack S pads extending beyond board edge (jack_y 104 → 106)**

With jacks at y=104, the top S pads (at relative `(0, -3)`) had centers at y=101 — their 3mm diameter circles extended 0.5mm above the board edge (y=100). DRC flagged board-edge clearance violations.

Fix: moved jacks from `jack_y = oy + 4.0` to `oy + 6.0`. Top S pad centers now at y=103, fully inside the board with 1.5mm margin. All trace coordinates updated to match.

### Current state
- Signal traces and vias: all 6 nets routed, 0 unconnected after zone fill
- Remaining DRC items: 2 shorting violations (AIN1 trace at y=106 passes through J2.S pad at (108.92, 106); AIN2 B.Cu hop at y=107 passes near J3.S pad) — need to reroute AIN1 horizontal and AIN2 B.Cu to avoid jack S pad areas
- GND vias flagged as "dangling" because `kicad-cli` DRC doesn't fill zones — these resolve when zones are filled in KiCad GUI
- lib_footprint_mismatch warnings are expected (our simplified footprints vs full KiCad library)

### Next steps
- ~~Fix AIN1 horizontal trace to avoid J2.S pad~~ → resolved via FreeRouting (see 2026-03-02)
- ~~Fix AIN2 B.Cu hop to avoid J3.S pad~~ → resolved via FreeRouting
- ~~Verify clean DRC (0 errors) after fixes~~ → done
- Consider adding silkscreen labels and courtyard outlines to custom footprints

---

## 2026-03-02 — Daughter/Output Board: FreeRouting Autoroute + Gerber Export

### What was done
Scrapped the manual trace routing from gen_pcb.py (which had 2 shorting errors at jack S pads) and replaced it with a FreeRouting autoroute via Specctra DSN/SES round-trip. Fixed silkscreen clipping DRC warnings in the process. Board is now fully routed and Gerbers are exported.

### Pipeline used

1. **Silkscreen fix** — Custom `Switchcraft_112BPC` footprint silk rectangle extended past all 4 board edges when jacks were placed at y=20 with 90° rotation. Clipped from `(-4, -6.35)→(22, 10)` to `(0.5, -6.35)→(19.5, 9.5)` in both the `.kicad_mod` library file and all 4 inline instances in the `.kicad_pcb`. Also moved board title text from y=22 to y=18.5.

2. **DSN export** — Used KiCad's pcbnew Python API (`pcbnew.ExportSpecctraDSN()`) since `kicad-cli` doesn't support DSN export. Patched the DSN to split the single `kicad_default` net class into three classes matching the project settings:
   - `Audio_Analog` (AIN1-4): 0.3mm width, 0.25mm clearance
   - `Power` (+5VA): 0.5mm width
   - `kicad_default` (GND): 0.25mm width

3. **FreeRouting** — Loaded DSN, ran autoroute, exported SES. FreeRouting produced 44 trace segments using both F.Cu and B.Cu with vias, correctly respecting net class widths.

4. **SES import** — `pcbnew.ImportSpecctraSES()` to bring routes back into the PCB.

5. **Post-routing validation** — Opened project via kicadmixelpixx MCP (`open_project`), refilled GND zone (`refill_zones`), saved (`save_project`), verified components (`get_component_list`), queried traces (`query_traces`). DRC via kicad-cli: **0 errors, 0 unconnected, 26 cosmetic warnings**.

6. **Gerber export** — `kicad-cli pcb export gerbers` + `pcb export drill`. 7 Gerber layers + drill + job file in `gerbers/`.

### Tooling notes

- **kicadmixelpixx MCP (SWIG backend)**: works well for `open_project`, `get_component_list`, `refill_zones`, `save_project`, `query_traces`. DRC and Gerber export tools wrap `kicad-cli` and fail if it's not on PATH. `get_board_2d_view` needs Cairo DLL.
- **kicad-cli** (at `D:\programs\KiCad\9.0\bin\kicad-cli.exe`): not on PATH by default. Used with full path for DRC and Gerber export. No DSN export subcommand in KiCad 9.
- **pcbnew Python** (at `D:\programs\KiCad\9.0\bin\python.exe`): has `ExportSpecctraDSN()` and `ImportSpecctraSES()` — the only scriptable path for Specctra round-trip.

### DRC breakdown (post-route)

| Type | Count | Notes |
|------|-------|-------|
| silk_overlap | 10 | Adjacent jack silkscreens + board title text |
| lib_footprint_mismatch | 6 | Generated footprints vs KiCad library |
| silk_edge_clearance | 4 | J2/J3 ref text at board edge, J5 silk past top |
| lib_footprint_issues | 4 | Custom lib path resolution in kicad-cli |
| silk_over_copper | 2 | Silk segments over exposed pads |
| **Errors** | **0** | |
| **Unconnected** | **0** | |

### Files produced
- `hardware/kicad/mixtee-daughter-output/mixtee-daughter-output.dsn` — Specctra DSN with net classes
- `hardware/kicad/mixtee-daughter-output/mixtee-daughter-output.ses` — FreeRouting session
- `hardware/kicad/mixtee-daughter-output/gerbers/` — 7 Gerber + drill + job (9 files)
