# 01 — Executive summary

## The product bet

Build a **from-scratch keyboard workshop** that can:

1. Take a **bare microcontroller** (CircuitPython / UF2) and produce a working custom keyboard — Pog’s current job, which VIA/Vial/Remap cannot do.
2. Then behave like **Vial** for the rest of the keyboard’s life: live remap, macros, combos, tap dance, encoders, lighting, settings, save/load, no reflash for keymap changes.
3. Speak a **firmware-agnostic board + keymap model**, with a first-class **KMK/CircuitPython backend** and designed-in backends for **QMK/Vial** and later **ZMK**.

If the successor only rebuilds Pog in a new UI toolkit, it remains a niche DIY flasher. If it only rebuilds Vial, it abandons the only reason Pog exists. The product is the **union**, with a better protocol and a better workshop UX.

---

## What Pog already proved

Pog is a working answer to a problem the QMK world mostly ignores:

> “I have a Pico, some switches, and a diode matrix. I do not have a QMK keyboard JSON, a merged PR, or a compiler toolchain. Make it type.”

It already does, end to end:

- Discover CIRCUITPY drives and Pog-identified serial ports
- Flash detection firmware and **infer the matrix by pressing keys**
- Edit a visual layout (KLE-like geometry, variants, rotation, ISO)
- Assign pins, diode direction, split type, encoders, RGB
- Generate `pog.json` + a set of Python files + a pinned KMK tree
- Push config over USB files *or* a chunked 9600-baud serial protocol
- Hide the USB drive via NVM (`ToggleDrive`) and still talk serial
- Keep a local history of boards with backups

That is a real product. It is also unfinished as a *configurator* relative to 2026 user expectations set by Vial and Remap.

---

## Where Pog loses

Against Vial / VIA / Remap, Pog is missing the features people now treat as table stakes:

| Table-stakes feature | Pog today |
|---|---|
| Instant keymap write, no file copy | Partial (serial save exists, flaky, not the default path) |
| Visual key tester with matrix highlight | Missing |
| Real macro editor (record, delays, tap/down/up) | Stub (2–4 key Press/Release only) |
| Combos / sequences GUI | Firmware schema only |
| Hold-tap / mod-tap / layer-tap builder | Templates + raw text |
| Tap dance editor | Template string only |
| Key overrides | Missing |
| Encoder click, per-layer encoder UI that feels finished | Pins + CW/CCW only |
| Lighting that is more than “pick a pin and a mode int” | Minimal |
| HID live protocol (works when CIRCUITPY is hidden, no baud, no chunks) | Missing |
| Community keymap sharing that is not a Web3 stub | Missing |
| Host locale / label language | Missing |
| Board catalog (“this is a Corne / a Lily58 / a Sweep”) | Missing — every board is a custom project |
| QMK / ZMK as targets | Missing |
| Automatic serial-port pairing by serial number | Partial |
| Controller pin validation against a real MCU database | Stub |

Pog’s unique strengths (bare-MCU setup, matrix autodetection, Python generation, drive-hide workflow) are **not** present in VIA/Vial. Those must be kept and made reliable.

---

## Competitive position in one sentence

**VIA/Vial/Remap configure known keyboards. Pog creates unknown ones. The successor should create unknown ones and then configure them as if they had always been known.**

---

## Recommended product shape

Four user-visible products, one core:

```
┌─────────────────────────────────────────────────────────┐
│                     Successor core                       │
│  Board definition · Keymap · Behaviors · Layout · I/O    │
└─────────────┬───────────────┬───────────────┬────────────┘
              │               │               │
     ┌────────▼─────┐ ┌───────▼──────┐ ┌─────▼──────┐
     │ Workshop app │ │ Live session │ │ Share hub  │
     │ (desktop)    │ │ (HID/serial) │ │ (optional) │
     └──────────────┘ └──────────────┘ └────────────┘
              │
     ┌────────▼────────┐
     │ Device firmware │
     │ KMK first, then │
     │ QMK / ZMK       │
     └─────────────────┘
```

- **Workshop app (desktop, required):** the Pog replacement. Hardware access, flashing, matrix detection, layout editor, firmware generation.
- **Live session (desktop first, web later):** Vial-class remapping over a proper protocol.
- **Share hub (later):** keymap/board sharing without Web3. Git-backed or a simple catalog API.
- **Device firmware:** a maintained KMK “runtime” that exposes capabilities, not a pile of generated Python the user is afraid to touch.

---

## Stack recommendation (short)

Full comparison: [09 — Stack comparison](09-stack-compose-vs-wails.md).

| Question | Answer |
|---|---|
| Should desktop UI be Kotlin/Native? | **No.** Compose Desktop is JVM. Kotlin/Native is for iOS and optional native helpers, not the desktop shell. |
| Is Compose Multiplatform viable? | **Yes**, and it is the better *long-term* bet if Android/iOS companions and a native-feeling editor matter. |
| Is Wails 3 viable? | **Yes**, and it is the faster *desktop-first* bet if the team wants to keep a web UI and lean on Go for serial/HID/USB. |
| What this study recommends | **Wails 3 for v1 (desktop KMK workshop + live protocol)**, with a **strict core in Go that is UI-agnostic**, so a Compose shell can be added later *if* mobile becomes real. If the team is Kotlin-native and mobile is in v1 scope, invert that: **CMP desktop + later Android**, with a thin native I/O layer. |

The architecture in `08` is written so either shell can sit on the same domain core. Do not let the UI framework own the keyboard model.

**Do not start from Electron again.** Pog’s current stack is the reason the app is heavy, awkward to native-module, and hard to reason about around serial lifetime. Both nominated stacks fix that.

---

## What “feature complete” means here

Three compliance bars, in order:

1. **Pog-complete** — every useful Pog capability preserved, including automatic setup, serial config, ToggleDrive, KLE import, split, encoders, RGB, KMK install. No regressions a current Pog user would feel.
2. **Vial-complete (KMK edition)** — live remap; layers with names; macros with delays and tap/down/up; combos; tap dance; hold-tap/mod-tap/layer-tap builders; one-shot; autoshift; capsword; encoder map; lighting; settings panel; save/load; key tester; unlock story.
3. **Workshop-complete** — matrix autodetection that is trustworthy; MCU pin database; wiring preview; firmware capability negotiation; backup/restore; flashing that does not brick; diagnostics.

QMK/ZMK backends are a **second product line**, not a v1 gate, unless the owner explicitly wants a multi-firmware company on day one. The *model* must not block them.

---

## The features that would actually stand out

These are not “also have combos.” They are bets the incumbents are structurally bad at. Detail in [11](11-ux-and-differentiators.md).

1. **Press-to-map workshop** — detect matrix, assign coord map, and generate a layout by *using the unfinished keyboard*, with a live wiring diagram.
2. **Capability-negotiated UI** — the app asks the firmware what it can do (layers, combos, tap dance slots, RGB, split, BLE) and only shows editors that will work. Vial approximates this; Pog does not.
3. **One model, many firmwares** — a board file that can emit KMK today and QMK `info.json` / ZMK physical layout tomorrow.
4. **Safe live + generation hybrid** — keymap changes go live over HID/serial; structural changes (pins, MCU, split) regenerate firmware and say so honestly.
5. **Host-aware labels** — German/Nordic/JIS legends on the keymap, not just US QWERTY, plus OS detection when the firmware supports it.
6. **Offline-first share format** — a single signed `.kbpack` (layout + keymap + wiring + photos + firmware pin) that can be mailed, QR’d, or published. Pog’s README already wanted this.
7. **Split as a first-class object** — two halves, two ports, two drives, one keyboard, with a visual “which half am I talking to?”
8. **Rehearsal / typing practice on *your* keymap** — Remap just shipped this in 2026; do it better by using the live board as the input device.
9. **Firmware diff and “why did this break?”** — show `pog.json` vs generated files vs what is on the device. Pog’s wishlist already named this.
10. **Mobile companion for live remap only** — Android (CMP) or a WebHID page for keymap tweaks; workshop stays desktop.

---

## Risks that will kill the project if ignored

1. **Treating KMK Python generation as the product.** Generated `kb.py` / `code.py` will always drift. The product is the *model + a stable device runtime*.
2. **Inventing a worse VIA protocol.** HID raw reports exist. Serial chunk-and-checksum at 9600 baud is a Pog-era workaround. Keep serial as a fallback, not the flagship.
3. **Boiling the ocean (QMK + ZMK + KMK + BLE + mobile in v1).** Ship KMK workshop + live remap. Leave sockets for the rest.
4. **UI framework owning hardware.** Serial lifetime, drive mounts, and flashing must live in a native service with tests, not in Vue/Compose widgets.
5. **Wails 3 beta risk vs CMP JVM size risk.** Pick one consciously; both are manageable; neither is free.
6. **Security of SerialACE-class features.** KMK’s SerialACE is arbitrary code execution. The successor must never expose “run Python on the keyboard” as a casual button.

---

## Immediate decisions the owner needs to make

These unblock architecture. They are not engineering preferences.

1. **Firmware v1 target:** KMK-only, or KMK + read-only QMK/Vial, or full multi-firmware?
2. **Live protocol v1:** HID (new firmware module) vs improved serial vs both?
3. **UI stack:** Wails 3 (recommended for desktop-first) or Compose Multiplatform (recommended if Android is in-scope for v1)?
4. **Compatibility with existing `pog.json`:** must import 100% of current files, or best-effort?
5. **Cloud/share:** local-only for v1, or a catalog from day one?
6. **License:** MIT like Pog, or GPL if a QMK-derived protocol/firmware fork is involved?

Recommended defaults if the owner does not want to decide yet: **KMK-only v1, HID+serial, Wails 3, 100% `pog.json` import, local-only share via `.kbpack`, MIT for the app and a clearly separated firmware runtime license.**
