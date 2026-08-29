# 02 — Pog as-is analysis

Pog (this repository, v2.2.3) is an Electron 21 + Vue 3 desktop app that configures **KMK firmware on CircuitPython** boards. Author: Jan Lunge. This fork lives at `0x49b/pog`; upstream is `JanLunge/pog`. License: MIT.

It is **not** a QMK tool. It does not speak VIA/Vial HID. It does not compile C. It writes files onto a CIRCUITPY volume and, optionally, shuttles JSON over USB serial.

---

## 1. Users and jobs

### Primary user

A DIY keyboard builder with:

- an RP2040 (or similar CircuitPython MCU)
- a hand-wired or PCB matrix
- no QMK port, no VIA definition, no compiler

Job: **get a typing keyboard without writing Python by hand.**

### Secondary user

Someone who already has a Pog keyboard and wants to change the keymap, add a layer, tweak RGB, or hide the USB drive.

Job: **edit and redeploy config.** Today this still feels like flashing, not like VIA.

### Non-users (today)

- Owners of VIA/Vial/QMK production keyboards
- ZMK / nice!nano wireless users
- People who only want to remap an already-finished board in a browser

---

## 2. Application map

### Process architecture

```
Renderer (Vue 3, hash router, Tailwind/DaisyUI)
        │  contextBridge window.api
        ▼
Preload (src/preload/index.ts)
        │  ipcRenderer
        ▼
Main (Electron)
  index.ts          window, menus, serial scan, chunk protocol, debug REPL
  selectKeyboard.ts drive picker, pog.json load, dual-port pairing
  saveConfig.ts     write pog.json + Python templates
  kmkUpdater.ts     download pinned KMK zip, copy onto drive, flash detection
  keyboardDetector.ts  115200 detection session on port B
  store.ts          currentKeyboard path/id/ports
  pythontemplates/  string literals that become device files
```

There is no domain layer independent of Vue or Electron. The `Keyboard` class in the renderer *is* the schema. Main process has a much thinner `currentKeyboard` struct. They drift.

### Screens and routes

| Route | Job |
|---|---|
| `/` Launch | History of boards, connection badges, add keyboard |
| `/add-keyboard` | Choose manual vs automatic vs import |
| `/setup-wizard` | Manual: KMK → info → matrix → pins → coordmap → layout |
| `/automatic-setup/circuit-python` | Pick CIRCUITPY drive |
| `/automatic-setup/method` | Setup method |
| `/automatic-setup/firmware` | Install Pog detection/runtime files |
| `/automatic-setup/mapping` | Press-each-key matrix detection |
| `/keyboard-selector` | Pick among discovered boards |
| `/configurator/keymap` | Visual keymap + picker + macro modal |
| `/configurator/layout-editor` | Geometry, KLE/QMK import, variants |
| `/configurator/encoder` | Encoder pads + per-layer CW/CCW |
| `/configurator/rgb` | Pin, LED count, animation mode, HSV |
| `/configurator/info` | Name, manufacturer, tags, feature flags |
| `/configurator/matrix` | Rows/cols, wiring method, split type |
| `/configurator/pins` | Pin lists, diode, pin prefix, MCU, split pins |
| `/configurator/coordmap` | Assistant + textarea grid |
| `/configurator/raw-keymap` | Plain text keycodes |
| `/configurator/firmware` | Install/update KMK, update Pog files, backup/restore |
| `/configurator/community` | WalletConnect stub — not in the sidebar |

---

## 3. Feature inventory (honest)

### 3.1 Launch and library

**Present**

- Keyboard history in `localStorage` (`keyboardHistory`)
- Up to 100 backups per board id
- Layout preview on each card (`KeyboardLayout` static mode)
- USB-mounted vs disconnected badge (path existence check)
- Serial-available / serial-only / read-only-serial badges
- Rescan button
- Remove from history
- Add keyboard CTA

**Weak**

- History is a blob in localStorage, not a file library
- No search, tags filter, or folders
- No “this board is the same physical device as that history entry” merge besides id
- Serial scan filters manufacturer `pog` / `pog-*` / `*-pog` — anything else is invisible
- `listKeyboards` IPC is commented out

### 3.2 Connection modes

| Mode | How | What you can do |
|---|---|---|
| USB mass storage | CIRCUITPY mounted, path known | Full read/write of `pog.json` and Python files, KMK copy |
| Serial (beta) | CDC at 9600 baud, manufacturer Pog | Pull full config (`info`), push config (`save`), debug REPL |
| Dual | Drive + serial | Best case; UI still treats them as parallel accidents |
| History-only | Neither | Read-only memory of last serialize() |

Two CDC ports are common on CircuitPython (console + data). Pog tries to sort by path and call the lower one A and the higher one B. Detection uses B at 115200. Config serial uses the Pog-identified port at 9600. Debug REPL is a *third* connection (`debugPort`) that fights with the others.

**This is the single biggest reliability hole in the app.** Port ownership is not a state machine. Close-on-quit is a known stack-overflow / hang risk (`triedToQuit`, `process.exit(0)`).

### 3.3 Automatic setup (the crown jewel)

1. User installs CircuitPython themselves (app only links to circuitpython.org).
2. User picks a USB drive from `drivelist` (removable USB only).
3. App writes detection `code.py` + `boot.py`.
4. Board resets. App opens serial port B @ 115200.
5. Detection firmware brute-forces GPIO: drive one pin low, read others, infer row/col pairs.
6. Device emits JSON: `start_detection`, `new_key_press`, `existing_key_press`, `used_pins`.
7. UI shows detected rows, cols, pressed keys; user presses each switch once.
8. Later: coordmap assistant (different firmware mode) to order keys for the keymap.
9. KMK tree is downloaded (pinned SHA) and copied onto the drive.
10. Pog Python files are written.

**Strength:** no other mainstream configurator does this.

**Weakness:** GPIO brute force is slow, noisy, and MCU-specific. No skip-list of power/USB pins per controller. Helios GP29 is the only special-case mentioned. No wiring preview. No “I already know my matrix” fast path inside the automatic flow (that is the manual wizard).

### 3.4 Manual setup wizard

Linear: firmware → name/features → matrix → pins → coordmap → layout.

Useful, but it is a series of forms, not a guided hardware interview. Missing: “show me a Pico pinout and click pins,” diode orientation test, and generate-layout-from-matrix.

### 3.5 Layout editor

**Present**

- KLE-like key geometry: `x,y,w,h,x2,y2,w2,h2,r,rx,ry`
- Add/remove keys, multi-select, delta nudges
- KLE JSON import (`KleToPog` via JSON5)
- QMK `info.json` layout import
- Export raw Pog layout JSON
- Layout variants (`layouts[]` + per-key `variant: [layoutIdx, variantIdx]`)
- Encoder binding (`encoderIndex`)
- Coord index (`idx`) and leftover `matrix: [row,col]` labels

**Missing vs Keyboard Layout Editor / Keyboard Layout Studio**

- 9-position legends
- Proper ISO enter as a first-class tool (geometry exists, no dedicated control)
- Undo/redo stack (partial at best)
- Alignment / distribute
- Generate N×M ortho grid from matrix dimensions
- Bulk delete of a box select as a named action (viselect is present)
- Switch photos / case overlay
- Stabilizer / switch metadata

### 3.6 Keymap editor

**Present**

- Unlimited layers (KMK has no VIA-style 4-layer EEPROM cap)
- Add / remove / duplicate layer
- Per-layer name and color
- Visual keyboard, multi-select
- Key picker layouts: QWERTY, Colemak, Colemak DH, Dvorak
- Categories: Basic, Layers (MO/TG/TO/TT/LM), KMK (RESET/RELOAD/DEBUG), App/Media/Mouse, RGB, Advanced docs
- Raw keycode input
- Auto-select next key, reduce colors
- Templates: Macro, String, Tap Dance, Custom Key (they insert *strings*)
- Custom macro modal: 2–4 keys → `KC.MACRO(Press(...), Release(...))` simultaneous combo-like macro, not a timed sequence

**Missing vs Vial**

- Nested builders for `HT` / `MT` / `LT` (hold-tap, mod-tap, layer-tap)
- One-shot (OSM/OSL) picker
- Tap dance *editor* (tap 1, tap 2, hold, tapping term)
- Combo editor
- Macro recorder, delays, down/up/tap actions, mouse moves
- Key overrides
- “Any” key with validation against firmware capabilities
- Searchable keycode catalog (names + aliases + docs)
- Host locale legends
- Layer tap-toggle count, default layer (DF)

Layer-tap `KC.LT()` is commented out in the picker. That is a Vial-basic feature.

### 3.7 Wiring, pins, MCU

**Present**

- Matrix vs direct pin
- COL2ROW / ROW2COL
- Pin prefix: `gp` → `board.GP17`, `board` → `board.GP17`-style names, `none`, `quickpin` → `pins[n]`
- Split types: normal, splitBLE, splitSerial, splitOnewire
- Split side: left, right, vbus, label
- `splitPinA/B`, `vbusPin`, `splitUsePio`, `splitFlip`, `splitUartFlip`
- MCU catalog JSON: 0xCB Helios, Raspberry Pi Pico, with images
- Direct pin count vs matrix rows×cols; split doubles physical key count

**Missing**

- Real pinmux database (ADC, USB, flash, LED, debug)
- Click-to-assign on a pinout drawing
- Validation that a pin exists *and* is legal for that role
- Wiring preview (schematic or “row 2 is GP4”)
- I2C / SPI / UART conflict detection (OLED + split UART + RGB)
- Support for more than two MCUs in a useful way
- Known typo: MatrixSetup uses `splitBle` vs store `splitBLE`

### 3.8 Coord map

KMK maps matrix coordinates to keymap indices via `coord_mapping`. Pog has:

- A firmware assistant (`coordmaphelper.py`) that types 3-digit indices
- A textarea editor (`001`, `spc`, etc.)
- `coordMapSetup` flag that makes `code.py` boot the assistant instead of the keyboard

This is necessary and poorly explained. Users do not know why they are typing numbers. The successor must visualize “physical key 27 = matrix (3,5) = keymap index 27.”

### 3.9 Encoders and RGB

Encoders: list of `{pad_a, pad_b}`, keymap `[layer][encoder][cw, ccw]`. No detent config, no press/click pin, no encoder-as-scanner for split-both-sides (KMK docs warn about this).

RGB: `rgbPin`, `rgbNumLeds`, `rgbOptions` (animationMode 0–8, HSV, speed, breathe, knight length). No per-key RGB matrix, no LED map matching layout, no layer-reactive lighting GUI, no backlight (mono LED) despite `ledPin`/`ledLength` existing in `pog.py`.

### 3.10 Firmware generation

On save to a mounted drive, Pog always writes `pog.json`, then writes these Python files **if missing or if `writeFirmware`**:

| File | Role |
|---|---|
| `pog.py` | Load JSON, render pins, expose tuples |
| `code.py` | Entry: coordmap helper *or* `POGKeyboard().go()` |
| `kb.py` | Feature-flagged KMK modules/extensions + split + scanners |
| `keymap.py` | `eval` keymap strings, encoder map, combos |
| `boot.py` | USB ID “Pog”, dual CDC, NVM drive hide |
| `pog_serial.py` | Chunk protocol |
| `coordmaphelper.py` | Index discovery keyboard |
| `customkeys.py` | DFUMODE, SAFEMODE, ToggleDrive |
| `kmk/` | Full upstream tree at SHA `5a6669d1da219444e027fb20f57d4f5b3ecdedfe` |

KMK install is a zip download to `%APPDATA%/pog/` then a recursive copy with progress events. It requires the drive to be mounted. Serial-only keyboards cannot update KMK.

**Structural problem:** the runtime is *copied source*, not a versioned, tested “Pog KMK distribution.” Any user edit to `kb.py` is one “Update POG files” away from being destroyed. The firmware screen warns, but the model does not support a user overlay.

### 3.11 Serial protocol (config)

9600 baud, newline-delimited.

Host → device: `info`, `info_simple`, `save`, `saveKeymap`, `reset`, `drive`, `1`, `0`, `y`.

Device → host (`info`): JSON chunks `{type: pogconfig, current_chunk, total_chunks, data, totalsize, cross_sum}` of 800 bytes.

Host → device (`save`): JSON chunks of 1200 bytes.

Integrity: Unicode cross-sum, not CRC. No session id. No capability query beyond `info_simple` (`driveMounted`, `name`, `manufacturer`, `id`, `board`).

`writeKeymapViaSerial` exists in main and has **no renderer caller**.

### 3.12 Debug REPL

A second serial session at 9600 with `ctrlc` (`\x03\x03`) and `ctrld` (`\x04`). Lines stream to a store. Useful, easy to collide with the config port.

### 3.13 Features KMK supports that Pog only half-exposes

Enabled by `kbFeatures` in `kb.py`:

| Flag | KMK piece | GUI |
|---|---|---|
| `basic` | Layers, MediaKeys | Implicit |
| `serial` | `pogSerial` | Implicit if using serial |
| `oneshot` | StickyKeys | Toggle only |
| `tapdance` | TapDance, tap_time=200 hardcoded | Template |
| `holdtap` | HoldTap | Template / raw |
| `mousekeys` | MouseKeys | Some picker keys |
| `combos` | Combos | Schema in JSON, no editor |
| `macros` | Macros **always loaded** | Mini builder |
| `rgb` | RGB extension | RGB screen |
| `international` | International | Info toggle |
| `capsword` | CapsWord (commented TODO) | Info toggle |
| `autoshift` | Autoshift, `autoshiftTapTime` | Firmware only |

KMK also has, unused by Pog: Power, MIDI, Steno, Dynamic Sequences, Mouse Jiggler, SpaceMouse, ADNS9800, Pimoroni trackball, EasyPoint, OLED, LockStatus, Status LED, Stringy Keymaps, SerialACE (dangerous), BLE HID.

### 3.14 Community / Web3

`Community.vue` instantiates WalletConnect + wagmi against a hardcoded project id. Not linked from the configurator menu. Dependencies (`@wagmi/core`, `@web3modal/*`, `ethers`) still ship. This should not be carried forward.

---

## 4. Data model as it exists

Canonical serialize() fields (renderer `Keyboard`):

```
id, name, manufacturer, description, tags, controller, keyboardType,
wiringMethod, diodeDirection, rows, cols, pins,
rowPins, colPins, directPins,
encoders, layouts, keys, keymap, encoderKeymap, layers,
split, splitPinA, splitPinB, splitSide, vbusPin,
splitUsePio, splitFlip, splitUartFlip,
coordMap, pinPrefix, coordMapSetup,
rgbPin, rgbNumLeds, rgbOptions, kbFeatures,
flashingMode, lastEdited
```

Firmware-only / drifted fields: `combos[]`, `splitTargetLeft`, `autoshiftTapTime`, `ledPin`, `ledLength`.

Key object: geometry + `matrix` + `variant` + `idx` + `encoderIndex`.

This schema is **keyboard-as-one-JSON**. That is good for DIY. It is bad for:

- sharing a keymap without sharing wiring
- applying a community keymap to a different wiring
- QMK `info.json` vs `keymap.json` separation
- capability negotiation

The successor should split **Board / Layout / Keymap / RuntimeSettings** while remaining able to emit one `.kbpack` or one `pog.json` for compatibility.

---

## 5. Quality, bugs, and operational reality

From README and code:

- Maximum call stack on quit (serial close vs Electron lifecycle)
- Node 16 documented, CI uses Node 18, Electron 21 is old
- `request` (deprecated) for KMK download
- `npmRebuild: false` — native modules (`serialport`, `drivelist`) are packaging landmines
- Auto-updater present and mostly commented out
- Help menu opens electronjs.org
- Duplicate `coordmap` route in the router
- Community crypto deps
- Pin validation stub
- Detection requires two serial ports; some boards only expose one
- CircuitPython 10 fixes landed recently — the device runtime *does* move

CI: tag `v*` → Linux AppImage, Windows portable/NSIS, macOS x64+arm64 signed/notarized, draft GitHub release.

---

## 6. What to preserve at all costs

These are Pog’s actual IP. A rewrite that drops them is a different, worse product.

1. Bare-MCU onboarding (CircuitPython → files on CIRCUITPY)
2. Automatic matrix detection by keypress
3. Coord map assistant
4. `pog.json` as a human-readable source of truth
5. Generation of a bootable KMK tree without a C compiler
6. Drive hide + serial still works (`ToggleDrive`, NVM[0])
7. Split serial / BLE / onewire configuration in the GUI
8. KLE import
9. Local history with backups
10. “Update KMK” and “Update Pog files” as separate actions
11. Custom keys file the user can own
12. Offline operation (except KMK zip download)

---

## 7. What not to preserve

1. Electron + contextBridge IPC as the architecture
2. Vue class store as the schema
3. WalletConnect
4. 9600-baud chunk protocol as the *primary* live path (keep as fallback)
5. String-eval keymaps (`keymap.py` evals keycode strings) as the long-term runtime
6. Copying the entire KMK git tree onto every keyboard as the only distribution
7. Hardcoded Helios/Pico as “the MCU catalog”
8. Feature flags that do not match a capability query
9. Templates that only paste source text
10. A Help link to Electron’s website
