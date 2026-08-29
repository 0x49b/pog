# 07 — Domain model and interchange

The domain is the product. UI and firmware are projections of it.

Pog collapsed everything into one `Keyboard` class and one `pog.json`. That is easy to dump onto CIRCUITPY and hard to share, validate, or retarget.

---

## 1. Aggregates

```
Library
  └── BoardProject                 // one folder on disk
        ├── Board                  // identity + hardware
        ├── PhysicalLayout         // geometry + variants
        ├── Wiring                 // matrix / direct / split / pins
        ├── Keymap                 // layers + assignments
        ├── BehaviorLibrary        // macros, combos, tapdances, custom
        ├── Lighting
        ├── RuntimeSettings        // tapping term, etc.
        └── Overlays               // user files, never auto-clobber
```

A **DeviceSnapshot** is what we last saw on a PhysicalDevice (capabilities, hashes, ports). It is not part of the project file; it is cache.

---

## 2. Identifiers

| Entity | Id | Notes |
|---|---|---|
| Board | ULID | Pog already uses ULID; keep |
| PhysicalDevice | USB serial or composite key `vid:pid:serial` | |
| LayoutKey | stable ULID | Do not use array index as identity |
| Layer | index + optional ULID | Indices matter for `MO(1)` |
| Behavior | interned canonical string + structured form | See §5 |
| Macro / Combo / TapDance | ULID | Referenced from keymap |

Pog keys have ULIDs in memory but serialize without them (geometry-only). **Successor must persist LayoutKey ids** so combos and backups remain stable when keys move.

---

## 3. Board

```json
{
  "schema": "workshop.board/1",
  "id": "01HX...",
  "name": "My Handwire",
  "manufacturer": "",
  "description": "",
  "tags": ["40%", "ortho"],
  "controller": "rpi-pico",
  "createdAt": "2026-08-29T00:00:00Z",
  "updatedAt": "2026-08-29T00:00:00Z"
}
```

USB identity derived from Board:

- manufacturer = app or vendor
- product = `name`
- serial number = `id` (CircuitPython can set this in `boot.py` / `supervisor`)

---

## 4. Wiring

```json
{
  "method": "matrix",
  "diode": "COL2ROW",
  "rows": 4,
  "cols": 12,
  "rowPins": ["GP2", "GP3", "GP4", "GP5"],
  "colPins": ["GP6", "..."],
  "directPins": [],
  "pinNamespace": "gp",
  "split": {
    "kind": "uart2",
    "sideDetect": "vbus",
    "pinA": "GP0",
    "pinB": "GP1",
    "vbusPin": "VBUS_SENSE",
    "usePio": true,
    "flip": false,
    "uartFlip": false,
    "targetLeft": true
  },
  "encoders": [
    { "id": "01HY...", "name": "Volume", "a": "GP21", "b": "GP22", "button": "GP20", "invert": false }
  ],
  "rgb": { "pin": "GP28", "count": 12, "map": [0, 1, 2] },
  "backlight": { "pin": "", "count": 0 },
  "oled": null
}
```

**Pin** is a structured type, not a magic string:

```
Pin = { port: "GP"|"BOARD"|"RAW"|"QUICK", token: "2"|"LED"|"pins[2]" }
```

Formatters produce KMK `board.GP2`. Validators consult `mcu-catalog.json`.

Pog `pinPrefix` remains an import hint.

---

## 5. Behavior IR

Free strings (`KC.HT(KC.A, KC.LCTRL)`) are the #1 source of GUI poverty. Store **both** structured IR and a canonical print for the raw box.

```json
{
  "kind": "holdTap",
  "tap": { "kind": "hid", "code": "A" },
  "hold": { "kind": "hid", "code": "LCTRL" },
  "preferHold": true,
  "tapInterrupted": false,
  "tapTimeMs": null,
  "repeat": "none"
}
```

Closed kind catalog for v1:

| kind | Fields | KMK print | QMK analogue | ZMK analogue |
|---|---|---|---|---|
| `none` | | `KC.NO` | `KC_NO` | `&none` |
| `trans` | | `KC.TRNS` | `KC_TRNS` | `&trans` |
| `hid` | code, mods[] | `KC.A`, `KC.LGUI(KC.A)` | `KC_A`, `LGUI(KC_A)` | `&kp A` |
| `layerMo` `layerTg` `layerTo` `layerTt` `layerDf` | layer | `KC.MO(1)` | `MO(1)` | `&mo 1` |
| `layerMod` | layer, mods | `KC.LM(1, KC.LGUI)` | `LM(1, MOD_LGUI)` | custom |
| `layerTap` | layer, tap | `KC.LT(1, KC.TAB)` | `LT(1, KC_TAB)` | `&lt 1 TAB` |
| `holdTap` | tap, hold, flags | `KC.HT(...)` | `MT` / generic | `&mt` / `&ht` |
| `oneShot` | inner | `KC.OS(...)` | `OSM`/`OSL` | `&sk` |
| `tapDance` | ref | `KC.TD(...)` or ref | `TD(n)` | `&td` |
| `macro` | ref | `KC.MACRO(...)` / ref | `MACRO` / `QK_MACRO` | `&macro` |
| `rgb` | op | `KC.RGB_TOG` | `RGB_TOG` | `&rgb_ug` |
| `mouse` | op | `KC.MS_UP` | `KC_MS_UP` | `&mmv` / `&mkp` |
| `media` | op | `KC.MPLY` | `KC_MPLY` | `&kp C_PLAY` |
| `workshop` | op | `customkeys.ToggleDrive` | custom | custom |
| `user` | name | `customkeys.X` | `USER00` | custom |
| `raw` | text | as-is | as-is | as-is |

`raw` is the escape hatch (Vial Any). It must round-trip. The parser should *promote* raw to structured when it recognizes a pattern.

Refs (`macro`, `tapDance`) point at BehaviorLibrary entries so editing a macro updates all keys.

---

## 6. Layout

KLE-compatible geometry plus bindings:

```json
{
  "keys": [
    {
      "id": "01HZ...",
      "x": 0, "y": 0, "w": 1, "h": 1,
      "r": 0, "rx": 0, "ry": 0,
      "matrix": [0, 1],
      "directIndex": null,
      "coordIndex": 1,
      "encoderId": null,
      "variant": [0, 0]
    }
  ],
  "variants": [
    { "name": "ISO", "options": ["ANSI enter", "ISO enter"], "selected": 0 }
  ]
}
```

`coordIndex` is the keymap slot (Pog `idx`). `matrix` is documentation + detection. They can disagree during setup; the app must show the disagreement (FR-S02).

---

## 7. Keymap and layers

```json
{
  "layers": [
    {
      "id": "01J0...",
      "name": "Base",
      "color": "#88aa00",
      "bindings": {
        "01HZ...": { "kind": "hid", "code": "A" }
      },
      "encoders": {
        "01HY...": {
          "cw": { "kind": "media", "op": "VOLU" },
          "ccw": { "kind": "media", "op": "VOLD" },
          "click": { "kind": "media", "op": "MUTE" }
        }
      }
    }
  ]
}
```

Bindings keyed by **LayoutKey id**, not array position. Emitters flatten using `coordIndex` for KMK.

Empty binding = transparent for layers > 0, `none` for layer 0 (configurable).

---

## 8. Behavior library

```json
{
  "macros": [
    {
      "id": "01J1...",
      "name": "Email",
      "actions": [
        { "type": "tap", "behavior": { "kind": "hid", "code": "H" } },
        { "type": "delay", "ms": 20 },
        { "type": "text", "string": "ello" }
      ]
    }
  ],
  "combos": [
    {
      "id": "01J2...",
      "name": "Esc",
      "style": "chord",
      "keys": ["01HZ...", "01HZ2..."],
      "result": { "kind": "hid", "code": "ESC" },
      "timeoutMs": 50,
      "matchCoord": false,
      "perKeyTimeout": false,
      "fastReset": true
    }
  ],
  "tapDances": [
    {
      "id": "01J3...",
      "name": "Q/Esc",
      "taps": [
        { "kind": "hid", "code": "Q" },
        { "kind": "hid", "code": "ESC" }
      ],
      "hold": { "kind": "layerMo", "layer": 1 },
      "tapTimeMs": 200
    }
  ]
}
```

Pog `combos[]` on the root JSON maps here on import.

---

## 9. Runtime settings

```json
{
  "tapTimeMs": 200,
  "comboTermMs": 50,
  "oneShotMs": 1000,
  "autoshift": { "enabled": false, "tapTimeMs": 300 },
  "mouse": { "intervalMs": 16, "maxSpeed": 10 },
  "unlock": { "required": true, "comboKeyIds": ["...", "..."] },
  "flashingMode": "automatic"
}
```

---

## 10. MCU catalog

`mcu-catalog.toml` / json, shipped in the app, updatable:

```json
{
  "id": "rpi-pico",
  "name": "Raspberry Pi Pico",
  "mcu": "RP2040",
  "circuitPythonUf2": { "url": "...", "sha256": "..." },
  "pins": [
    { "name": "GP0", "gpio": 0, "uart": "UART0_TX", "reserved": false },
    { "name": "GP25", "gpio": 25, "led": true },
    { "name": "GP23", "reservedReason": "PSRAM/onboard on some boards" }
  ],
  "forbidInMatrix": ["GP24", "GP25", "VBUS_SENSE"],
  "image": "pico.svg"
}
```

Pog’s two-board JSON is the seed, not the catalog.

---

## 11. On-disk project

```
~/Library/Application Support/workshop/library/<boardId>/
  project.json          // pointers + versions
  board.json
  layout.json
  wiring.json
  keymap.json
  behaviors.json
  settings.json
  pog-compat.json       // last exported 2.x view
  backups/              // timestamped zips
  assets/               // photos, pin overlays
```

Alternatively one `project.json` with all sections. Split files make diffs and share (“just the keymap”) easier.

Device filesystem (v1 runtime):

```
/board.json             // wiring + board + settings (needed to boot)
/keymap.json            // layers + library
/code.py                // stub
/lib/kmk/
/lib/workshop/
/overlays/customkeys.py
/kmk/version            // or lib/kmk/VERSION
```

Keep writing a synthesized `pog.json` next to these for one major version so old Pog and old firmware still make sense.

---

## 12. `.kbpack` v1

ZIP, store compression, no zip-slip:

```
manifest.json           // schema, board id, hashes, createdAt, app version
board.json
layout.json
wiring.json
keymap.json
behaviors.json
settings.json
assets/*
optional: photos of wiring
optional: lock.sig      // later
```

`manifest.json` lists SHA-256 of each file. Importer verifies, then stages, then prompts.

---

## 13. Pog 2.x mapping

| Pog field | Destination |
|---|---|
| id, name, manufacturer, description, tags, controller | Board |
| keys[] | Layout (generate ids if missing) |
| layouts[] | Layout.variants |
| wiringMethod, rows, cols, pins, rowPins, colPins, directPins, diodeDirection, pinPrefix | Wiring |
| keyboardType, split* | Wiring.split (`splitBLE`/`splitBle` both accepted) |
| encoders, encoderKeymap | Wiring.encoders + Keymap.encoders |
| keymap[][] | Keymap via coordIndex |
| layers[] | Keymap.layers metadata |
| rgb* | Wiring.rgb + Lighting + settings |
| kbFeatures | hints to enable modules; capabilities win after first connect |
| flashingMode | settings |
| combos[] | BehaviorLibrary.combos |
| lastEdited | Board.updatedAt |
| everything else | `extensions.pog["field"]` |

Keycode strings parse through a KMK parser into IR; on failure store `raw`.

---

## 14. Validation rules (always on)

1. Unique pins across rows, cols, direct, encoders, RGB, split, OLED.
2. Pins exist on MCU and are not forbidden for that role.
3. `coordIndex` range = physical key count (split-aware).
4. Layer refs in behaviors < layer count.
5. Combo members exist.
6. Encoder ids in keymap exist in wiring.
7. Variant indices in range.
8. Schema version known or rejected with upgrade path.

The editor may be temporarily invalid (user is typing). **Write to device** may not.
