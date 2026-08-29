# 10 — Protocols and hardware

How the host talks to the board, in enough detail to implement.

---

## 1. Physical interfaces on a typical CircuitPython keyboard

| Interface | Role today (Pog) | Role in successor |
|---|---|---|
| USB MSC (CIRCUITPY) | Primary read/write of all files | Generate + backup + overlay; optional when hidden |
| USB CDC A (lower path) | Often REPL | REPL / logs (developer) |
| USB CDC B (higher path) | Detection @115200; sometimes data | Workshop serial @115200 (fallback live) |
| USB HID keyboard | Typing | Typing |
| USB HID raw / vendor | Unused | **Flagship live protocol** |
| USB HID extra (mouse/consumer) | KMK features | Unchanged |
| BLE GATT | Unused (KMK can HID-over-BLE) | Later live + host connect |
| UF2 MSC (RPI-RP2) | Unused (manual) | CircuitPython / recovery drop |

Pog’s 9600-baud chunk protocol on an unidentified CDC port is the wrong flagship: slow, collision-prone with REPL, no OS driver story on phones, no VIA ecosystem overlap.

---

## 2. Transport selection

```
if hid_workshop_present && unlocked_or_read:
    use HID
elif cdc_workshop_present:
    use Serial v2
elif msc_mounted:
    use files only (no live tester)
elif uf2_volume:
    bootloader mode
else:
    disconnected
```

HID and Serial v2 speak the **same operations** (`ProtocolOp`). Only framing changes.

---

## 3. Capability document

First message after open (HID feature report or serial hello).

```json
{
  "protocol": 2,
  "runtime": { "name": "workshop", "version": "1.0.0" },
  "kmk": { "sha": "5a6669d1..." },
  "boardId": "01HX...",
  "hashes": { "board": "…", "keymap": "…" },
  "layers": { "count": 4, "max": 16 },
  "features": ["holdtap", "tapdance", "combos", "macros", "rgb", "split", "detect"],
  "limits": { "macroActions": 256, "combos": 64, "tapDances": 32 },
  "hardware": {
    "mcu": "RP2040",
    "driveMounted": false,
    "splitSide": "left",
    "encoders": 1,
    "rgbCount": 12
  },
  "unlock": { "required": true, "unlocked": false }
}
```

`protocol: 1` is Pog 9600 JSON chunks (read/adapt only).

---

## 4. HID framing (flagship)

Goals: work when MSC is disabled; work on WebHID later; feel like VIA.

Recommended shape (implementer may adjust constants after a spike):

- Usage page `0xFF60` (QMK raw HID convention) or a **dedicated registered page** if we do not want to collide with VIA devices. Prefer a distinct page/usage so VIA apps ignore us.
- Report size 32 bytes (compatible with QMK raw HID) or 64 if we control the descriptor.
- Report ID 0.

Frame:

```
0     opcode        uint8
1     flags         uint8   // bit0=more, bit1=resp, bit2=err
2..3  seq           uint16 LE
4..5  length        uint16 LE  // payload bytes in this report or total
6..31 payload
```

Long messages: `more` bit, same `seq`, reassemble. CRC32 in the last 4 bytes of the logical message.

Opcodes (v2 minimum):

| op | name | payload |
|---|---|---|
| 0x01 | CAPS | — / capability JSON (chunked) |
| 0x02 | UNLOCK | host nonce; device confirms after combo |
| 0x03 | LOCK | — |
| 0x10 | GET_KEY | layer, coord |
| 0x11 | SET_KEY | layer, coord, behavior IR (cbor/json) |
| 0x12 | GET_KEYMAP | — |
| 0x13 | SET_KEYMAP | full keymap (chunked) |
| 0x20 | GET_LIB | macros/combos/td |
| 0x21 | SET_LIB | |
| 0x30 | GET_SETTINGS | |
| 0x31 | SET_SETTINGS | |
| 0x40 | RGB_SET | mode, hsv, speed |
| 0x50 | RESET | |
| 0x51 | DFU | |
| 0x52 | DRIVE | 0 hide / 1 show / 2 toggle |
| 0x60 | DETECT_START | |
| 0x61 | DETECT_STOP | |
| 0x62 | DETECT_EVENT | device → host (pressed pair) |
| 0x70 | TESTER_EVENT | device → host (coord down/up) |
| 0x7E | LOG | device → host UTF-8 line |
| 0x7F | ERROR | code, message |

CBOR is preferable to JSON on 32-byte reports; JSON is easier to debug. Compromise: JSON for Serial v2 and first HID; switch to CBOR if profiling demands it. **Do not mix without a flag.**

Unlock: host sends UNLOCK; device LEDs blink; user holds combo within 10 s; device returns unlocked=true. All SET_* fail with `LOCKED` otherwise.

---

## 5. Serial v2 (fallback)

115200 8N1, CDC data port (not REPL).

Line discipline:

```
< W2 | {json}\n
> W2 | {json}\n
```

Or length-prefixed binary if JSON newlines in payloads hurt: `W2` + uint32be length + crc32 + bytes.

Must:

- Advertise protocol 2 in hello
- Same opcodes as HID
- CRC32
- Idle heartbeat every 2 s so the host detects unplug
- Coexist with REPL on the other CDC: **never** take the REPL port for protocol

### Serial v1 (Pog compatibility)

9600, newline JSON, commands `info`, `info_simple`, `save`, `saveKeymap`, `1`, `y`, `drive`, `reset`. Chunks 800/1200, cross-sum.

Host auto-detect: if first line is not `W2`, try Pog `info_simple`. Offer “upgrade runtime.” Writes of new IR should upgrade, not keep generating v1 `keymap.py` eval files, unless the user pins old firmware.

---

## 6. MSC / file protocol

Not a protocol so much as a contract.

| Path | Writer | Atomic |
|---|---|---|
| `board.json` | FlashService / Live flush | yes |
| `keymap.json` | LiveSession | yes |
| `lib/workshop/**` | FlashService generate | replace set |
| `lib/kmk/**` | FlashService, hashed | replace set |
| `code.py` | FlashService only if stub/auto | yes |
| `boot.py` | FlashService only on generate + confirm | yes |
| `overlays/**` | User / explicit import | never auto |
| `pog.json` | optional compat emit | yes |

Atomic on FAT12/16 (CIRCUITPY): write `name.tmp`, fsync, rename, delete old. CircuitPython may still see a partial file if power-cut mid-rename; keep `name.bak` from last good.

While Live writes `keymap.json`, do not also write it from MSC in another thread.

---

## 7. Detection protocol

Stay close to Pog’s device messages so the existing detector firmware can be a stepping stone:

```json
{"type":"start_detection","pins":["GP0", "..."]}
{"type":"new_key_press","row":"GP2","col":"GP8"}
{"type":"existing_key_press","row":"GP2","col":"GP8"}
{"type":"used_pins","rows":["GP2"],"cols":["GP8"]}
```

Improvements:

- Prefer HID `DETECT_EVENT` so baud/port issues vanish
- Include timestamp and raw pin bitmask for debug
- `skip_pins` from MCU catalog sent in `DETECT_START`
- `undo` opcode
- Diode-direction inference event
- Exit without rewriting `code.py`

Algorithm (device): for each candidate output pin, drive low, read others as inputs with pull-ups, record pairs. Skip USB, SWD, flash, LED unless user overrides.

---

## 8. Tester events

Device sends coord index + down/up. Host highlights LayoutKey with that `coordIndex`.

If coord map is wrong, tester still lights *something* — and that is how the user notices. Show matrix (r,c) in the status bar (VIA does this).

---

## 9. UF2

RP2040 bootloader volume: copy `firmware.uf2`.

The App should:

1. Cache official CircuitPython UF2 per MCU (URL + sha256 in catalog)
2. Optionally build a **recovery UF2** later (custom); not v1
3. Detect volume names: `RPI-RP2`, `RP2350`, board-specific

Do not implement every QMK Toolbox bootloader in v1.

---

## 10. Split hardware

Two PhysicalDevices, one Board.

| Half | MSC | CDC | HID |
|---|---|---|---|
| USB target (usually left) | maybe | yes | yes (host) |
| Peripheral | maybe (if plugged) | sometimes | no (UART/BLE to target) |

Live protocol talks to the **USB target**. Keymap is global. Detection may need each half plugged separately; UI must say which half is expected.

Pog stores `serialPortA/B` on one keyboard object. Successor stores:

```
halves: {
  left:  { usbSerial, lastPath, lastPorts[] },
  right: { usbSerial, lastPath, lastPorts[] }
}
```

Board.id stays one. USB product serial on the device should be `boardId-L` / `boardId-R` if CircuitPython allows; otherwise pair manually once and remember.

---

## 11. BLE (later)

Two different BLE jobs people conflate:

1. **Keyboard as HID host connection** (KMK BLE HID, ZMK to phone) — not a configurator transport
2. **Configurator transport** (ZMK Studio over BLE) — phone/desktop configures without USB

Job 2 needs a GATT service with the same `ProtocolOp`s. Do not start until HID/serial are boring.

---

## 12. Permissions (host OS)

| OS | Serial | HID | MSC |
|---|---|---|---|
| Windows | Usually free | WinUSB/HidD; may need no extra driver for HID | Free |
| macOS | User dialog; TCC | Entitlements; input monitoring is **not** the same as raw HID | Free |
| Linux | `dialout`/`uucp` | udev rule for usage page | Free; udisks |

Ship:

- `99-workshop.rules` sample
- in-app copy button
- first-run check (`groups`, test-open)

Vial and ZMK Studio already tax users here. A good first-run is a differentiator.

---

## 13. Concurrency and shutdown

Host session mutex (see `08`):

1. On unplug: cancel reads via close; do not wait forever
2. On quit: transition Halt, close, then return from main (no `os.Exit` from a read callback)
3. Read loops use deadlines
4. One writer at a time

Pog’s `before-quit` + `process.exit(0)` + `debugPort.close` is the anti-pattern.

---

## 14. Security

| Attack | Defense |
|---|---|
| Evil USB host rewrites macros to exfiltrate | Unlock combo; optional write-lock switch |
| Evil `.kbpack` | Hash + path check + overlay review |
| Evil serial on a shared PC | Unlock; don’t bind 0.0.0.0; local device only |
| Firmware download MITM | HTTPS + sha256 |
| REPL exposed | Not the protocol port; developer toggle |

VIA has been criticized for no unlock. Vial and ZMK Studio have unlock. **We follow Vial.**

---

## 15. Migration from Pog serial

| Pog | Successor |
|---|---|
| `info` chunked pog.json | CAPS + GET_KEYMAP |
| `save` chunked pog.json | SET_KEYMAP + optional board write |
| `saveKeymap` | SET_KEYMAP (unused in Pog UI) |
| `info_simple` | CAPS subset |
| `drive` | DRIVE |
| `reset` | RESET |
| Detection JSON on 115200 | DETECT_* |
| REPL debug | LOG + optional REPL pane |

Keep a `pogcompat` transport adapter for one major version.
