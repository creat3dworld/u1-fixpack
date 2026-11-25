# Re-verification of PR #869 on the 2.1.2 develop build

Date: 2026-11-25. Status: batch-mode re-verification complete. Live-U1 test still owed.

## What was tested

PR #869 ("Resume after a skipped object", u1-klipper#22) against the Paxx12
Extended Firmware 2.1.2 develop build, in Klipper batch mode with the shipped
`printer.cfg` and `fluidd.cfg`, using `tests/test_issue22.py` (4 cases).

## Versions

- u1-klipper: `7ac453b50fe6779b2ec0b759038a17da5e6d671d` ("Release firmware
  version 2.1.0", 2026-11-03). This is the newest commit in
  Snapmaker/u1-klipper — no newer commit exists for stock 2.1.2, so the
  2.1.2 build's Klipper source is still 7ac453b.
- Paxx develop HEAD: `8d8af0d5fd34be52dff22942e6730a7bcf1751d0`
  (2026-11-21, "Install HelixScreen from the release `.zip`... (#915)") —
  the rolling 2.1.2 build (stock base 2.1.2.159).
- Extended Klipper overlay patches 01–08 at 8d8af0d are byte-identical to
  those at 601e196 (2026-11-04). The overlay did not change.
- PR #869 head: `6e84c9306c316cf89984effd1bd23c7c2138af8a`; patch file
  `10_exclude_object_manual_move.patch` (3,113 bytes). Its code hunks are
  identical to `patches/0022-exclude-object-manual-move.patch` in this repo
  (verified by comparing all added/removed lines).

## Method

Native (no Docker) run of the existing rig: `/opt/klipper` at 7ac453b,
dicts built from `lava/` Kconfigs with ARM GCC 13.3.1
(`at32f403a_config`, `at32f415_config`, `klippy_mcu_config`), `c_helper.so`
built from the same tree, Python 3.12 venv with cffi 1.17.1 / Jinja2 3.1.4.

- Baseline: `FIXPACK_PATCHES` = ext 01–08 only (the 2.1.2 build as shipped).
- Patched: ext 01–08 + patch 10.

## Results

| Case | Baseline (2.1.2 as built) | + patch 10 |
|---|---|---|
| pause then resume | FAIL — `Move out of range on Y axis: 33.000 448.500 5.800` | PASS |
| end-of-print park + E-only move | FAIL — `Move out of range on Y axis: 35.000 450.500 1.200` | PASS |
| park inside skipped object | FAIL — gcode pos drifts from toolhead (after park: toolhead X45.000 Y300.500 vs gcode X33.000 Y448.500; at end: toolhead X127.000 Y32.000 vs gcode X117.000 Y182.000) | PASS (toolhead stays X45.000 Y300.500) |
| tool change after skip | FAIL — prime lands 10 mm off in X (gcode X82.700, head X92.700) | PASS |

Baseline: 4/4 fail (unittest exit 1). Patched: 4/4 pass (unittest exit 0).
Per-case `responses.txt`, `klippy.log`, `trace.txt` are in `baseline/` and
`patched/` next to this file.

## Gotcha found while testing

The PR patch file as stored in the overlay is silently skipped by
`git apply` outside a git repo (exit 0, nothing applied — `Skipped patch
'klippy/extras/exclude_object.py'`). The build itself applies overlays with
`patch -F 0`, which applies it fine, and the same code as a plain git patch
applies fine too. The first patched run in this session hit exactly this
trap (4/4 fail with the patch "applied"); the re-run used the identical-code
local patch file and went 4/4 pass, with the patch confirmed present in the
tested tree (`last_tx_position` × 5 in `exclude_object.py`).

## Still owed

A run on the physical U1 once it is on 2.1.2: skip an object, pause, resume,
as in Snapmaker/u1-klipper#22. The draft comment for PR #869 is held with
this verification (not posted).
