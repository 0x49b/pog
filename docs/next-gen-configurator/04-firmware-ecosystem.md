# 04 — Firmware ecosystem

A configurator is only as complete as the firmware it can honestly drive. This document is the firmware-side requirements context: what each stack can do, how a host talks to it, and what a successor should generate or speak.

---

## 1. The four runtimes that matter

| Runtime | Language | Typical MCU | Strength | Host config story |
|---|---|---|---|---|
| **KMK** | Python on CircuitPython | RP2040, some nRF, SAMD | Fast DIY, filesystem, REPL | Almost none except Pog |
| **QMK** | C | AVR, STM32, RP2040, lots | Depth, keyboards, VIA/Vial | VIA, Vial, Remap, Configurator |
| **ZMK** | C + devicetree on Zephyr | nRF52, some STM32, RP2040 emerging | BLE, splits, power | Studio (MVP), text `.keymap` |
| **Kaleidoscope** | C++ Arduino | Keyboardio, ATMega | Plugins, Focus | Chrysalis |

v1 recommendation: **KMK is the generated and live-controlled runtime.** QMK and ZMK are *modelled* and optionally *imported/exported*. Kaleidoscope is out of scope.

---

## 2. KMK in depth (v1 target)

### 2.1 Execution model

CircuitPython mounts a USB mass-storage volume (CIRCUITPY) and usually two CDC ports (REPL + data). `boot.py` runs first (USB config, drive hide). `code.py` is the main loop.

KMK’s `KMKKeyboard` owns:

- `row_pins`, `col_pins`, `diode_orientation` **or** a custom `matrix` / `KeysScanner`
- `coord_mapping` — flattened index remap
- `keymap` — list of layers, each a list of `Key` objects
- `modules[]` — change core behavior (layers, holdtap, split, …)
- `extensions[]` — sandboxed extras (RGB, media, OLED)

Pog generates this graph from `pog.json` at **import time** (`pog.py` uses `eval` on pin strings; `keymap.py` uses `eval` on keycode strings). That is convenient and dangerous. The successor runtime should parse a structured keymap into KMK keys in Python without `eval`.

### 2.2 Module catalog the GUI must cover

#### Must (Pog-complete + Vial-complete)

| Module | Keycodes / API the GUI must emit | Settings the GUI must expose |
|---|---|---|
| Layers | `KC.MO n`, `TG`, `TO`, `TT`, `LM(layer, mod)`, `LT(layer, kc)`, `DF` if available | Combo layers (optional) |
| HoldTap | `KC.HT(tap, hold, prefer_hold, tap_interrupted, tap_time, repeat)` plus aliases `LSFT`, `LCTL`, … | Global `tap_time`; per-key tap_time |
| TapDance | `KC.TD(kc1, kc2, …)` | `tap_time` |
| Combos | `Chord(keys, result, timeout, match_coord, per_key_timeout, fast_reset)`; `Sequence(...)`; `KC.LEADER` | Global combo term |
| Macros | `KC.MACRO(Press, Release, Tap, Delay, OnPress callback)` | None; macros are data |
| StickyKeys | `KC.OS()`, oneshot mods | Release timeout |
| MouseKeys | `KC.MS_UP` etc. | Speed / interval if KMK exposes them |
| Split | No keycodes; constructor args | type, side, pins, pio, flip, uart_flip, target_left |
| EncoderHandler | Encoder map triples/pairs | pad_a, pad_b, optional button |
| RGB | `KC.RGB_*` | pin, num_pixels, hue/sat/val, animation, speed |
| Macros always-on | Custom keys | User `customkeys` overlay |

#### Should (workshop-complete)

| Module / ext | Why |
|---|---|
| Power | Battery / wireless KMK boards |
| BLE HID | Connect to host without USB — KMK supports this independently of split BLE |
| OLED | Status of layer, RGB, BLE; very common on DIY |
| LED (mono backlight) | Already half-parsed in `pog.py` |
| LockStatus | Drive caps/num/scroll indicators |
| International | ISO keys, already a toggle |
| Autoshift | Already in `kb.py` |
| CapsWord | Already in `kb.py` (untested) |
| Dynamic Sequences | Vial-like record-on-keyboard macros |

#### Later / hardware-specific

Pointing devices (Pimoroni, ADNS9800, EasyPoint), MIDI, Steno, SpaceMouse, Mouse Jiggler.

#### Never expose casually

**SerialACE** — arbitrary code execution over the data serial. If a debug “run snippet” exists, it is a developer-mode, unlock-gated, clearly labeled hazard.

### 2.3 Scanners

KMK can scan:

- Diode matrix (`DiodeOrientation.COL2ROW` / `ROW2COL`)
- Direct pins (`KeysScanner`)
- Encoder-as-scanner (needed when both split halves have encoders)

Pog supports the first two. The successor must keep both and add “encoder scanner” when split + encoders-on-both-sides is selected (KMK docs require it).

### 2.4 Split

| Type | Pog name | KMK |
|---|---|---|
| UART 1-wire or 2-wire | `splitSerial` / `splitOnewire` | `SplitType.UART`, `data_pin`, optional `data_pin2`, `use_pio`, `uart_flip` |
| BLE half-to-half | `splitBLE` | `SplitType.BLE` (testing / limited support upstream) |
| Side detect | left/right/vbus/label | `SplitSide`, VBUS pin, drive label `…L`/`…R`, EE-hands style `split_side=None` |

Successor must model **two devices that share one Board id**: left drive, right drive, left serial, right serial, which side is USB target.

### 2.5 Storage and “EEPROM”

CircuitPython has a filesystem. This is a **strategic advantage over VIA/Vial**.

Do not imitate AVR EEPROM limits (4 layers, 16 macros) unless the user is on a tiny board. Persist:

- `/board.json` or `/pog.json` — definition + keymap
- `/keymap.json` — optional split so keymap share is easy
- `/runtime.json` — RGB state, last layer, BLE peers
- `/kmk/` — library
- `/overlays/customkeys.py` — user-owned, never overwritten unless asked

Live remap can write JSON and trigger a reload, or mutate in-memory keymap and flush. Reloading CircuitPython is slower than QMK EEPROM write. Measure both; prefer in-memory mutate + atomic file replace.

### 2.6 USB identity

Pog’s `boot.py` calls `supervisor.set_usb_identification("Pog", "Pog Keyboard")` so the host can find it. The successor should set:

- Manufacturer: product name
- Product: board name
- Serial number: stable Board id (ULID) so port pairing is deterministic

Without a stable USB serial, “auto-select the correct port” (Pog wishlist) stays unsolved.

### 2.7 CircuitPython as a platform constraint

- Users must flash CircuitPython UF2 themselves today. The successor should detect a UF2 bootloader drive (`RPI-RP2`, `PICODISK`, etc.) and offer to drop the correct CircuitPython UF2 (download cache by MCU + CP version).
- CircuitPython version drift broke Pog before (CP 10 fixes in history). Pin a supported CP range and warn.
- Drive hide via `storage.disable_usb_drive()` + NVM is a beloved Pog feature. Keep it. Live protocol must work with the drive hidden — **this is why HID or a dedicated CDC protocol is mandatory**, not optional polish.
- REPL on CDC is a gift for diagnostics and a footgun (user pastes code, baud mismatch, port busy).

---

## 3. QMK / VIA / Vial (v2 backend)

### 3.1 Why support it later

Most keyboards people buy already run QMK. If the successor never speaks VIA/Vial, it cannot become “the” configurator; it remains a KMK workshop. That can be a valid business/product choice. It must be an explicit one.

### 3.2 How hosts talk to QMK

**VIA protocol (Raw HID):**

- Usage page / usage specified by VIA
- Command ids for dynamic keymap get/set, macros, lighting, custom menus
- Keyboard id in firmware → fetch definition from via.json catalog or sideload
- EEPROM layout is the capacity model

**Vial protocol:**

- VIA-compatible subset plus extra command ids
- Definition (layout, lighting, feature bits, unlock) stored **in firmware**
- Unlock combo required before writes (security)

**QMK compile path:**

- `qmk compile -kb X -km Y`
- `info.json` / `keyboard.json` + `keymap.json` or `keymap.c`
- Flash via Toolbox / `qmk flash`

### 3.3 What the successor can do without becoming vial-qmk

Phase Q1 (read-only): detect a VIA/Vial device, render layout from embedded or sideloaded definition, show keymap, export `.vil` / VIA JSON.

Phase Q2 (write): implement VIA protocol writes for keymap, macros, lighting. Reuse public protocol docs. Do not fork QMK.

Phase Q3 (workshop → QMK): emit `info.json` + `keymap.json` from our Board model so a handwire can *become* a QMK project. Cloud or local compile is a separate product decision.

**Do not** ship a GPL vial-qmk fork as the only KMK path. Keep licenses clean.

### 3.4 Definition formats to ingest/emit

| Format | Use |
|---|---|
| VIA `via.json` / keyboard definition | Import layout + lighting menus + matrix |
| Vial embedded + `.vil` keymap | Import/export live keymap |
| QMK `info.json` / `keyboard.json` | Import layouts, matrix pins, USB ids (Pog already imports layouts) |
| QMK `keymap.json` | Import/export keymaps |
| KLE JSON | Geometry |

---

## 4. ZMK (v3 backend)

### 4.1 Model differences

ZMK keymaps are **devicetree behaviors**, not keycode strings.

```
&mt LCTRL A    // mod-tap
&lt 1 TAB      // layer-tap
&mo 1          // momentary
&kp ESC
```

A naive “store `KC.HT(KC.A, KC.LCTRL)` and print ZMK” works for a subset and fails for Mod-Morph, conditional layers, and sensor bindings.

Studio RPC is protobuf-like over USB UART (and BLE). Reverse-engineering Studio is possible but hostile. Prefer: our model → `.keymap` export first; Studio protocol later if ZMK publishes a stable API.

### 4.2 What to steal regardless of backend

- `reserved` extra layers at build time
- Unlock key on the device
- Physical layout as a first-class firmware object (`keys` positions)
- BLE as a first-class transport
- Behavior catalog with parameters, not free-string keycodes

---

## 5. Bootloader and flashing matrix

| Board state | Volume / USB | Successor action |
|---|---|---|
| RP2040 bootloader | `RPI-RP2` UF2 | Copy CircuitPython UF2 or a custom UF2 |
| CircuitPython, drive visible | `CIRCUITPY` | Write files, copy KMK |
| CircuitPython, drive hidden | CDC only | Live protocol only; `drive` command to remount |
| CircuitPython safe mode | CIRCUITPY + yellow | Show reason, offer to restore `code.py` |
| QMK bootloader | DFU/HID/CDC/MSC | Toolbox-class flash (later) |
| ZMK UF2 | `NICENANO` etc. | Copy `.uf2` from a build |
| Broken `code.py` | Often still CIRCUITPY | Recovery: write known-good `code.py` |

Pog handles rows 2–3 only. Workshop-complete requires rows 1, 4, and 7.

---

## 6. Capability negotiation (required concept)

VIA/Vial encode capabilities in firmware + definition. Pog encodes them as a checkbox list in JSON that may lie.

The successor device runtime must answer:

```
GET /capabilities →
  protocolVersion
  firmware: { family: "kmk", version, git }
  layers: { max, current }
  features: ["holdtap","tapdance","combos","macros","rgb","split","ble",...]
  limits: { macros: 64, tapdance: 32, combos: 64 }
  hardware: { mcu, pins[], encoders, rgbCount, split, pointing }
  storage: { driveMounted, freeBytes }
  unlock: { required, unlocked }
```

The UI is a function of this document, not of a local checkbox. If the user enables OLED in the workshop, the **next generated firmware** advertises `oled`, and the live session then shows the OLED editor.

This is the architectural fix for “GUI features that do not match firmware.”

---

## 7. Security and safety

| Risk | Mitigation |
|---|---|
| Malicious host rewrites keymap via HID | Unlock combo or button; session timeout; optional lock |
| Malicious `.kbpack` overwrites `customkeys.py` with SerialACE | Signed packs optional; never auto-run Python overlays; diff preview |
| Drive copy during PC write (FAT corruption) | Atomic replace (`pog.json.tmp` → rename); unmount guidance |
| Bricking `boot.py` | Keep a recovery UF2 / safe-mode story; never write `boot.py` unless asked |
| SerialACE | Hidden behind developer unlock |
| Downloading KMK zip over HTTP | HTTPS only; pin SHA; verify hash (Pog pins SHA but does not verify) |
| USB identification spoofing | Treat manufacturer string as a hint, not trust; pair by serial + user confirm |

---

## 8. Recommended KMK runtime strategy

Stop treating “whatever Python we last generated” as the firmware product.

Ship a **versioned runtime package**:

```
/lib/kmk/                 # upstream KMK, pinned, hashed
/lib/workshop/            # our protocol, capability, live keymap, detectors
code.py                   # 10-line stub: import workshop; workshop.main()
board.json                # data
overlays/customkeys.py    # user
```

Workshop Python is tested in CI on a CircuitPython unix port or hardware-in-the-loop. Generation becomes “write `board.json` + maybe `code.py` stub,” not “overwrite eight templates.”

Pog’s templates remain a **compatibility emitter** for old boards.

This single decision removes half of Pog’s support burden (`kb.py` module order, CP 10 breakages, user edits getting clobbered).
