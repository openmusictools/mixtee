# 025 — PHONEE extracted to standalone repo

**Date:** 2026-03-21
**Scope:** `d:\projects\openaudiotools\phonee\` (new repo), `hardware/pcbs/hp/README.md` (canonical-source note)

## Summary

Extracted PHONEE headphone output module into its own standalone repository at `openaudiotools/phonee`, following the same pattern as DESPEE. The standalone repo contains device-agnostic documentation (architecture, connections, integration guide with grounding/isolation strategies). MIXTEE's `hardware/pcbs/hp/` retains device-specific integration context with a canonical-source note pointing to the standalone repo.

## What was created

| File | Content |
|------|---------|
| `phonee/README.md` | Project overview, specs, links |
| `phonee/CLAUDE.md` | Agent guidance (passive analog module, no firmware) |
| `phonee/LICENSE` | Triple license (CERN-OHL-P v2, CC BY 4.0, MIT) |
| `phonee/docs/architecture.md` | TPA6132A2 circuit, signal path, BOM, power, PCB notes — device-agnostic |
| `phonee/docs/connections.md` | Input connector, TRS jack, pot pinouts — generic signal names (Audio L/R, VCC, GND) |
| `phonee/docs/integration-guide.md` | Power options, grounding strategies (shared/isolated/star), detect routing (GPIO/I2C/ignore), input requirements, mechanical integration, MIXTEE example |
| `phonee/hardware/bom.csv` | PHONEE-only BOM (~$5.60 total) |

## What changed in MIXTEE

| File | Change |
|------|--------|
| `hardware/pcbs/hp/README.md` | Added canonical-source note pointing to openaudiotools/phonee |

## Key design decisions

- **Device-agnostic docs:** PHONEE docs use generic names (Audio L/R, VCC, GND) — no MIXTEE-specific terms (Master L/R, 5V_ISO, GND_ISO, Board 1-top)
- **Integration guide** covers three grounding strategies (shared, isolated, star) and three detect routing options (direct GPIO, I2C expander, ignore)
- **MIXTEE as example:** The integration guide includes a concrete example of how MIXTEE integrates PHONEE (isolated domain, MCP23008 detect)
- **Canonical-source pattern:** Same as DESPEE — design changes go to phonee repo first, MIXTEE syncs
