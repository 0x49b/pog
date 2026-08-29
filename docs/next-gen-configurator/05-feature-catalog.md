# 05 — Feature catalog

This is the backlog. Every item is a product requirement, not a brainstorm.

**Priority**

| Tag | Meaning |
|---|---|
| **P0** | v1. Must ship or we are behind Pog *or* we cannot call it a configurator |
| **P1** | v1.1. Vial/Remap table stakes or Pog wishlist that users already hit |
| **P2** | Differentiator or second firmware |
| **P3** | Later / niche |

**Compliance**

- `POG` — exists in Pog today (preserve or replace with a strict upgrade)
- `VIA` `VIAL` `REMAP` `QMKC` `ZMK` `KLE` `CHRY` — required to claim parity with that tool
- `NEW` — we invent it

**Stack notes** mark hardware or OS difficulty.

---

## A. Product foundation

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| A1 | Offline-first desktop app (Win x64, macOS arm64+x64, Linux x64 AppImage/deb) | P0 | POG VIA VIAL | Codesign + notarize on macOS |
| A2 | Auto-update with signed artifacts | P1 | POG (partial) | Pog has electron-updater, mostly off |
| A3 | Dark/light theme, density, high-contrast | P1 | NEW | DaisyUI themes are not accessibility |
| A4 | i18n of the *app* (EN first, DE next — author is DE) | P1 | NEW | Separate from host-locale *key legends* |
| A5 | Crash log + diagnostics bundle (OS, ports, last protocol error) | P0 | NEW | Serial/drive bugs are otherwise unsupportable |
| A6 | First-run hardware permission guide (macOS input/USB, Linux udev/dialout) | P0 | VIAL ZMK | Vial/ZMK already lose users here |
| A7 | About / licenses / firmware attribution (KMK SHA, CP version) | P0 | POG | Legal + support |
| A8 | CLI: `workshop devices`, `workshop push`, `workshop import` | P2 | NEW | For CI and power users |
| A9 | Headless / CI mode for generating files | P2 | NEW | Kit vendors |

---

## B. Device discovery and connection

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| B1 | List USB mass-storage candidates (CIRCUITPY, RPI-RP2, NICENANO, custom labels) | P0 | POG | Not only “removable USB” — include labeled volumes |
| B2 | List serial/CDC ports with USB VID/PID, manufacturer, product, serial | P0 | POG | Pog already lists; pairing is weak |
| B3 | Pair ports + drives of the same USB device by serial number | P0 | POG wishlist | Two CDC + one MSC is the CircuitPython default |
| B4 | Filter/identify first-party runtime by USB ID *and* capability ping | P0 | POG | Do not trust manufacturer string alone |
| B5 | Watch plug/unplug and update UI without full rescan button | P0 | VIA VIAL | Pog is poll/manual refresh |
| B6 | Connection state machine: Idle, Enumerating, Drive, Live, Detecting, Flashing, Error, Quitting | P0 | NEW | Fixes Pog quit hang |
| B7 | Multi-keyboard connected at once; picker | P1 | VIA VIAL | |
| B8 | Split as one logical device (left+right) | P0 | POG ZMK | See F-split |
| B9 | HID device enumeration (VIA/Vial usage page) | P2 | VIA VIAL REMAP | Backend Q1 |
| B10 | BLE central scan + connect (configure over wireless) | P2 | ZMK | Linux/mac first; Windows harder |
| B11 | Remember last transport per Board id | P0 | POG | |
| B12 | “Drive hidden” is a normal state, not an error | P0 | POG | Live protocol required |

---

## C. Library, history, projects

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| C1 | Local library of boards (not only localStorage) | P0 | POG upgrade | App data dir, one folder per board |
| C2 | Autosave + numbered backups (keep N) | P0 | POG | Pog: 100 in localStorage |
| C3 | Open folder / reveal on disk | P1 | POG | |
| C4 | Import existing `pog.json` 100% | P0 | POG | Contract |
| C5 | Import KLE, QMK info.json/keyboard.json, VIA def, Vial .vil, keymap.json | P1 | POG KLE QMKC VIA VIAL | Staged |
| C6 | Export `.kbpack` (zip: board, keymap, layout, images, hashes) | P1 | POG wishlist | Offline share |
| C7 | Export KLE, QMK info.json, VIA definition, `.vil` | P1 | KLE QMKC VIA VIAL | Even before QMK backend |
| C8 | Duplicate board, “save as” | P1 | POG | |
| C9 | Tags, search, sort (recent, name, connected) | P1 | REMAP | |
| C10 | Backup/restore keymap only vs whole board | P0 | POG VIA VIAL | Pog firmware screen has restore |
| C11 | Diff two backups / two files | P1 | REMAP | Remap “compare changes” |
| C12 | Cloud catalog / login | P3 | REMAP | Not v1 |

---

## D. Workshop: bare MCU onboarding

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| D1 | Detect UF2 bootloader drives and flash CircuitPython UF2 from cache | P0 | NEW / POG gap | Pog only links the website |
| D2 | MCU catalog with pinout art, aliases, forbidden pins, ADC, USB, LED, debug | P0 | POG stub | Pico, Helios, and 20+ common RP2040 boards |
| D3 | Guided “do you have diodes? direct wiring? split?” interview | P0 | POG wizard | |
| D4 | Flash detection firmware / enter detection mode | P0 | POG | Prefer mode in runtime, not overwrite `code.py` forever |
| D5 | Matrix autodetection by keypress | P0 | POG | Show live matrix, debounce, undo last key |
| D6 | Direct-pin autodetection | P1 | POG | Same detector, different interpretation |
| D7 | Diode-direction test | P1 | NEW | Infer COL2ROW vs ROW2COL |
| D8 | Skip / edit detected pins; lock a pin as unused | P0 | POG gap | Power pins must be skippable |
| D9 | Coord-map capture as visual “press in reading order” | P0 | POG | Replace number-row macros as the only UX |
| D10 | Generate ortho/staggered layout from matrix + optional KLE | P0 | POG wishlist | |
| D11 | Install pinned KMK + workshop runtime with hash verify | P0 | POG | HTTPS + SHA256 |
| D12 | Write `board.json` + stub `code.py` without clobbering overlays | P0 | POG upgrade | |
| D13 | Progress UI per file + cancellable copy | P0 | POG | |
| D14 | Post-setup “type here to verify” | P0 | VIA tester | |
| D15 | Recovery: restore last known-good `code.py` / `board.json` | P0 | NEW | Safe mode |
| D16 | QR / photo attach of wiring | P2 | POG wishlist | Into `.kbpack` |

---

## E. Physical layout editor

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| E1 | KLE-compatible geometry (x,y,w,h,x2,y2,w2,h2,r,rx,ry) | P0 | POG KLE | |
| E2 | Add, delete, duplicate, multi-select, nudge, snap to 0.25u | P0 | POG KLE | |
| E3 | Undo/redo | P0 | KLE | Pog is weak here |
| E4 | Align, distribute, pack to grid | P1 | KLE | |
| E5 | ISO enter, stepped caps, decals | P1 | KLE | |
| E6 | 9-position legends for documentation (not firmware) | P2 | KLE | |
| E7 | Bind key → matrix (r,c) or direct index or encoder | P0 | POG | Visual, not only fields |
| E8 | Layout variants (ANSI/ISO, blockers, split space) | P0 | POG VIA ZMK | |
| E9 | Import KLE / QMK layouts | P0 | POG | |
| E10 | Export KLE / screenshot / SVG | P1 | KLE | |
| E11 | Case/plate overlay image | P2 | NEW | |
| E12 | Generate layout from matrix dimensions | P0 | POG wishlist | |
| E13 | Mirror (for split right half) | P1 | ZMK | |
| E14 | Keyboard-as-input to select the key you pressed | P1 | VIA tester | Huge for assignment |

---

## F. Wiring, pins, split, hardware

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| F1 | Matrix rows/cols + pin lists | P0 | POG | |
| F2 | Direct pin lists | P0 | POG | |
| F3 | Diode direction | P0 | POG | |
| F4 | Pin prefix modes (gp/board/none/quickpin) **or** structured Pin ids | P0 | POG | Prefer structured + display formatter |
| F5 | Click pins on MCU drawing | P0 | NEW | |
| F6 | Conflict detection (same pin twice, reserved pins, I2C vs UART) | P0 | POG wishlist | |
| F7 | Wiring preview (which physical key is row3/col4) | P1 | POG wishlist | |
| F8 | Split types: none, UART, UART-2wire, onewire, BLE | P0 | POG KMK | |
| F9 | Split side: fixed L/R, VBUS, label, EE-hands | P0 | POG | |
| F10 | Split pins, PIO, flip, uart_flip, target_left | P0 | POG | |
| F11 | Dual-half workflow (flash both, detect both, one keymap) | P0 | POG gap | |
| F12 | Encoders: pads, optional switch pin, invert | P0 | POG VIAL | Click is Pog wishlist |
| F13 | Encoder-as-scanner when both halves have encoders | P1 | KMK | |
| F14 | RGB pin + count + map to keys (optional) | P0 | POG VIAL | Map is P1 |
| F15 | Mono backlight pin + count | P1 | POG hidden | |
| F16 | OLED present + size + I2C pins | P2 | KMK | |
| F17 | Pointing device (trackball/cirque) | P3 | KMK QMK | |
| F18 | Battery / power pins | P2 | KMK ZMK | |
| F19 | Controller picker with community boards | P1 | POG | |

---

## G. Keymap editor (the daily driver)

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| G1 | Visual keymap on the real layout | P0 | ALL | |
| G2 | Multi-select assign | P0 | POG | |
| G3 | Auto-advance to next key | P0 | POG | |
| G4 | Searchable keycode/behavior catalog | P0 | VIA VIAL | Categories + search + aliases |
| G5 | Host-locale legends (US, DE, UK, Nordic, JIS, …) | P1 | POG wishlist ZMK | Labels only; HID unchanged |
| G6 | Raw/Any behavior with validation | P0 | POG VIAL | |
| G7 | Transparent / None | P0 | ALL | |
| G8 | Basic HID: letters, mods, nav, numpad, ISO, F13–24 | P0 | ALL | |
| G9 | Media, app, consumer | P0 | ALL | |
| G10 | Mouse keys | P0 | POG VIAL | |
| G11 | Layer keys: MO TG TO TT LM LT DF (+ PDF if runtime has it) | P0 | POG VIAL | LT is missing in Pog picker |
| G12 | Nested builder: pick LT → pick layer → pick tap key | P0 | VIAL REMAP | |
| G13 | Hold-tap / mod-tap builder with prefer_hold, tap_interrupted, per-key tap_time | P0 | VIAL KMK | Home-row mods |
| G14 | One-shot mods and one-shot layers | P0 | VIAL | |
| G15 | RGB / backlight keycodes | P0 | POG VIA | |
| G16 | Workshop custom keys (DFU, Safe, ToggleDrive, user overlays) | P0 | POG | |
| G17 | MIDI keycodes | P2 | VIAL KMK | |
| G18 | Keyboard picker layouts (QWERTY/Colemak/DH/Dvorak) as *input method* | P1 | POG | Keep |
| G19 | Ghost/disabled keys for variants | P1 | POG VIA | |
| G20 | Copy key, copy selection, paste | P0 | NEW | |
| G21 | Layer-colored keycaps, reduce-color mode | P0 | POG | |

---

## H. Layers

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| H1 | Add, delete, reorder, duplicate layers | P0 | POG VIA | |
| H2 | Name + color | P0 | POG ZMK | |
| H3 | No artificial 4-layer cap | P0 | POG | Warn on tiny MCU only |
| H4 | Default layer | P1 | VIA CHRY | |
| H5 | Copy layer to another | P1 | CHRY | |
| H6 | Conditional / combo layers (KMK combo_layers) | P2 | KMK | |
| H7 | Reserved extra layers for live enable (ZMK style) | P2 | ZMK | |
| H8 | Per-layer encoder map and RGB hint | P1 | VIAL | |

---

## I. Advanced behaviors

### I1 Macros — P0 — VIA VIAL REMAP POG-upgrade

- List of named macros
- Actions: Tap, Down, Up, Delay(ms), Text (UTF-8 via OS-specific unicode sequence *or* KMK string), Wait-for-release
- Recorder (listen to HID or to the board)
- Assign macro to a key
- Show compiled size / count vs firmware limit
- Import/export
- Pog’s 2–4 key simultaneous Press/Release remains a *preset*, not the whole editor

### I2 Combos — P0 — VIAL KMK

- Chord vs sequence
- Select 2+ keys on the layout (or by keycode)
- Result = any behavior (key, layer, macro)
- timeout, per_key_timeout, fast_reset, match_coord
- Enable/disable per layer (if runtime allows)
- Visual: highlight combo members

### I3 Tap dance — P0 — VIAL KMK

- N taps + optional hold
- Each slot is a full behavior
- Per-dance tap_time
- Preview: “1 tap = X, 2 taps = Y, hold = Z”

### I4 Hold-tap / mod-tap / layer-tap — P0 — VIAL REMAP KMK

- Visual: tap face / hold face
- Params: tap_time, prefer_hold, tap_interrupted, repeat
- Presets: HRM (home-row mods) wizard with recommended settings (Vial’s 2026 recipe)

### I5 One-shot — P0 — VIAL

- OSM / OSL
- Timeout setting

### I6 Autoshift — P1 — VIAL KMK

- Enable, tap_time, optional per-key exclude

### I7 Caps Word — P1 — VIAL KMK

- Assign key; document continue/stop keys

### I8 Key overrides — P1 — VIAL (needs runtime work on KMK)

- If KMK lacks it, implement a workshop module or hide until QMK backend

### I9 Leader / sequences — P2 — QMK KMK combos sequences

- KMK Sequence + `KC.LEADER` covers a lot without QMK Leader

### I10 Dynamic sequences (record on the keyboard) — P2 — KMK

### I11 Key lock / Layer lock / Repeat / Alt-repeat — P2 — VIAL

Implement if cheap in KMK; otherwise QMK backend.

### I12 Unicode — P1 — QMK (Vial hack)

- First-class “type this character” using OS profile (Win/macOS/Linux)
- Better than telling users to build macros by hand

### I13 Custom / user keycodes — P1 — VIAL POG

- Overlay Python or named hooks
- Appear in the picker under User
- Never overwritten by “update runtime”

---

## J. Encoders, pointing, extra inputs

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| J1 | Per-layer CW / CCW behavior | P0 | POG VIAL | |
| J2 | Encoder click as a key | P0 | POG wishlist VIAL | |
| J3 | Multiple encoders, named | P0 | POG | |
| J4 | Encoder map visual on the layout | P1 | VIAL | |
| J5 | Mouse / trackball / cirque map | P3 | QMK KMK | |
| J6 | Analog / joystick | P3 | QMK | |

---

## K. Lighting

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| K1 | RGB underglow: mode, HSV, speed, on/off | P0 | POG VIA VIAL | |
| K2 | Live preview / live write | P1 | VIA | |
| K3 | RGB keycodes on keymap | P0 | POG VIA | |
| K4 | Per-key colormap (if hardware is matrix RGB) | P2 | VIAL CHRY | Rare on KMK handwires |
| K5 | Layer-reactive lighting rules | P2 | QMK | |
| K6 | Mono backlight | P1 | VIA | |
| K7 | Status LEDs (layer indicators) | P2 | KMK | |
| K8 | Definition-driven custom menus (range, toggle, dropdown, color) | P2 | VIA | Useful if we grow hardware modules |

---

## L. Settings (Vial “QMK Settings” equivalent)

| ID | Setting group | Pri | Compliance |
|---|---|---|---|
| L1 | Tapping term (global + per-key) | P0 | VIAL |
| L2 | Permissive hold / tap interrupted / prefer hold (KMK names) | P0 | VIAL |
| L3 | Combo term | P0 | VIAL |
| L4 | One-shot timeout | P0 | VIAL |
| L5 | Mouse key speed | P1 | VIAL |
| L6 | Autoshift timing | P1 | VIAL |
| L7 | Grave escape / magic-key equivalents if runtime has them | P2 | VIAL |
| L8 | USB / NKRO / bootmagic equivalents | P2 | QMK |
| L9 | Power / sleep timeouts | P2 | KMK ZMK |

Expose **only** what capabilities report.

---

## M. Live protocol and persist

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| M1 | Capability query | P0 | VIAL NEW | |
| M2 | Get/set keymap entry | P0 | VIA VIAL | |
| M3 | Get/set whole keymap | P0 | POG serial | |
| M4 | Get/set macros, combos, tap dances | P0 | VIAL | |
| M5 | Get/set RGB now | P1 | VIA | |
| M6 | Reload / soft reset / DFU / safe | P0 | POG | |
| M7 | Toggle drive visibility | P0 | POG | |
| M8 | Unlock session | P0 | VIAL ZMK | |
| M9 | HID raw (flagship) | P0 | VIA VIAL | New KMK module |
| M10 | Serial fallback (improved Pog protocol: versioned, CRC, larger chunks, 115200) | P0 | POG | Drive-hidden boards |
| M11 | Atomic file replace on device | P0 | POG | |
| M12 | Progress + cancel + retry | P0 | POG | |
| M13 | Conflict: device edited elsewhere | P1 | NEW | |

---

## N. Testing and diagnostics

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| N1 | Visual key tester (press → highlight, show matrix + behavior) | P0 | VIA VIAL REMAP | |
| N2 | Optional click sound | P2 | VIA | |
| N3 | Matrix ghosting / NKRO check | P1 | REMAP | |
| N4 | Serial / HID console (REPL + workshop logs) | P0 | POG QMK Toolbox | |
| N5 | Send ctrl-c / ctrl-d / raw line to REPL | P1 | POG | Gated |
| N6 | HID listen (QMK console) | P2 | QMK Toolbox | |
| N7 | “What firmware is on this board vs this project” diff | P0 | POG wishlist | |
| N8 | Pin probe (set high/low, read) in detection mode | P2 | NEW | |
| N9 | Typing practice on current keymap | P2 | REMAP | Differentiator if live |

---

## O. Firmware management

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| O1 | Install/update KMK runtime, hashed | P0 | POG | |
| O2 | Update workshop runtime without wiping keymap | P0 | POG gap | |
| O3 | Flashing mode automatic vs manual (user-owned `code.py`) | P0 | POG | |
| O4 | Feature flags → capability-driven generation | P0 | POG upgrade | |
| O5 | Show generated files, allow inspect before write | P0 | NEW | |
| O6 | Never overwrite `overlays/` unless forced | P0 | POG gap | |
| O7 | CircuitPython UF2 library (cache) | P1 | NEW | |
| O8 | QMK compile/flash | P3 | QMKC Toolbox | |
| O9 | ZMK build (west / GH) | P3 | ZMK | |
| O10 | Bundled known-good images per official kit | P2 | CHRY REMAP | |

---

## P. Community and sharing

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| P1 | `.kbpack` send via file | P1 | POG wishlist | |
| P2 | QR of *wiring summary* or pack URL | P2 | POG wishlist | |
| P3 | Publish to a catalog (opt-in) | P3 | REMAP | |
| P4 | Import community keymap onto a compatible board (layout fingerprint) | P2 | REMAP | |
| P5 | No Web3 | P0 | anti-POG | Explicit non-feature |
| P6 | Vendor org accounts | P3 | REMAP | |

---

## Q. Multi-firmware

| ID | Feature | Pri | Compliance | Notes |
|---|---|---|---|---|
| Q1 | Canonical Behavior IR (intermediate representation) | P0 | NEW | Even for KMK-only |
| Q2 | KMK emitter + parser | P0 | POG | |
| Q3 | QMK keymap.json / info.json emitter | P1 | QMKC | Export-only is enough at first |
| Q4 | VIA/Vial HID client | P2 | VIA VIAL | |
| Q5 | ZMK `.keymap` emitter | P2 | ZMK | |
| Q6 | ZMK Studio RPC | P3 | ZMK | |
| Q7 | Kaleidoscope Focus | P3 | CHRY | Out of scope unless asked |

---

## R. Standout features (see also 11)

| ID | Feature | Pri | Why it wins |
|---|---|---|---|
| R1 | Press-to-map workshop with live wiring diagram | P0/P1 | Nobody else |
| R2 | Capability-negotiated UI | P0 | Honest Vial |
| R3 | Safe live + generation split | P0 | Honest Pog |
| R4 | `.kbpack` as emailable keyboard | P1 | Share without a startup |
| R5 | Host-aware legends + Unicode helper | P1 | DE/EU users |
| R6 | Split as one object | P0 | Ergo DIY |
| R7 | Firmware diff / “why did this break” | P0 | Support |
| R8 | HRM wizard | P1 | Vial users still struggle |
| R9 | Typing practice against the live board | P2 | Remap 2026 |
| R10 | Mobile live-remap companion | P2 | CMP path |
| R11 | Board fingerprint → apply community keymap | P2 | Remap without account |
| R12 | Photo + pin overlay documentation mode | P2 | Kit vendors |

---

## S. Explicit non-goals (v1)

1. Becoming a general CircuitPython IDE
2. Supporting AVR QMK as a *workshop* target (no C compiler in v1)
3. Per-application keymap switching at the OS level
4. Cloud accounts required for any P0 feature
5. Cryptocurrency, wallets, NFTs
6. Implementing SerialACE in the GUI
7. 100% QMK feature coverage (Leader, Autocorrect, Swap Hands, pointing devices)
8. Replacing QMK Toolbox for every bootloader
9. Designing PCBs / KiCad
10. Running the workshop exclusively in a browser (WebHID cannot copy CIRCUITPY reliably)

Browser **live remap** for already-provisioned boards is a P2 *companion*, not the workshop.

---

## T. v1 cut line (what “MVP ship” means)

Ship when all **P0** items are done or explicitly waived in writing.

A truthful v1 tagline:

> Install CircuitPython on a Pico, press every key once, get a keyboard, then remap it live like Vial — including combos, tap dance, hold-tap, and macros — even with the USB drive hidden.

If that sentence is false, it is not v1.
