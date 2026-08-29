# 09 — Stack comparison: Compose Multiplatform vs Wails 3

The owner asked for a from-scratch implementation in **Kotlin + Compose Multiplatform**, possibly with **Kotlin/Native**, **or** in **Wails 3**.

This document is the engineering recommendation. The architecture in `08` is valid for both.

---

## 1. Clarify Kotlin/Native

**Kotlin/Native is not the desktop UI stack.**

| Target | Typical UI | Typical I/O |
|---|---|---|
| Compose Desktop | JVM + Skia | Java/JNI libraries |
| Compose Android | JVM/ART | Android USB APIs |
| Compose iOS | Kotlin/Native + Skia | Native bindings |
| Compose Web | Kotlin/JS or Wasm | WebHID / no MSC |
| “Kotlin Native desktop” | Not a supported Compose product in the way people mean | POSIX |

You *can* compile a Kotlin/Native dylib and `System.load` it from Compose Desktop (JNI-less via JNA, or JNI). That is a **helper**, not the app.

If someone says “write the whole configurator in Kotlin Native,” they are choosing an unsupported path for USB/HID/UI. Do not do that.

**Correct CMP reading:** shared Kotlin domain + Compose UI on **desktop JVM first**, Android later, iOS optional, Kotlin/Native only for iOS or a macOS helper.

---

## 2. What this app actually needs from a stack

Weighted by Pog’s real work, not by blog-post desktop-app criteria:

| Need | Weight | Wails 3 | CMP Desktop (JVM) | Kotlin/Native app |
|---|---|---|---|---|
| Serial CDC read/write, exclusive open, clean shutdown | Critical | Excellent (`go.bug.st/serial`) | Excellent (`jSerialComm`) | DIY per OS |
| USB HID raw (VIA-like + our protocol) | Critical | Good (`karalabe/hid`, `sstallion/go-hid`) | Good-enough (`hid4java`, `purejavahidapi`) — more sharp edges | DIY |
| Filesystem + volume listing + atomic write | Critical | Excellent | Excellent | Partial |
| USB hotplug | High | Go + OS | JVM + native | Native |
| Complex canvas editor (keys, snap, select) | High | Web (SVG/Canvas/WebGL) — known | Compose Canvas — known | Pain |
| Native look / high refresh editor | Medium | OS WebView (not Chromium) | Skia, very good | — |
| Binary size | Medium | ~8–20 MB | ~60–120 MB with JRE | Small |
| RAM | Medium | Low–mid | Higher (JVM) | Low |
| Team reuses Pog Vue skill | Medium | Yes | No | No |
| One language everywhere | Medium | No (Go + TS) | Yes (Kotlin) | Yes |
| Android companion | Medium-low v1 | Experimental Wails mobile | First-class CMP | — |
| iOS companion | Low | Experimental | CMP + Native | — |
| Web live-remap companion | Medium | Same frontend possible | Compose Web or separate | — |
| Packaging / signing | High | Good, still beta toolchain | jpackage, documented | Hard |
| Maturity in 2026 | High | **v3 is beta**; v2 stable but different API | Production (JB, many apps) | N/A |
| Testability of domain | High | Go tests | Kotlin tests | — |
| Plugin / bindings | Medium | Generated TS from Go services | In-process | — |
| Serial lifetime on quit | Critical | Must use shutdown hooks (known class of bugs) | Same class of bugs | Same |

Hardware access is **not** a reason to reject either stack. Both can do serial, HID, and MSC. Pog’s pain was Electron + Node native modules (`serialport`, `drivelist`, `npmRebuild: false`), not “JS cannot do serial.”

---

## 3. Wails 3

### What it is

Go host + OS WebView (WebView2 / WKWebView / WebKitGTK). v3 (beta): explicit app/window lifecycle, **services** with static TS bindings, multi-window, Taskfile builds, Windows/macOS/Linux amd64+arm64. Mobile experimental, not in the desktop compatibility promise.

v2 remains the “stable” line; v3 is a real port, not a bump. Teams already use v3 in production, per upstream. That is a **managed risk**, not a toy.

### Why it fits this product

1. **Go is an unusually good language for this host.** USB, serial, zip, hashing, concurrent session mutexes, UF2 copies — boring and well-tested.
2. **You can keep a Vue (or Svelte) editor** and not throw away Pog’s layout/keymap UX knowledge. The domain still moves to Go; the widgets can be new.
3. **Small artifacts**, easy CI matrix, no JRE story.
4. **Multi-window v3** matches a workshop: editor + floating tester + console.
5. **Server build** (headless) gives the CLI (`workshop push`) for free.
6. Generated bindings beat Pog’s hand-written `window.api` IPC.

### Costs

1. **Two languages** (Go + TS). Domain must live in Go or you will duplicate it.
2. **WebView variance** (especially Linux GTK4/WebKitGTK 6). Test Linux early.
3. **Beta** until 3.0 GA: pin a commit, read WEPs, expect binding churn.
4. **Android/iOS** are not a promise. If mobile is v1, this is the wrong flagship.
5. Serial close on Windows can hang in *drivers*; Wails does not save you (see public issues). The state machine in `08` does.

### Suggested Wails module map

```
cmd/workshop/          // desktop
cmd/workshop-cli/      // same services, no window
internal/domain/
internal/app/          // services
internal/transport/{msc,serial,hid}
internal/backend/kmk/
internal/detect/
frontend/              // Vue or Svelte
```

Each `internal/app` service is a Wails service. UI never opens a port.

---

## 4. Compose Multiplatform

### What it is

Kotlin Multiplatform + Compose UI. Desktop target is **JVM** (Skia). Packaging via `org.jetbrains.compose` + `jpackage` (dmg/msi/deb). Android is the same UI toolkit. iOS is Kotlin/Native.

### Why it fits this product

1. **One language** for domain + UI + (later) Android.
2. **Layout editor** as Compose Canvas is a joy compared to fighting DOM transforms (Pog scales a picker with CSS `transform`).
3. **In-process**: no IPC, no binding generator, no “main vs renderer” serial bugs.
4. **Maturity** of Compose Desktop is higher than Wails 3’s API freeze story.
5. **Android companion** for live remap + tester is a real product (USB-OTG or BLE later).
6. JetBrains-class text/accessibility if you invest.

### Costs

1. **JVM weight.** Accept ~80–120 MB installs or invest in GraalVM native-image (high risk with JNI serial/HID).
2. **HID on JVM** is the weakest I/O piece. Budget a native helper if hid4java fails on a target OS.
3. **Hotplug** is not free; you will write expect/actual.
4. **You rewrite the entire UI.** Pog Vue does not come along.
5. **Web companion** is a second UI (Compose Web is not “the same desktop app in Chrome”).
6. **Kotlin/Native helpers** add a build matrix (dylib/so/dll signing).
7. Linux packaging (deb vs AppImage) is more work than Wails’ single binary.

### Suggested CMP module map

```
:domain            // commonMain, pure
:application       // commonMain
:transport-api     // commonMain interfaces
:transport-jvm     // jSerialComm, hid, java.nio
:transport-android // UsbManager, UsbSerial
:backend-kmk       // commonMain (file plan) + resources
:composeApp        // desktop + android UI
:desktopMain       // jpackage, hotplug
```

No domain types in `@Composable` files beyond immutable snapshots.

---

## 5. Decision matrix (scored)

Score 1–5, weighted. Higher is better for *this* product.

| Criterion | W | Wails 3 | CMP Desktop |
|---|---|---|---|
| Serial/HID/MSC host quality | 5 | 5 | 4 |
| Time to a Pog-complete workshop | 5 | 5 | 3 |
| Time to Vial-complete editors | 4 | 4 | 4 |
| Connection lifetime / shutdown | 5 | 4 | 4 |
| Layout editor quality ceiling | 3 | 3 | 5 |
| Binary size / update UX | 2 | 5 | 2 |
| Mobile path | 3 | 1 | 5 |
| Web live-remap path | 2 | 4 | 2 |
| API / toolchain stability (2026) | 4 | 3 | 5 |
| Team: Vue + systems | 3 | 5 | 2 |
| Team: Kotlin | 3 | 2 | 5 |
| License simplicity | 2 | 5 | 5 |
| Testability | 3 | 5 | 5 |
| **Weighted** | | **4.2** | **3.8** |

The gap is small. **People** flip it.

- If the implementing team is **you + Vue/Go comfort, desktop-first, mobile “someday”** → **Wails 3**.
- If the implementing team is **Kotlin-first, Android companion in the first year, willing to rewrite UI** → **CMP**.
- If the implementing team is **“Kotlin Native everywhere”** → **re-educate, then CMP Desktop**, not Native.

---

## 6. Recommendation

**Default recommendation: Wails 3 host + new Vue/Svelte UI, domain in Go, KMK runtime in Python.**

Reasons, in order:

1. The hard unique work is **host I/O + protocol + runtime**, not pixel-identical Compose.
2. Pog already taught you what the UI must do; a web UI is the shortest path to *workshop-complete*.
3. Headless CLI and later a **tiny WebHID companion** (same protocol, same frontend components) fall out of Wails more naturally than out of CMP.
4. Size and flashing-tool feel matter for DIY users (they compare to Vial’s small desktop app).
5. Wails 3 beta risk is smaller than “HID on JVM + 100 MB + full UI rewrite + no tests yet.”

**Switch to CMP if** any of these is true at kickoff:

- Android live-remap is a v1 committed feature
- The team will not write Go
- You consider the layout editor the product (vendor design tool) more than flashing

**Hybrid that looks smart and is usually a trap:** Go sidecar + Compose UI. Two runtimes, two packagers, Pog-like IPC. Only do this if you already have both codebases.

**Kotlin/Native:** use it later for an iOS companion or a macOS IOKit helper. Do not start there.

---

## 7. Risks and mitigations

### Wails 3

| Risk | Mitigation |
|---|---|
| v3 API churn | Pin module versions; wrap services behind our ports |
| Linux WebKitGTK | CI on Ubuntu 22.04 + 24.04 from week 1 |
| WebView file drag (UF2) | Do drops in Go via native dialogs |
| “We’ll just reuse Pog Vue” | Do not. New UI on old store will port bugs. Reuse *ideas*, not `keyboardStore` |

### CMP

| Risk | Mitigation |
|---|---|
| hid4java failures | Protocol works on serial first; HID behind interface |
| GraalVM temptation | Don’t, until JNI-free transports |
| Android USB-OTG mess | Desktop v1; Android is live-remap only |
| Huge install | jlink custom runtime; document size |

### Both

| Risk | Mitigation |
|---|---|
| Serial hang on quit | State machine + timeouts + never block shutdown on `Read` |
| FAT corruption | Atomic replace + “eject” guidance |
| Scope explosion | P0 cut line in `05` |

---

## 8. What to build first (stack-independent spike)

A 1–2 week spike that must run on **the chosen stack** before UI polish:

1. Enumerate MSC + serial, group by USB serial
2. Open one CDC, exclusive, clean close on quit (automated test)
3. Copy a file atomically to CIRCUITPY
4. Parse a Pog 2.x `pog.json` into domain
5. Fake HID/serial capability + set one keymap key
6. Package a signed (or at least arch-correct) binary on Win/mac/Linux

If step 2 or 6 fails, the stack is not ready. Change stack then, not after the keymap editor exists.

---

## 9. License note

- Wails: MIT
- Go serial/hid libs: typically BSD/MIT
- Compose: Apache 2.0
- KMK: typically MIT
- QMK/Vial: GPL-2.0 — **keep QMK protocol clients in a clearly documented module**; do not copy vial-qmk into the app
- Pog: MIT — importing `pog.json` and rewriting is fine

Prefer MIT for the successor app to stay compatible with Pog users and vendors.
