# 08 — Architecture (stack-agnostic)

This is the shape of the system if the UI is Wails, Compose, or something else in three years.

---

## 1. Principles

1. **The domain does not import UI or OS toolkits.**
2. **Transports are replaceable.** HID, serial, MSC, and (later) BLE implement one `DeviceTransport`.
3. **Generation is a backend.** “KMK workshop runtime” is the first `FirmwareBackend`. QMK/ZMK are additional backends, not if-else soup in widgets.
4. **The device is the source of capabilities.** The project is the source of intent. Drift is a first-class UI state.
5. **One connection owner.** A state machine in the host process owns ports. Widgets request sessions.
6. **Writes are classified:** `Live` (keymap, RGB now, macros) vs `Generate` (pins, modules, boot).
7. **User overlays are sacred.**

---

## 2. Logical modules

```
┌──────────────────────────────────────────────────────────────┐
│ Presentation                                                 │
│  Library · Workshop wizard · Editors · Tester · Console      │
└─────────────────────────────┬────────────────────────────────┘
                              │ ports (use cases)
┌─────────────────────────────▼────────────────────────────────┐
│ Application                                                  │
│  ProjectService  DeviceService  FlashService  DetectService  │
│  LiveSession     ImportExport    UpdateService               │
└───────┬───────────────┬──────────────────┬───────────────────┘
        │               │                  │
┌───────▼──────┐ ┌──────▼──────┐ ┌─────────▼─────────┐
│ Domain       │ │ Transports  │ │ Backends          │
│ Board, IR,   │ │ MSC         │ │ KmkBackend        │
│ validate,    │ │ Serial      │ │ (QmkBackend)      │
│ migrate pog  │ │ Hid         │ │ (ZmkBackend)      │
└──────────────┘ │ Ble         │ └───────────────────┘
                 └─────────────┘
┌──────────────────────────────────────────────────────────────┐
│ Platform                                                     │
│  FS watch · USB hotplug · dialogs · secure storage · log     │
└──────────────────────────────────────────────────────────────┘
```

Presentation is the only layer that changes if you swap Wails for CMP.

---

## 3. Process model

### Recommended: two-tier desktop

| Process | Role |
|---|---|
| **Host** (Go binary *or* JVM) | Transports, USB, FS, generation, project IO, updates |
| **UI** (WebView *or* Compose) | Rendering, editing, wizards |

Do not put serial reads on a UI thread. Do not put FAT copies on a UI thread.

### Optional third: helper

A tiny native helper (Kotlin/Native or CGO) only if the host cannot list volumes or claim HID on a platform. Avoid it until proven necessary.

### Do not

- Electron main + renderer + preload (Pog). Three worlds, one serial port, no owner.
- “Pure” Kotlin/Native desktop GUI. Compose Desktop is JVM; fighting that is a research project.

---

## 4. Device connection state machine

```
          plug
 Idle ─────────► Enumerating
                    │
                    ├─ MSC+CDC ──► Ready
                    ├─ CDC only ─► ReadyHidden
                    ├─ UF2 vol ──► Bootloader
                    └─ fail ─────► Error
 Ready / ReadyHidden
    │ open live
    ▼
 Live ── unlock ──► LiveUnlocked
    │ detect
    ▼
 Detecting
    │ flash
    ▼
 Flashing ──► Ready
    │
    └─ unplug / error ──► Idle / Error
 Quit: any ── close handles ── Halt
```

Rules:

- Only `Flashing` and `Detecting` may write `code.py` besides explicit Generate.
- `Live` reads are allowed locked; writes need `LiveUnlocked` if `unlock.required`.
- `Quit` is a state, not `process.exit` from a callback (Pog bug).

---

## 5. Use cases (application services)

### ProjectService

- create, open, list, backup, restore, import pog/KLE/QMK, export kbpack/pog-compat
- validate before save

### DeviceService

- hotplug fan-in
- bind PhysicalDevice ↔ BoardProject (by USB serial = board id, or user match)
- open/close sessions

### DetectService

- push detection mode
- collect events
- propose Wiring + Layout
- commit to project

### FlashService

- plan file set (diff)
- copy runtime (hashed)
- write JSON
- UF2 drop
- progress / cancel

### LiveSession

- capability
- get/set keymap, library, settings, RGB
- subscribe tester events
- toggle drive, reset, DFU

### ImportExport / UpdateService

- artifact cache (CP UF2, KMK zip)
- hash verify
- app self-update

Each use case is callable from UI *and* CLI.

---

## 6. Transport interface

```
interface DeviceTransport {
  id: TransportId
  open(device: PhysicalDevice): Session
}

interface Session {
  capabilities(): Capabilities
  readFile?(path): Bytes          // MSC
  writeFile?(path, bytes, atomic)
  listFiles?()
  invoke(op: ProtocolOp): Result  // HID/serial/BLE
  subscribe(events: EventKind): Flow
  close()
}
```

MSC session is a `FileSession`. HID/serial implement `invoke`. A **CompositeSession** (typical CircuitPython) offers both.

Pog’s mistake was two independent serial opens (config + debug) plus MSC, unsynchronized. CompositeSession serializes access with a mutex and a priority (user cancel > flash > live write > logs).

---

## 7. Firmware backend interface

```
interface FirmwareBackend {
  id: "kmk" | "qmk" | "zmk"
  canLive(caps): Boolean
  generate(project): FilePlan          // files to write
  parseDevice(files | caps): ProjectHint
  printBehavior(ir): string
  parseBehavior(text): Behavior
}
```

`FilePlan` is a list of `{ path, bytes, mode: create|replace|skipIfExists, role: runtime|data|overlay }`.

FlashService executes FilePlans. UI shows them (FR-S01).

---

## 8. Event bus (inside host)

Internal events, not a global Vue event soup:

- `DeviceArrived` / `DeviceLeft`
- `SessionState`
- `DetectionEvent`
- `TesterEvent` (coord or matrix)
- `LiveApplied` / `LiveFailed`
- `FlashProgress`
- `ProjectDirty`

UI subscribes. Tests emit.

---

## 9. Persistence

- Library root configurable
- SQLite optional for index (search); source of truth remains JSON files
- Device snapshot cache in SQLite or sidecar is fine
- No cloud in v1

---

## 10. Firmware runtime architecture (on device)

```
code.py
  → workshop.main()
       load board.json + keymap.json
       build KMK keyboard from IR (no eval)
       register modules from capabilities plan
       start Protocol (HID + CDC)
       start Keyboard loop
```

Protocol thread/task:

- capability
- keymap get/set (single key or bulk)
- library get/set
- rgb set
- reset / dfu / drive
- detect mode (GPIO scan) as a *mode*, not a different `code.py`

Detection as a mode means FR-W08 is natural: exit detect → same runtime, new wiring written to `board.json`, reboot once.

---

## 11. Security architecture

```
Host                         Device
  Live write ──► HID ──► unlock gate ──► apply
                       │
                       └── locked: reject
```

Unlock: user holds two keys (stored in settings, compiled into runtime as coords). Timeout 5 minutes. LED/OLED indicates unlocked.

`.kbpack`: verify hashes; show overlay diff; never execute host-side scripts from a pack.

---

## 12. Testing strategy

| Layer | How |
|---|---|
| Domain | Pure unit tests, Pog fixtures |
| KMK parser/printer | Golden strings |
| Protocol | Golden frames, fuzz CRC |
| Transports | FakeSession + optional hardware CI |
| FlashService | temp dirs, atomicity, cancel |
| State machine | table tests |
| UI | a few journey tests; do not bind domain tests to UI |
| Runtime | CircuitPython unix port if viable; else one Pico in CI |

Pog has **no automated tests**. This is non-negotiable for a rewrite.

---

## 13. Logging and support

Structured log (JSON lines) in app-data:

- transport open/close
- protocol errors
- flash file list
- never keymaps by default (privacy); opt-in “include project” in diagnostics zip

FR-A5 diagnostics bundle: log tail + OS + device list + capability JSON + schema versions.

---

## 14. Versioning

| Artifact | Scheme |
|---|---|
| App | semver |
| `workshop.board` schema | integer, migrations in Domain |
| Protocol | integer, advertised in capabilities |
| Runtime Python | semver, must match protocol minor |
| KMK pin | git SHA + date |

App may talk to runtime N and N-1. Older: offer update. Newer: refuse writes.

---

## 15. Mapping onto nominated stacks

| Module | Wails 3 | Compose Multiplatform |
|---|---|---|
| Presentation | Vue/Svelte/React in WebView | Compose (desktop JVM; later Android) |
| Application + Domain | Go packages | Kotlin `commonMain` (or still Go via sidecar — don’t) |
| Transports | Go serial/hid/fs | JVM jSerialComm + hid4java + java.nio; expect/actual |
| Platform hotplug | Go + OS APIs | JVM + small native |
| Device runtime | unchanged Python | unchanged Python |
| Bindings | Wails generated TS | no IPC; in-process |

**The device runtime and the file formats do not change with the UI stack.** That is the point.
