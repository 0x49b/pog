# Next-Generation Keyboard Configurator

## Requirements, competitive analysis, and architecture study

This folder is a from-scratch product and engineering study for a successor to **Pog**: a desktop (and later mobile/web) configurator for programmable keyboards.

It is written as if a requirements engineer and a software engineer sat down together after:

1. Inventorying every feature, protocol, and firmware file in this repository (`pog` v2.2.3).
2. Comparing that surface to the tools people actually use: **VIA, Vial, QMK Configurator, Remap, ZMK Studio, Chrysalis, QMK Toolbox, Keyboard Layout Editor, and KMK itself**.
3. Evaluating two implementation stacks the product owner nominated: **Kotlin + Compose Multiplatform** (with optional Kotlin/Native) and **Wails 3**.

The study does **not** implement the new app. It defines what to build, why, and how.

---

## How to read this

| If you need… | Read |
|---|---|
| The decision and the bet | [01 — Executive summary](01-executive-summary.md) |
| What Pog already is, including gaps | [02 — Pog as-is analysis](02-pog-as-is.md) |
| What every competing tool actually offers | [03 — Competitive landscape](03-competitive-landscape.md) |
| QMK / KMK / ZMK / CircuitPython as platforms | [04 — Firmware ecosystem](04-firmware-ecosystem.md) |
| The full feature list with priorities | [05 — Feature catalog](05-feature-catalog.md) |
| Formal functional and non-functional requirements | [06 — Requirements](06-requirements.md) |
| Canonical data model and interchange | [07 — Domain model](07-domain-model.md) |
| System architecture independent of UI stack | [08 — Architecture](08-architecture.md) |
| Compose Multiplatform vs Wails 3, with a recommendation | [09 — Stack comparison](09-stack-compose-vs-wails.md) |
| Serial, HID, USB MSC, BLE, flashing | [10 — Protocols and hardware](10-protocols-and-hardware.md) |
| UX, journeys, and features that would stand out | [11 — UX and differentiators](11-ux-and-differentiators.md) |
| Phased delivery | [12 — Roadmap](12-roadmap.md) |

Read **01 → 05 → 09 → 12** if you only have time for the spine. The rest is evidence and specification.

---

## Working names used in this study

| Term | Meaning |
|---|---|
| **Pog** | The existing Electron + Vue 3 + KMK/CircuitPython app in this repo |
| **Successor** | The new product to be built from scratch |
| **Board definition** | Hardware + layout + wiring metadata (what VIA calls a definition, Vial embeds in firmware, Pog stores in `pog.json`) |
| **Keymap** | Layered assignment of behaviors to physical positions |
| **Behavior** | Anything a key can do: a HID code, a layer switch, a hold-tap, a combo result, a macro |
| **Live protocol** | Runtime read/write of keymap/settings without rewriting firmware files |
| **Generation protocol** | Writing Python/C/devicetree onto a board or producing a firmware image |
| **Bare MCU path** | Starting from a blank CircuitPython (or UF2) controller and ending with a working keyboard — Pog’s unique job |
| **Catalog path** | Starting from a known keyboard that already has firmware — VIA/Vial/Remap’s job |

Pog is almost entirely a **bare MCU + generation** tool. The market-leading tools are almost entirely **catalog + live protocol** tools. The successor must do both, or it will lose on day-to-day remapping *and* fail to keep Pog’s DIY reason to exist.

---

## Scope of research (sources)

Primary in-repo sources: `src/main/*`, `src/renderer/src/{store,screens,components,helpers}/*`, `src/main/pythontemplates/*`, `src/preload/index.ts`, `docs/toggle-drive-guide.md`, `README.md`, `package.json`, `electron-builder.yml`.

External sources consulted in 2026:

- VIA: usevia.app, the-via/app, VIA user guides
- Vial: get.vial.today, getreuer.info Vial guide (updated April 2026), vial-qmk
- QMK: docs.qmk.fm (layers, combos, tap dance, config options)
- KMK: kmkfw.io / KMKfw/kmk_firmware docs (modules, extensions, split, holdtap, encoder)
- ZMK Studio: zmk.dev/docs/features/studio
- Remap: remap-keys.app
- Chrysalis: keyboardio/Chrysalis
- QMK Toolbox: qmk/qmk_toolbox
- Keyboard Layout Editor / Keyboard Layout Studio
- Wails v3 beta docs (v3.wails.io)
- Compose Multiplatform packaging and native-access docs
- jSerialComm, go.bug.st/serial, hidapi-class libraries

---

## Document status

This is a design baseline, not a frozen spec. Treat numbered requirements in `06` as the contract. Treat `05` as the backlog. Treat `09` as a recommendation, not a mandate — the architecture in `08` is stack-agnostic on purpose.
