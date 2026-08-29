# 03 — Competitive landscape

This document is the feature-compliance baseline. “At least as good as X” is meaningless unless X is named and listed.

Tools are grouped by **job**, not by brand. A successor that copies VIA’s remap UI but cannot flash a Pico is not competing with VIA; it is abandoning Pog. A successor that only flashes Picos is not competing with Vial; it is staying a workshop.

---

## 1. Map of the market

```
                    Need a compiler?
                    no │ yes
                       │
 Live remap?  yes ─────┼─────────────────── QMK Configurator
                       │                    ZMK user config + GH Actions
                       │                    hand-written KMK
                       │
                  VIA  │
                  Vial │
                 Remap │
          ZMK Studio   │
            Chrysalis  │
           Pog serial  │
                       │
              no ──────┼─────────────────── QMK Toolbox (flash only)
                       │                    CircuitPython drag-drop
                       │                    Pog USB file write
                       │                    KLE (layout only)
```

Pog sits in the **bottom-left of “live remap”** (serial is live-ish) and the **bottom-right of “compiler”** (file copy is a kind of flash). It is the only common tool that also occupies **“I do not have a keyboard definition yet.”**

---

## 2. Tool-by-tool

### 2.1 QMK Firmware (not a GUI)

**What it is:** The default C firmware for custom mechanical keyboards. GPL-2.0. Huge keyboard tree. Host-side tools assume it.

**What it offers that users expect a configurator to *express*, even if the GUI cannot compile all of it:**

| Area | Capabilities |
|---|---|
| Layers | Up to 32; MO, TG, TO, TT, LT, LM, DF, PDF, layer_on/off in macros |
| Tap-hold | Mod-Tap, Layer-Tap, `TAPPING_TERM`, Permissive Hold, Hold On Other Key Press, Retro Tapping, Quick Tap, Chordal Hold, Tap Flow, per-key tapping term |
| One-shot | OSM, OSL, oneshot timeouts, oneshot mods |
| Tap dance | N actions per key (tap, hold, double, …), layer move/toggle helpers |
| Combos | Chords, per-combo term, must-hold, layer-independent combos, combo ref layers |
| Leader | Leader key sequences, per-key timing |
| Macros | DYNAMIC macros, old SEND_STRING, tap-hold in macros |
| Key overrides | Replace key+mod with another key |
| Caps Word / Autoshift / Autocorrect | Word-scoped caps, hold-to-shift, dictionary autocorrect |
| Repeat / Alt Repeat | Repeat last key, opposite-hand alt-repeat |
| Unicode | Several OS input modes |
| Swap Hands | Mirror layout |
| OS detection | Guess host OS from USB descriptors |
| Mouse keys | Cursor, buttons, wheel; extended reports |
| Pointing devices | Cirque, PMW3360, analog sticks, etc. |
| Encoders | Encoder map per layer; click as matrix key |
| RGB | RGBLight (underglow) and RGB Matrix (per-key), many effects, layer segments |
| Backlight | Mono LEDs, breathing |
| OLED / displays | Custom rendering |
| Audio, haptic, solenoid | Niche but real |
| Split | Serial, I2C, transport, EE_HANDS, RGB/OLED across halves |
| Bluetooth | Optional, vendor-specific, weaker than ZMK |
| MIDI, Steno, Joystick, Digitizer | Present |
| VIA/Vial protocol | Optional compiled-in raw HID |

**Configurator implication:** a “QMK-complete GUI” is not achievable (QMK Configurator itself refuses Tap Dance and Unicode). A “Vial-complete GUI” is the realistic QMK-facing bar.

### 2.2 QMK Configurator (config.qmk.fm)

| | |
|---|---|
| Job | Build a `.hex`/`.bin` from a web keymap for a **known** keyboard |
| Open source | Yes |
| Live remap | No — compile + flash every change |
| Keyboard source | In-tree QMK keyboards only |
| Layers | Many |
| Advanced behaviors | **No** Tap Dance, Unicode, or anything that needs `keymap.c` functions |
| Encoders | Yes (basic) |
| Macros | Via JSON, limited |
| Lighting | Limited to what the keyboard JSON exposes |
| Import/export | Keymap JSON, download firmware + source |
| Unique | No local toolchain required for supported boards |

**Compliance note:** do not treat QMK Configurator as the feature ceiling. It is a compiler front-end with a deliberately small keymap DSL.

### 2.3 VIA (usevia.app)

| | |
|---|---|
| Job | Live remap of QMK boards that shipped with VIA |
| License | App is open; historically treated as closed/process-heavy; definitions must be merged |
| Transport | WebHID raw HID (VIA protocol) |
| Flash required after first VIA firmware | No |
| Keyboard recognition | Official definition repo **or** sideload JSON |
| Layers | Typically **4** (EEPROM) |
| Keymap | Full basic + layers + lighting keycodes + custom |
| Macros | ~16, recorder + keycode text `{KC_...}`, stored on device |
| Lighting | Backlight + RGBLight/RGB Matrix via definition menus (range, toggle, dropdown, color) |
| Encoders | Historically weak / absent in older VIA; still not Vial-class |
| Tap dance / combos / key overrides | **No** (ANY key can type `LT(...)` as a hack) |
| Key tester | **Yes** — visual + optional sound, matrix position, keycode |
| Design tab | Edit/create definitions |
| Save/load | JSON keymap backup |
| Desktop | Legacy; web is the product |
| Unlock | None beyond HID access |

VIA sets the **UX standard** for: pick key → pick code → it works now. It also sets the **political standard** of a central definition repo. Pog’s “no PR required” stance is closer to Vial.

**V1 successor must match VIA on:** live remap feel, key tester, macros with recorder, lighting controls, save/load, definition sideload.

### 2.4 Vial (vial.rocks + desktop)

Vial is the **feature ceiling** for a live QMK configurator in 2026.

| | |
|---|---|
| Job | Live remap + advanced QMK features without a central PR |
| License | GPL-2.0 (vial-qmk fork + vial-gui) |
| Transport | WebHID / desktop HID; definition **embedded in firmware** |
| Layers | Default ~4, configurable up to 32 if firmware built for it |
| Keymap | VIA-like palette plus Quantum tab |
| Tap dance | First-class tab, EEPROM-backed entries |
| Combos | First-class tab |
| Key overrides | First-class tab |
| Macros | Dynamic, tap/down/up, count limited by EEPROM (default 16, tunable) |
| Encoders | Supported |
| MIDI | Tab |
| RGB / backlight | Full-er than VIA |
| QMK Settings | Grave Escape, tap-hold (Permissive Hold, Quick Tap, Chordal Hold, Tap Flow), Auto Shift, Combos term, One Shot, Mouse Keys, Magic Keys |
| Caps Word, Repeat, Alt Repeat, Layer Lock, Key Lock | Present in Quantum tab (April 2026 guide) |
| Home-row mods | Documented recipe using Mod-Tap + settings (not Tap Dance) |
| Custom keycodes | User tab, still need C `process_record_user` |
| Unlock | Unlock combo compiled into firmware (security against malicious HID) |
| Missing vs full QMK | Per-key tap-hold, Autocorrect, Leader, OS detection, Swap Hands, Unicode (macro hack only) |
| Memory | Heavier than VIA; AVR boards often cut features |

**V1 successor must match Vial on:** tap dance, combos, macros, key overrides *or an equivalent KMK feature*, encoder map, settings panel, unlock story, embedded definition so no sideload is required for first-party firmware.

KMK does not have QMK Key Overrides or Leader as first-class Pog features today. Matching Vial on those means **implementing them in the KMK runtime**, not only drawing a tab.

### 2.5 Remap (remap-keys.app)

A VIA-protocol web app with a **catalog and community** layer QMK Configurator never grew.

| Feature | Notes |
|---|---|
| WebHID | Same family as VIA |
| Keyboard catalog | Find kits by filters |
| Firmware flash from browser | Yes |
| Workbench | In-browser QMK project + **cloud compile** (account, paid extra builds as of 2025) |
| Easy assign of Hold/Tap | Marketed explicitly |
| Save/restore keymaps | Cloud + local |
| **Share keymaps** | Community |
| Lighting | Backlight + underglow |
| Macro editor | Yes |
| Compare changes | Diff UI for keymap |
| Test matrix | After build |
| Typing practice | Shipped January 2026 |
| Organizations | Shops can manage keyboards |

**Successor implication:** Remap is what happens when VIA grows a store and a social layer. Sharing, catalog, flash, and practice are the features users will compare against, even on KMK.

Pog’s Community screen is not a competitor to this. Do not ship Web3; ship Remap-like share if you ship share at all.

### 2.6 ZMK + ZMK Studio

ZMK is the wireless-first Zephyr firmware (nice!nano, nRF52). Studio is a 2024– MVP runtime configurator.

**Studio can (today):**

- Change keymaps live over USB
- Native apps: Windows, Linux, macOS
- BLE configuration from Linux web + native apps
- Assign predefined and user-defined behaviors
- Select alternate physical layouts already in the firmware
- Rename layers, enable extra reserved layers

**Studio cannot / will not:**

- Define new physical layouts
- Add more layers than firmware reserved
- Define new behaviors not in devicetree
- (Planned) combos, conditional layers, tap dance/macro property editors, encoder assign, host locale, import/export

Unlock via `&studio_unlock` on the keymap. Firmware must be built with `ZMK_STUDIO` and `studio-rpc-usb-uart`. RAM-heavy; some STM32F072 boards barely fit.

**ZMK behaviors a full configurator would need to *model* even if Studio cannot edit them yet:**

Key press, Transparent, None, Mod-Tap, Hold-Tap, Layer-Tap, Momentary layer, Toggle layer, Sticky keys, Tap dance, Macros, Mod-Morph, Combos, Conditional layers, Mouse button/move, Bluetooth profile ops, RGB underglow, Reset, Soft off, Studio unlock, Sensor rotation (encoders).

**Successor implication:** do not build a ZMK compiler in v1. Do design `Behavior` as a closed-ish catalog with parameters so a ZMK backend can map `&mt`, `&lt`, `&mo` later. Studio’s BLE path is the reference for “configure a split over wireless.”

### 2.7 Chrysalis (Keyboardio / Kaleidoscope)

Electron GUI for Kaleidoscope boards (Model 01, Atreus, etc.).

- Live layout editor, copy layer, set default layer
- Per-key colormap editor
- Bundled firmware images + custom firmware if FocusSerial + EEPROM plugins are present
- Serial “Focus” protocol (text commands), not VIA HID

**Useful ideas:** bundled known-good firmware artifacts; colormap as a first-class editor; protocol that is inspectable (Focus is line-based, like Pog serial but designed).

### 2.8 QMK Toolbox

Not a configurator. A **flasher + HID console**.

Bootloaders: ARM DFU, Atmel/LUFA/QMK DFU, SAM-BA, BootloadHID, Caterina (avrdude), HalfKay, LUFA HID, WB32 DFU, LUFA MSC.

Also: auto-detect bootloader, hid_listen-compatible console (`0xFF31` / `0x0074`).

**Successor implication:** if you ever target QMK, you will either shell out to these tools or re-implement a subset. For KMK/CircuitPython, the equivalent is: UF2 drop, CIRCUITPY copy, and `microcontroller.on_next_reset(UF2)`. Pog already has DFUMODE/SAFEMODE custom keys. A Toolbox-like **flash activity log** is still worth copying.

### 2.9 Keyboard Layout Editor (KLE) and successors

KLE (keyboard-layout-editor.com) is the lingua franca of *geometry*.

- JSON (sometimes JSON5) array format
- Key properties: legends (12 positions in the wild), colors, profile, `w/h/x/y`, rotation, decal, ghost, stepped, nub
- No matrix, no firmware, no keycodes in the original

Keyboard Layout Studio (2020s) adds: 1–16 layers of keycodes, QMK `keyboard.json` export, Vial `vial.json` export, KLE import, matrix row/col assign, undo/redo, 9-position legends.

**Successor implication:** KLE import is mandatory (Pog already has it). Export to KLE + QMK `info.json` / `keyboard.json` + Vial definition is how the workshop joins the rest of the ecosystem. A layout editor that cannot export is a trap.

### 2.10 KMK itself (as a “configurator”)

KMK’s configurator is **Python**. The docs assume you write `kb.py` and `main.py`/`code.py`.

Modules: Combos, Layers, HoldTap, Macros, Mouse Keys, SpaceMouse, Sticky Keys, Power, Split, SerialACE, TapDance, Dynamic Sequences, Mouse Jiggler, MIDI, plus hardware modules (Encoder, ADNS9800, trackball, EasyPoint).

Extensions: International, LED, LockStatus, MediaKeys, OLED, RGB, SpaceMouse Status, Status LED, Stringy Keymaps.

**Pog is the only serious GUI for this stack.** That is a moat if — and only if — the GUI covers the modules people actually enable. Today it covers Layers, a taste of HoldTap/TapDance/Macros, RGB, Encoder, Split, StickyKeys, MouseKeys. It does not cover OLED, pointing devices, MIDI, power, dynamic sequences, or BLE HID as a host connection.

### 2.11 Others worth knowing

| Tool | Why it matters |
|---|---|
| **Vial desktop** | Proof that a native app still matters for HID permissions and offline |
| **usevia.app Design tab** | In-app definition editor — Pog’s layout editor is the seed of this |
| **ZMK Keymap Editor** (third-party web) | People already edit `.keymap` visually without Studio |
| **KeymapDB / keyboards.fyi** | Community keymap archaeology — import target |
| **Oryx (ZSA)** | Polished proprietary workshop: lighting, layers, training, tenting — UX to beat |
| **Wootility, G HUB, Synapse, iCUE** | Consumer RGB/macros. Users coming from gaming expect profiles, per-app switch, cloud. Do not copy the bloat; do copy “profiles.” |
| **Kaleidoscope Focus** | Text protocol design reference |
| **hid_listen / QMK console** | Debug story users already know |

---

## 3. Feature matrix (configurators only)

Legend: **Y** yes, **P** partial, **N** no, **—** not applicable.

| Feature | Pog | VIA | Vial | Remap | QMK Conf | ZMK Studio | Chrysalis |
|---|---|---|---|---|---|---|---|
| Open source | Y | P | Y | Y | Y | Y | Y |
| Offline desktop | Y | P | Y | N | N | Y | Y |
| Live remap | P | Y | Y | Y | N | Y | Y |
| No compile for keymap | Y | Y | Y | Y | N | Y | Y |
| Works without official PR | Y | N | Y | P | N | Y | Y |
| Bare MCU / no definition | Y | N | N | N | N | N | N |
| Matrix autodetection | Y | N | N | N | N | N | N |
| Generate firmware files | Y | N | N | P | Y | N | P |
| Flash UF2 / copy files | Y | N | N | Y | N | N | Y |
| HID protocol | N | Y | Y | Y | N | P | N |
| Serial protocol | Y | N | N | N | N | Y | Y |
| BLE configure | N | N | N | N | N | Y | N |
| Visual layout editor | Y | P | N | N | N | N | N |
| KLE import | Y | N | N | N | N | N | N |
| Sideload definition | — | Y | Y | Y | — | — | — |
| Embedded definition | P | N | Y | N | — | Y | Y |
| Layers unlimited / many | Y | N | P | N | Y | P | P |
| Layer names/colors | Y | N | P | P | N | Y | P |
| Key tester | N | Y | Y | Y | N | N | N |
| Macro recorder | N | Y | Y | Y | N | N | N |
| Macro delays / down-up | N | Y | Y | Y | P | N | P |
| Combos GUI | N | N | Y | N | N | N | N |
| Tap dance GUI | N | N | Y | N | N | N | N |
| Hold/mod/layer-tap builder | N | P | Y | Y | P | P | P |
| Key overrides | N | N | Y | N | N | N | N |
| One-shot | P | P | Y | P | P | P | P |
| Autoshift / Caps Word | P | N | Y | N | N | N | N |
| Encoders | P | N | Y | P | Y | N | N |
| RGB underglow | P | Y | Y | Y | P | P | P |
| Per-key RGB / colormap | N | P | Y | P | P | N | Y |
| Settings (tapping term etc.) | N | N | Y | N | N | N | P |
| Unlock / security | N | N | Y | N | — | Y | N |
| Save/load keymap file | P | Y | Y | Y | Y | N | Y |
| Share community keymaps | N | N | N | Y | N | N | N |
| Typing practice | N | N | N | Y | N | N | N |
| Split-aware UI | P | P | P | P | P | P | N |
| Debug console | P | N | N | N | N | N | N |
| MCU pin database | P | — | — | — | — | — | — |
| Host locale legends | N | N | N | N | N | Planned | N |

---

## 4. Compliance bars derived from this matrix

### Bar A — “Do not regress Pog”

Everything in the Pog **Y** column must remain **Y**. Especially: bare MCU, matrix autodetection, file generation, KLE, serial, drive hide, local desktop.

### Bar B — “Vial-complete for KMK”

Turn every Vial **Y** that KMK can support into **Y** in the successor. Where KMK cannot (key overrides, leader, QMK Magic), either implement in the successor runtime or show “requires QMK backend.”

Minimum Vial-complete list:

1. Live remap that feels instant
2. Key tester
3. Macro editor with tap/down/up + delay
4. Combo editor
5. Tap dance editor
6. Hold-tap / mod-tap / layer-tap visual builder
7. Encoder map
8. Lighting controls beyond a mode integer
9. Settings: tapping term, combo term, oneshot term, mousekey speed, autoshift
10. Save/load
11. Unlock or equivalent “I trust this host”
12. Capability-limited UI (do not offer 32 tap dances if firmware has 4)

### Bar C — “Remap-complete social” (not v1)

Catalog, share, compare, typing practice, cloud compile. Design the file format so these can land without a rewrite.

### Bar D — “Workshop-complete” (Pog’s chance to win)

1. MCU pinout click-assign with conflict detection
2. Wiring preview
3. Press-to-map that produces a usable layout, not just pin lists
4. Split dual-drive / dual-port as one object
5. Firmware vs model diff
6. `.kbpack` export (layout + keymap + wiring + photos)
7. QMK/Vial definition export from a Pog board (even if we do not compile QMK)

---

## 5. What we should *not* copy

| From | Do not copy | Why |
|---|---|---|
| VIA | Central merge-gate for every keyboard | Kills DIY boards |
| VIA | 4-layer mindset | KMK can do more; DIY 40% boards need more |
| Vial | EEPROM-as-capacity UX as the only model | CircuitPython has a filesystem; use it |
| Vial | GPL infection of the whole app if avoidable | App can stay MIT if protocol/firmware are separate |
| Remap | Paid cloud compile as a core loop | KMK should stay local |
| Remap | Account wall for basic remap | Desktop-first, offline-first |
| Pog | Web3 community | Irrelevant to the job |
| Oryx | Closed firmware | Users must own their board |
| Vendor RGB suites | Per-app profiles via kernel hooks | Out of scope; OS-level |
| QMK Configurator | “If it needs C, it does not exist” | That is why Vial won |

---

## 6. Personas vs tools (who we steal from)

| Persona | Today they use | Successor should steal |
|---|---|---|
| Handwire on a Pico | Pog, raw KMK | Pog workshop, better detection |
| Bought a VIA kit | VIA / Remap | Live remap feel, tester, macros |
| Power user 40% | Vial | Combos, tap dance, settings, HRM |
| Wireless ergo | ZMK Studio / text keymap | Behavior model, BLE later |
| Keyboard designer | KLE + QMK info.json | Layout editor + export |
| Shop / kit vendor | Remap orgs, VIA PR | `.kbpack` + catalog later |
| Gamer coming from iCUE | VIA | Profiles, lighting, low fear |
