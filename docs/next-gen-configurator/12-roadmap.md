# 12 — Implementation roadmap

Phased so the unique workshop and the Vial-class live path come up together, not “pretty keymap then maybe flashing.”

Estimates are **complexity**, not calendar time: **S** small, **M** medium, **L** large, **XL** extra.

---

## Phase 0 — Decisions and spike

**Exit:** stack chosen, spike passes `09` §8, licenses written.

| Work | Size | Notes |
|---|---|---|
| Decide stack (Wails 3 default) | S | Written confirmation |
| Decide v1 firmware (KMK only) | S | |
| Repo layout, CI, packaging hello-world | M | Win/mac/Linux artifact |
| Enumerate MSC + serial, group by USB serial | M | |
| Exclusive CDC open + quit without hang | M | Automated test |
| Atomic write on a FAT volume | S | |
| Pog 2.x `pog.json` → domain fixture tests | M | All documented fields |
| MCU catalog seed (Pico, Helios, 8 more) | M | |

**Do not** start a keymap pixel-clone in this phase.

---

## Phase 1 — Domain + Pog import + file generate

**Exit:** can open a Pog board from disk, edit name/layers as data, write `board.json`/`keymap.json` + stub runtime to CIRCUITPY, board still types if old `kb.py` remains (compat emit).

| Work | Size |
|---|---|
| Domain types + validation | L |
| Pog importer + extensions bag | L |
| Pog-compat exporter | M |
| FilePlan + FlashService (no KMK download yet) | M |
| Library on disk + backups | M |
| KMK behavior parser/printer (golden tests) | L |
| Minimal UI: library + JSON-ish editors acceptable | M |

Runtime can still be Pog templates here so hardware users are not blocked.

---

## Phase 2 — Workshop runtime v0 + Serial v2

**Exit:** new boards boot `workshop.main()`, no `eval`, capabilities over serial, SET_KEY works, drive hide works.

| Work | Size |
|---|---|
| Python runtime package (load JSON, build KMK) | XL |
| Serial v2 + Pog v1 detect | L |
| CAPS, GET/SET_KEY, GET/SET_KEYMAP, DRIVE, RESET | L |
| Unlock combo | M |
| Hash verify KMK download | S |
| Install/update runtime without wiping overlays | M |
| Console (logs only) | S |
| Diff before write | M |

This is the **new firmware product**. Treat it like one: version, changelog, hardware test on one Pico.

---

## Phase 3 — Workshop UX (Pog-complete)

**Exit:** AS-1 without HID (serial + MSC). Automatic detect, coord map, layout generate, pin validate, split fields, encoders, RGB, KMK copy.

| Work | Size |
|---|---|
| UF2 CircuitPython drop | M |
| Detection mode in runtime (not forever-`code.py`) | L |
| Detect UI + undo + skip pins | L |
| Coord-map visual | M |
| Layout editor (KLE import, undo, generate ortho) | XL |
| Pinout click + conflicts | L |
| Wiring / split / encoder / RGB forms | L |
| Dual-half sequencing | M |
| Recovery / safe mode copy | M |

At the end we are **Pog-complete** and already better on protocol and files.

---

## Phase 4 — Live UX (Vial-complete KMK)

**Exit:** the v1 tagline in `05` §T is true. HID preferred, serial fallback.

| Work | Size |
|---|---|
| HID descriptor + same opcodes | L |
| Visual keymap + catalog + nested HT/LT/MT | XL |
| Macros tap/down/up/delay/text + recorder | L |
| Combos GUI | L |
| Tap dance GUI | L |
| Settings (tap/combo/oneshot/autoshift) | M |
| Encoder click + per-layer map | M |
| RGB live | S |
| Key tester | M |
| Host-locale legends (US+DE) | M |
| HRM wizard | S |
| WebHID spike (optional companion) | M |

---

## Phase 5 — Hardening and 1.0

| Work | Size |
|---|---|
| `.kbpack` | M |
| QMK info.json / keymap.json export | M |
| Firmware diff UX | M |
| Diagnostics zip | S |
| i18n EN+DE app strings | M |
| Auto-update | M |
| Linux udev / first-run | S |
| Accessibility pass | M |
| Docs for builders and vendors | M |
| Import VIA definition (layout only) | M |
| Load test: 80-key split, 8 layers, 20 combos | S |

**1.0 ships** when P0 catalog items are done or waived, AS-1…AS-5 pass, and packages are signed.

---

## Phase 6+ — After 1.0 (do not schedule into v1)

| Item | Phase-ish |
|---|---|
| Typing practice | 1.1 |
| OLED / backlight / power | 1.1 |
| Key overrides if we implement in KMK | 1.2 |
| Unicode helper | 1.1 |
| VIA/Vial HID **client** (configure QMK boards) | 2.0 |
| ZMK `.keymap` export | 2.0 |
| BLE configure | 2.1 |
| Android companion (if CMP or WebHID) | 2.1 |
| Cloud catalog | 3.0 if ever |
| QMK compile in-app | probably never (link to qmk / Remap workbench) |

---

## Parallel workstreams (if two+ engineers)

```
        Phase 0 spike
         /        \
   Runtime+proto   Domain+import
         \        /
          Phase 2
         /        \
   Workshop UI     Live UI
         \        /
          Phase 5
```

Never let Live UI invent keycode strings the runtime cannot parse. IR is the contract; both sides consume tests.

---

## Suggested first-year staffing (roles, not headcount)

| Role | Owns |
|---|---|
| Host engineer | Transports, state machine, packaging |
| Firmware engineer | CircuitPython runtime, KMK integration, HID |
| Product/UI engineer | Workshop + Live editors |
| (part-time) | MCU catalog, docs, fixtures |

One person can do this in a long slog; the risk is skipping tests and HID.

---

## Definition of done (every feature)

1. Domain test or protocol golden
2. Capability bit if device-facing
3. FilePlan role if it writes
4. Unlock behavior documented
5. Pog import path if a Pog field exists
6. UI hidden when capability missing

---

## Kill criteria (stop and re-scope)

- HID + serial both fail on a stock Pico + CP in the spike
- Runtime cannot apply SET_KEY without full reload that drops keys for >1 s (find another approach before building the GUI)
- Team starts adding QMK compile “just for one vendor”
- Web3, accounts, or cloud compile appear in P0

---

## Immediate next actions after this study

1. Owner answers the six questions in `01` §Immediate decisions.
2. Create a new repository (do not grow this Electron app).
3. Run Phase 0 spike on the chosen stack.
4. Freeze `board.json` / protocol 2 drafts from `07` and `10` as ADRs.
5. Port Pog fixtures into the new repo as the import test corpus.
6. Leave this repo as the compatibility reference and, if desired, a `pog.json` exporter once the successor can emit 2.x.

Rewriting in place (this Electron tree) will drag `keyboardStore`, WalletConnect, and `process.exit` along. **From scratch means from scratch.** This folder is the spec, not a refactor plan for `src/`.
