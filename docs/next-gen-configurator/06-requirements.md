# 06 — Requirements

Formal requirements. IDs are stable; feature catalog IDs in `05` map into these.

Priority: **M**ust / **S**hould / **C**ould / **W**on’t (MoSCoW) for v1.

---

## 1. Actors

| Actor | Description |
|---|---|
| **Builder** | Creates a keyboard from a bare MCU |
| **Typist** | Remaps an already-workshopped board |
| **Vendor** | Ships kits; wants a `.kbpack` and a pinned runtime |
| **App** | The successor desktop process |
| **Device** | MCU running CircuitPython + workshop runtime (v1) |
| **Library** | Local on-disk project store |

---

## 2. Functional requirements

### 2.1 Projects and interchange

| ID | Req | v1 |
|---|---|---|
| FR-P01 | The App shall persist each board as a directory in the user app-data tree, not solely in web localStorage. | M |
| FR-P02 | The App shall import any `pog.json` produced by Pog 2.x into the domain model without data loss of documented fields. Unknown fields shall be retained in an `extensions` bag. | M |
| FR-P03 | The App shall export a Pog 2.x-compatible `pog.json` so a user can return to Pog if needed. | S |
| FR-P04 | The App shall import KLE JSON (including JSON5). | M |
| FR-P05 | The App shall import QMK `info.json` / `keyboard.json` layouts. | S |
| FR-P06 | The App shall export `.kbpack` (documented zip layout, see `07`). | S |
| FR-P07 | The App shall keep at least 20 automatic backups per board and allow restore. | M |
| FR-P08 | Restore shall offer “keep id” vs “new id” (Pog behavior). | M |

### 2.2 Discovery and transport

| ID | Req | v1 |
|---|---|---|
| FR-D01 | The App shall enumerate mounted volumes that look like CircuitPython, UF2 bootloaders, or user-labeled keyboard drives. | M |
| FR-D02 | The App shall enumerate serial ports with USB metadata (VID, PID, serial, manufacturer, product). | M |
| FR-D03 | The App shall group interfaces that share a USB serial number into one PhysicalDevice. | M |
| FR-D04 | The App shall treat “MSC unmounted, CDC present” as a valid connected state. | M |
| FR-D05 | The App shall subscribe to plug/unplug (or poll ≤ 1s) and update the UI without a mandatory refresh click. | M |
| FR-D06 | The App shall not open a port that another session in the same process already owns, except via an explicit steal with warning. | M |
| FR-D07 | On quit, the App shall close ports and then exit; it shall not hang. | M |
| FR-D08 | Linux first-run shall document `dialout`/`uucp` and udev. macOS shall document serial and files permission prompts. | M |

### 2.3 Workshop (bare MCU)

| ID | Req | v1 |
|---|---|---|
| FR-W01 | The App shall guide CircuitPython installation, including dropping a cached UF2 onto a bootloader volume when the MCU is known. | M |
| FR-W02 | The App shall offer matrix autodetection that records unique key positions from user presses and produces rowPins, colPins, and diodeDirection. | M |
| FR-W03 | The user shall be able to undo the last detected key and to mark pins unused. | M |
| FR-W04 | The App shall capture keymap order by prompting presses in reading order (coord map) and show the resulting indices on the layout. | M |
| FR-W05 | The App shall generate a default physical layout from matrix size when no KLE is provided. | M |
| FR-W06 | The App shall install a hashed KMK + workshop runtime onto the device filesystem. | M |
| FR-W07 | The App shall refuse to overwrite `overlays/` and user-marked files without an explicit destructive confirm. | M |
| FR-W08 | Detection shall not permanently replace the user’s keyboard firmware; leaving detection shall restore or write the workshop runtime. | M |

### 2.4 Editing

| ID | Req | v1 |
|---|---|---|
| FR-E01 | The App shall edit physical layout with KLE geometry, multi-select, snap, undo/redo. | M |
| FR-E02 | The App shall bind each layout key to a matrix coordinate, direct index, and/or encoder. | M |
| FR-E03 | The App shall edit an arbitrary number of named, colored layers. | M |
| FR-E04 | Assigning a behavior shall support catalog pick, nested builders (HT/MT/LT), and validated raw text. | M |
| FR-E05 | The App shall edit macros (tap/down/up/delay/text), combos (chord/sequence), and tap dances. | M |
| FR-E06 | The App shall edit hold-tap parameters and global tapping/combo/oneshot terms. | M |
| FR-E07 | The App shall edit encoder CW/CCW and click per layer. | M |
| FR-E08 | The App shall edit RGB pin, count, mode, HSV, speed. | M |
| FR-E09 | The App shall validate pins against the selected MCU catalog (existence + reservation). | M |
| FR-E10 | Split configuration shall match KMK Split constructor fields used by Pog today. | M |
| FR-E11 | The key tester shall highlight layout keys from live device events. | M |

### 2.5 Live session

| ID | Req | v1 |
|---|---|---|
| FR-L01 | The Device shall expose a capability document (see `04` §6). | M |
| FR-L02 | The App shall hide or disable editors the capability document does not allow. | M |
| FR-L03 | Keymap changes shall apply without copying the KMK tree again. | M |
| FR-L04 | When the drive is hidden, keymap changes shall still apply via HID or serial. | M |
| FR-L05 | Writes that change wiring, MCU, or enabled modules shall be labeled **firmware generation** and require confirm. | M |
| FR-L06 | An unlock gesture (key combo or physical button) shall be required before the first write of a session if the Device is configured to require it. Default for new boards: required. | M |
| FR-L07 | The App shall support both HID (preferred) and versioned serial (fallback). | M |
| FR-L08 | Serial protocol shall use a version header, CRC32, and ≥115200 baud. The old 9600 Pog protocol shall be auto-detected for import of existing boards. | M |

### 2.6 Safety and diagnostics

| ID | Req | v1 |
|---|---|---|
| FR-S01 | The App shall show a diff of files it is about to write. | M |
| FR-S02 | The App shall compare Device capability + `board.json` hash with the local project and show drift. | M |
| FR-S03 | The App shall provide a console for workshop logs; REPL interactive access shall be behind a “Developer” toggle. | M |
| FR-S04 | The App shall not offer SerialACE or free-form remote Python execution in the default UI. | M |
| FR-S05 | Downloads of runtime artifacts shall be HTTPS and hash-verified. | M |

### 2.7 Non-features (v1 Won’t)

| ID | Req | v1 |
|---|---|---|
| FR-X01 | The App shall not require a network account. | W (must not) |
| FR-X02 | The App shall not include cryptocurrency or wallet features. | W |
| FR-X03 | The App shall not compile QMK or ZMK. | W |
| FR-X04 | The App shall not be browser-only. | W |

---

## 3. Non-functional requirements

### 3.1 Performance

| ID | Req |
|---|---|
| NFR-P1 | Library load of 100 boards with 100-key layouts < 500 ms after first cache. |
| NFR-P2 | Key assignment feedback < 50 ms local; live write ACK < 200 ms typical HID. |
| NFR-P3 | Detection UI shall render each `new_key_press` < 50 ms after parse. |
| NFR-P4 | Copying KMK (~few thousand files) shall report progress at least every 100 ms and remain cancellable. |
| NFR-P5 | Idle CPU < 2% when connected and unfocused (no busy-wait serial reads). |

### 3.2 Reliability

| ID | Req |
|---|---|
| NFR-R1 | Unexpected device unplug shall never crash the App; it shall return to a reconnectable state. |
| NFR-R2 | FAT writes shall be atomic replace where the FS allows; on failure the previous `board.json` shall remain. |
| NFR-R3 | Automated tests shall cover domain serialize/import for all Pog 2.x fixture files in-repo or synthesized. |
| NFR-R4 | Protocol parsers shall have golden tests (chunking, CRC, old Pog 9600). |
| NFR-R5 | Quit with open ports shall succeed in < 2 s (NFR companion to FR-D07). |

### 3.3 Security

| ID | Req |
|---|---|
| NFR-S1 | Unlock required by default for writes. |
| NFR-S2 | No eval of device-supplied Python on the host. |
| NFR-S3 | `.kbpack` extract shall not follow zip slips; write only under a staging dir. |
| NFR-S4 | Auto-update shall verify signatures. |
| NFR-S5 | Developer REPL is off by default. |

### 3.4 Portability and packaging

| ID | Req |
|---|---|
| NFR-O1 | First-class: Windows 10/11 x64, macOS 13+ arm64, Ubuntu 22.04+ x64. |
| NFR-O2 | macOS x64 and Linux arm64 are Should. |
| NFR-O3 | Native modules/binaries for serial/HID shall be packaged per arch; no “npmRebuild: false and hope.” |
| NFR-O4 | App size target: < 40 MB compressed for Wails path; < 120 MB for CMP/JVM path. Either is acceptable if documented. |
| NFR-O5 | The App shall run offline except update-check and optional artifact download. |

### 3.5 Usability

| ID | Req |
|---|---|
| NFR-U1 | A new Builder shall go from Pico+CP to typing keys in ≤ 15 guided steps. |
| NFR-U2 | A Typist shall change one key in ≤ 3 clicks after the board is connected. |
| NFR-U3 | Destructive actions use confirm + type-to-confirm when overwriting firmware files. |
| NFR-U4 | All errors shall be actionable (“open udev guide”, “enter bootloader”, “unlock the keyboard”). |
| NFR-U5 | Keyboard-operable editor (arrow keys, enter to assign) for accessibility. |

### 3.6 Maintainability

| ID | Req |
|---|---|
| NFR-M1 | Domain model and transports shall have no dependency on the UI toolkit. |
| NFR-M2 | A second UI (web live-remap or CMP) shall be possible without rewriting protocol code. |
| NFR-M3 | Firmware runtime version is semver and listed in capabilities. |
| NFR-M4 | Public file formats (`board.json`, `.kbpack`) are versioned. |

---

## 4. Acceptance scenarios

### AS-1 Handwire happy path

1. Builder flashes CircuitPython (with or without the App’s UF2 drop).
2. App detects CIRCUITPY + two CDC ports as one device.
3. Builder selects MCU “Raspberry Pi Pico.”
4. Detection: 40 unique presses on a 4×12 matrix; App shows 4 row pins, 12 col pins, COL2ROW.
5. Coord map: 40 presses in order; layout generated as 4×12 ortho.
6. Runtime + KMK installed; `overlays/` empty.
7. Builder types in tester; all 40 keys highlight.
8. Builder assigns a layer and a hold-tap; change works with drive still mounted.
9. Builder hides drive via ToggleDrive; remaps again over live protocol; still works.

### AS-2 Import Pog 2.2 board

1. User opens a folder containing a 2.2.3 `pog.json` with splitSerial, encoders, RGB, custom keymap strings.
2. All keys, layers, pins, split fields, rgbOptions, kbFeatures appear.
3. Unknown field `autoshiftTapTime` is preserved and applied if the runtime supports it.
4. Export Pog-compat JSON reloads in old Pog without crash.

### AS-3 Unplug during write

1. User starts KMK copy.
2. Cable yanked.
3. App shows error, marks device disconnected, does not crash, does not corrupt the local project.
4. Replug offers “retry copy.”

### AS-4 Malicious pack

1. User imports a `.kbpack` with `../` paths and an overlay that is SerialACE-like.
2. Extract fails closed or sandboxes; overlay is shown as a diff and not auto-applied.

### AS-5 Split

1. Left and right halves appear as one Board with two PhysicalDevices.
2. Flashing runtime can target “left”, “right”, or “both.”
3. Keymap is shared; detection can be per half.

---

## 5. Traceability

| Requirement area | Catalog |
|---|---|
| FR-P* | C, interchange |
| FR-D* | B |
| FR-W* | D, F |
| FR-E* | E, G, H, I, J, K, L |
| FR-L* | M |
| FR-S* | N, O |
| Standout | R |
| Won’t | S, FR-X* |
