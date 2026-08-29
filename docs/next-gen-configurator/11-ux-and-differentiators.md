# 11 — UX, journeys, and differentiating features

Incumbents have trained users. VIA taught “click key, click code, done.” Vial taught “combos and tap dance are tabs.” Remap taught “catalog and practice.” Oryx taught “pretty and guided.” Pog taught “press your unfinished keyboard and it becomes real.”

The successor should feel like VIA on Tuesday and like a hardware lab on Saturday.

---

## 1. Information architecture

Three modes, one window, explicit switch:

| Mode | Who | Home |
|---|---|---|
| **Library** | Everyone | Boards, connected now, import |
| **Workshop** | Builder | Wizard + hardware editors (wiring, layout, flash) |
| **Live** | Typist | Keymap, behaviors, tester, lighting, settings |

Pog mixes these in one sidebar (Keymap next to Pins next to Firmware). That scares typists and hides the wizard from builders.

**Rule:** Live never shows pin text fields. Workshop never pretends a pin change is a live remap (FR-L05).

Secondary surfaces: Tester (can dock), Console (developer), Diff (before write).

---

## 2. Journeys

### J1 — New handwire (north-star)

1. Library → New board → “I have a blank controller”
2. Plug Pico. App sees `RPI-RP2` **or** `CIRCUITPY`. If bootloader, one button: “Install CircuitPython (cached).”
3. Pick MCU from catalog (pinout visible).
4. Interview: matrix vs direct; split?; encoders?; RGB?
5. “Enter detect.” LEDs blink. Copy: *Press each switch once. Esc undoes.*
6. Live matrix grows. Illegal pins already excluded.
7. “Press keys left-to-right, top-to-bottom” for order. Or “generate 4×12 ortho and I’ll edit.”
8. Optional: import KLE and *bind by pressing* (E14).
9. Review FilePlan (runtime + board.json). Confirm.
10. Tester: type. Done. Toggle “Hide USB drive.” Remap still works.

Time target: 15 steps, NFR-U1.

### J2 — Open last keyboard and change a key

1. Launch. Last board is selected if that USB serial is present (including drive-hidden).
2. Live → click key → pick `Escape` → instant.
3. Optional tester flash.

Three clicks, NFR-U2.

### J3 — Import Pog 2.2 project

1. Open folder / drop `pog.json`.
2. Migration report: “3 raw keycodes could not be parsed; stored as raw.”
3. Offer runtime upgrade.

### J4 — Home-row mods without reading QMK docs

1. Live → Magic → “Home row mods”
2. Choose GACS or CAGS, timing 200–250 ms
3. Preview on ASDF / JKL;
4. Apply hold-taps + recommended settings (Vial 2026 recipe translated to KMK names)
5. Tester + “too many mods? increase tap time” hint

### J5 — Split

1. Workshop knows two halves
2. Banner: “Plug **left** now”
3. Flash left → “Plug **right**”
4. One keymap; tester events from the USB half only, with a note

### J6 — Something broke

1. Safe mode / no HID
2. App sees CIRCUITPY or bootloader
3. “Restore last good keymap” / “Reinstall runtime (keep overlays)”
4. Diff of what was on the board vs last backup

---

## 3. Editor UX (steal with attribution)

| Pattern | Steal from | Our twist |
|---|---|---|
| Click key, click code | VIA | Nested builder chips on the keycap (tap | hold) |
| Tabs: keymap, TD, combo, macro, settings | Vial | Hide tabs the capability lacks |
| Key tester | VIA | Also works during detect |
| Macro recorder | VIA | Record from the *board* (tester events), not only host |
| Compare changes | Remap | Also vs device hashes |
| KLE geometry | KLE / Pog | Bind-by-press |
| Unlock | Vial / ZMK | Visible padlock, OLED/LED on device |
| Catalog of boards | Remap | Local `.kbpack` first, cloud later |
| Typing practice | Remap 2026 | Uses live HID from *your* board |
| Colormap | Chrysalis | Only if RGB map exists |
| Pinout click | KiCad / MCU datasheets / rare tools | First-class workshop |
| FilePlan review | `git add -p` | Non-negotiable before generate |

### Keycap anatomy

```
┌─────────────┐
│ HT          │  small kind tag if not hid
│ A     Ctrl  │  tap left, hold right
│ 3,5     #17 │  matrix / coord in tester mode
└─────────────┘
```

Pog shows a keycode string. Vial shows a short label. We show **structure**.

### Behavior builder

Not a text box with templates that paste `KC.TD(KC.A, KC.B)`.

- Palette of kinds
- Slot widgets (tap, hold, layer index stepper)
- Live canonical preview (`KC.HT(KC.A, KC.LCTRL, …)`) for the raw-curious
- Invalid state explained (“layer 4 does not exist”)

---

## 4. Copy and fear reduction

DIY firmware UIs fail on tone.

| Situation | Bad (Pog-ish) | Better |
|---|---|---|
| Drive hidden | “USB Drive Disconnected” error red | “Drive hidden — live mode on. Show drive.” |
| Serial only | “Read Only Serial” | “Connected over cable (no files). Keymap edits work.” |
| Update Pog files | Wall of overwrite warning | FilePlan with checkboxes, overlays unchecked |
| Detection | “Mapping your Pinout” | “Press every switch once. We’ll find the wires.” |
| Raw keycode | empty disabled input | Always-available, validates |
| Quit while serial open | hang | never |

German/English: ship EN, prepare DE. Host-locale legends are a different switch (“show DE keycaps”).

---

## 5. Differentiating features (the bets)

These are expanded from the executive summary. Each has a *why incumbents cannot easily copy* and a *v1 slice*.

### D1 — Press-to-map workshop

**Why we win:** VIA/Vial assume a definition. QMK Configurator assumes an in-tree keyboard. Remap assumes a catalog entry. We *create* the definition from electricity.

**v1 slice:** Pog’s detector + visual matrix + undo + skip pins + generate ortho + bind-by-press to KLE.

**v2:** Wiring diagram (schematic-ish), photo overlay, QR of pin list.

### D2 — Capability-negotiated UI

**Why we win:** Pog shows editors that lie. Vial approximates via compile-time feature bits. We query a live document and hide the rest.

**v1 slice:** Tabs and builders appear only if `features` and `limits` allow. Enable-OLED is a *Generate* action that changes capabilities after reboot.

### D3 — One IR, many firmwares

**Why we win:** Everyone else is a QMK app or a ZMK app or a KMK pile of strings.

**v1 slice:** IR + KMK printer/parser + QMK `keymap.json` / `info.json` **export** (no compile).

**Later:** VIA HID client; ZMK `.keymap` export.

### D4 — Honest live vs generate

**Why we win:** VIA pretends everything is live (until you want tap dance on VIA). Pog pretends everything is a file copy. We label the button.

**v1 slice:** Yellow “Will reflash runtime” banner on pin/module changes; green “Live” on keymap.

### D5 — `.kbpack`

**Why we win:** Remap share needs an account. Pog wanted “share pog.json” and QR. A signed zip with hashes is enough for forums, shops, and email.

**v1 slice:** Export/import + overlay review. No server.

### D6 — Host-aware legends + Unicode helper

**Why we win:** US-ANSI-only keycaps on a German board is a support tax. Vial Unicode is a macro essay. Pog listed language switcher as wishlist.

**v1 slice:** Locale pack for legends; Unicode character → OS-specific macro actions (WinCompose / mac hex / Linux Ctrl-Shift-U).

### D7 — Split as one object

**Why we win:** Every tool is bad at this. ZMK is closest (left central). Pog has fields but one path.

**v1 slice:** Two-half device model, sequenced flashing, shared keymap.

### D8 — Firmware diff / “why did this break?”

**Why we win:** Pog wishlist; nobody does it well. Support becomes a screenshot of a diff.

**v1 slice:** Hash compare + file list + “device newer / local newer.”

### D9 — HRM wizard

**Why we win:** Even Vial needs a blog post (Getreuer 2026). Shipping the recipe is product.

**v1 slice:** One modal, applies hold-taps + settings.

### D10 — Practice on the live board

**Why we win:** Remap just added typing practice in the *browser* against a virtual keymap. We have the real HID stream.

**v1 slice can wait (P2).** Design tester events so practice is a consumer.

### D11 — Mobile live-remap

**Why we win:** Nobody serious does KMK on Android. CMP makes this cheap *if* we chose CMP, or we ship WebHID for HID-only boards.

**Not v1** unless CMP was chosen for this reason.

### D12 — Vendor kit mode

**Why we win:** Chrysalis bundles firmware; Remap has orgs.

**v2:** A `.kbpack` + locked wiring + open keymap. Shop sells a kit; user never sees pins.

---

## 6. Accessibility and input

- Keyboard-operable layout grid (arrows, shift-select)
- Tester usable without seeing color (shape + label)
- Do not rely on red/green only for drive state
- 44 px click targets on wizard actions
- Screen-reader labels on keycaps (`"Key Q, hold Ctrl, layer 0"`)

---

## 7. Visual language (guidance, not a brand)

- Workshop: utilitarian, pinout paper, monospace pins
- Live: VIA-calm, lots of keycaps, few settings
- Danger: FilePlan and unlock, not every serial log line
- No WalletConnect orange, no crypto chrome, no Electron default icon energy

---

## 8. Empty and error states (required designs)

| State | Content |
|---|---|
| No boards | New / import / plug a Pico |
| Board in library, unplugged | Last preview + “plug to live-edit” |
| Drive hidden | Live available |
| Locked | How to unlock (keys highlighted) |
| Capability older | Update runtime |
| Parse raw | Show which keys are raw |
| Detect none | Check diodes, skip list, try invert diode |
| Linux permission | udev snippet + copy |

---

## 9. What would make a reviewer say “this is better than Vial”

Vial users will not care about our pinout wizard. They will care if:

1. Live remap is as instant
2. Combos / TD / macros are as complete
3. Settings exist
4. Unlock is clear
5. **Plus** we have unlimited layers on RP2040, filesystem backups, drive hide, and a path from a bag of parts

The last sentence is the review. Everything in this folder exists to make that sentence true.
