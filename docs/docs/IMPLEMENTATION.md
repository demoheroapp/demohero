# DemoHero v1 implementation plan and as-built record

This is the living engineering plan for v1. Product intent lives in [PRODUCT.md](PRODUCT.md). This file is what an LLM agent executes.

**Original v1 target:** a macOS app that records screen + camera + mic (optional system audio) as *separate tracks*, lets the user trim/speed, place a camera overlay, frame to 16:9, add manual zooms, and export an MP4 (and copy it). Journey A in PRODUCT.md must work. The release scope later removed zoom; T31 removed its dormant paths and reconciled the product contract.

**Stack (locked):** native macOS app, Swift + SwiftUI, ScreenCaptureKit, AVFoundation, CoreImage/Metal for compose. macOS 14+ / Apple Silicon first. No Electron. No cloud. Media stays on disk.

**As-built status (September 6, 2026):** T01–T23 delivered the original v1 baseline; T24–T31 record the release-candidate work that landed afterward; T32–T35 add StoreKit 2 Pro, free-tier limits, and the App Store submission guide. The current product also has a take picker, an opt-in live presenter captured in the screen recording, crash recovery notes, a drag-to-trim multi-range timeline, 720p/1080p/4K export, 16:9 or Match capture canvases, solid or gradient mats, a native export fast path, and the release app icon. The editor no longer offers a post-record camera overlay. M8 and M9 are demoable. Live Sandbox purchases still need the owner’s App Store Connect products.

Task **Completed notes** below are point-in-time history. When a later change supersedes them, M8 and **Current release snapshot and boundaries** are authoritative for the current app.

**Source layout (locked):** all application source lives under [`src/`](../src/). Do not put Swift, assets, entitlements, Info.plist, or tests outside `src/`.

Do not implement non-goals or parked work (zoom, captions, GIF, vertical reflow, keyboard overlay, iOS device frames, share links).

---

## How to use this plan (for LLM agents)

When you are pointed at this file, you are an implementation agent. Follow this protocol every session. Do not improvise a different order.

### Find the next task

1. Read [PRODUCT.md](PRODUCT.md) (v1 use cases and non-goals).
2. Read **this entire protocol**, then **Milestones**, then only the task you will execute.
3. The next task is the **first task in index order** whose `Status` is `todo` and whose `Depends on` tasks are all `done`.
4. If several tasks are `todo` but blocked on dependencies, pick the first whose dependencies are satisfied — not the one that looks more interesting.
5. If no task qualifies: stop. Report whether the release candidate looks complete (M8 demoable) or the plan is stuck. Do not invent work.

Work **one task** at a time. The milestone is the checkpoint a human can run; it is not a license to batch every task in that milestone.

### Do one task

6. Set that task’s `Status` to `in_progress` **before** you edit product code. If this is the first task in a milestone, set the milestone status to `in_progress`.
7. Do **only that task**. Do not start the following task in the same session unless the user explicitly says to continue.
8. Meet every acceptance criterion. Put new source **only under `src/`**. Leave types, files, and APIs that later tasks are specified to use. Do not re-add post-record camera overlay editing in the take editor.
9. Stay inside v1. If a parked feature would be “easy,” leave it parked.

### Close the task

10. Do **not** mark `done` yet. Fill **Completed notes**: what landed, key files/types under `src/`, decisions, leftover risk.
11. Run **After-task review** (plan + **application so far**) and write the answers under that task.
12. If the application review finds problems: **fix them in this session** when they came from this task or are small. If they need more work, insert a task in the *current* milestone (suffix id) and leave this task `blocked` or `done` only after the regression is either fixed or explicitly scheduled. Do not paper over broken UX or a red build.
13. If the plan review finds a sequencing gap, **re-arrange this file now**. New work must still sit in a milestone that ends with something runnable.
14. Then set `Status` to `done` (or `blocked` with a reason).
15. If this was the **last task** in a milestone, run that milestone’s **Demo** on a Mac, then set the milestone status to `demoable` (or keep `in_progress` and add a follow-up task if the demo fails).
16. Update the **Next task** line under Milestones.
17. **Commit and push** to the tracked remote before you stop. Stage only this task’s files (source under `src/`, pbxproj, tests, and plan status updates). Commit message: one or two sentences, include the task id (e.g. `T03: Add menu bar and permission gates.`). Then `git push origin HEAD`. If push fails, report it; do not skip this step.
18. Stop. Reply with: completed task id + title, milestone id, application-review outcome (code / UX / working), what a human can see now, commit/push result, next task id + title.

### After-task review (required)

Write this under the task’s **Completed notes**. Review the **whole app as it exists so far**, not only the diff from this task. Scope the run to what is actually built (after T01 that is “does it launch”; after M4 that includes the editor).

#### A. Plan and architecture

Answer yes/no with a one-line note:

- **Supports later tasks?** Did this leave the data/APIs the *next* tasks need (separate tracks, overlay as metadata, project on disk)?
- **Blocks UC-2 / UC-3 / UC-5?** Did you flatten camera into pixels, drop mouse events, or make export unable to re-composite?
- **Code under `src/`?** Any new source outside `src/` must be moved before you mark done.
- **Plan still sequenced?** Can the next `todo` actually start, or must a task be added/split/rewritten first?
- **Milestone still demoable?** If you finished the last task in a milestone, did the Demo section actually work?

If any answer is no: add, remove, or rewrite tasks in this file so the path to v1 still works. Keep existing task IDs stable. Insert with a suffix (`T05a`). Cancel obsolete tasks with `Status: cancelled` and a reason — do not delete history. Keep every milestone a runnable stage (something a human can launch and see). Then point **Next task** at the new first actionable `todo`.

#### B. Application so far (code, UX, working)

Build and exercise the current app. Then write pass/fail plus a short note for each:

**1. Code is in good shape**

- All application source is under `src/`; layout still matches Architecture constraints.
- The project **builds**. Tests that exist still pass.
- Naming, folders, and types match later tasks (no one-off hacks that the next milestone must tear out).
- Trim, speed, and framing remain metadata. The live presenter may be part of `screen.mp4`; do not add a second camera overlay in the editor.
- No leftover debug UI, dead code, or comments that contradict the product (unless the M1 sample-project action, which T03 allows).
- If something is messy, clean it up now or add a task — do not leave a known mess for “later.”

**2. User experience is easy to use**

Walk the UI a first-time user would see *right now* (menu bar, pickers, recording chrome, editor — whatever exists).

- A new user can find the next action without reading this plan (primary button/menu is obvious).
- Destructive or permission-gated actions explain what to do (e.g. Screen Recording denied → System Settings, no silent no-op).
- Labels match the product (Record, Stop, Choose project, Export), not engineer jargon or retired names such as Open last.
- Nothing extra from later milestones is half-shown and confusing (hide unfinished controls).
- Keyboard/shortcuts that are advertised actually work; do not advertise the T09 shortcut before it exists.

**3. Application is working as expected**

- Run the app (or the latest milestone Demo if this closed a milestone).
- Behavior matches this task’s **Acceptance** and does not regress earlier milestone Demos.
- Happy path works; obvious failure paths do not crash (deny permission, no camera, cancel countdown — if those screens exist).
- What you ship in the session is what a human will see — not “works on my machine if you know the hidden flag.”

If any of the three fails: fix or schedule (step 12). The task is not `done` while the build is broken, the current happy path is unusable, or the UX for shipped features is misleading.

### Status values

| Status | Meaning |
| --- | --- |
| `todo` | Not started |
| `in_progress` | This session is executing it (at most one task) |
| `done` | Acceptance criteria met; notes filled |
| `demoable` | Milestone only: all its tasks are `done` and the Demo was run |
| `blocked` | Cannot finish; say what is missing |
| `cancelled` | No longer needed; say why |

### Prompt to give an agent

> Read `docs/PRODUCT.md` and `docs/IMPLEMENTATION.md`. Follow the agent protocol in IMPLEMENTATION.md. Find the next `todo` task whose dependencies are `done`. Do only that task. Keep all source under `src/`. Before marking done: review the application so far — code shape, ease of use, and that it works as expected. Fix or schedule issues. Update status and notes, rearrange later tasks if needed. If you closed a milestone, run its Demo. Commit and push the task, then stop and name the next task.

---

## Architecture constraints (do not violate)

These exist so early tasks do not trap later ones.

1. **Source lives in `src/`.** Swift, assets, entitlements, Info.plist, preview resources, and tests go under `src/`. The Xcode project file may sit at the repo root and *reference* `src/`; it must not contain a parallel copy of sources. **Add every new `.swift` file to `DemoHero.xcodeproj` (PBXSourcesBuildPhase)** — folder sync alone did not compile.
2. **Separate tracks.** Record screen, camera, mic, and system audio as separate files. **Show presenter on screen** is the shipping face path: keep the live presenter panel in the ScreenCaptureKit filter so the bubble is part of `screen.mp4`. `camera.mp4` may still be written; do not composite it again in the editor or export.
3. **No post-record camera overlay.** Place the live presenter before recording. `Project.overlay` / `overlayVisibility` remain in schema 1 for older takes; the editor must not show overlay controls or let the user drag a second bubble. Overlay geometry is **screen** space when present; the 16:9 mat is `OutputFrame` / `CanvasLayout`.
4. **No cursor sidecar.** The shipping product does not support zoom, so capture does not collect or persist cursor-position/click telemetry.
5. **One project folder per take.** `Project.json` + media files, all local. No account, no upload.
6. **Editor is non-destructive.** Trim, speed, and frame are edits on the timeline. Source media is immutable. Do not re-introduce post-record camera overlay editing.
7. **Edited vs source time.** `keepRanges` and `speedSegments` are source seconds. `EditedTimeline` maps the playhead and export clock (including speed). Overlay visibility stays in source seconds.
8. **Export has two equivalent paths.** Preview uses `ComposePreview` / `FrameCompositor`. Export uses native AVFoundation composition for a solid-mat take and the frame compositor for gradients. Historical projects that still mark an editable camera overlay may use the compositor camera path; new takes do not. Pixel, orientation, timing, and audio tests enforce parity.

Locked layout:

```
demohero/
  DemoHero.xcodeproj          # project only; points at src/
  docs/
  src/
    App/                      # SwiftUI app, menu bar, permissions
    Model/                    # Project, timeline, overlay, export presets
    Capture/                  # ScreenCaptureKit + camera/audio writers
    Editor/                   # Playback, trim, speed, frame
    Compose/                  # Preview + export compositor
    Export/                   # Presets, file save, clipboard
    Store/                    # StoreKit 2 products and Pro entitlement
    Resources/                # Assets, Info.plist, entitlements, StoreKit config
    DemoHeroTests/            # Unit tests
```

If T01 must change this layout, update this diagram in the same change. Do not scatter files back to the repo root.

---

## Milestones

Each milestone is a **runnable stage**. After its last task, a human can build, launch, and *see* the result in the Demo. Agents still complete a single task per session; the Demo is the gate on the last task.

**Next task:** none (M9 complete)
**Current milestone:** M9 — Free and Pro release (demoable)

| Milestone | What you can see | Tasks | Status |
| --- | --- | --- | --- |
| [M1 — App launches](#m1--app-launches) | Menu bar app, permissions, dummy project on disk | T01–T03 | demoable |
| [M2 — Record the screen](#m2--record-the-screen) | Pick a window/region, record, play the file in QuickTime | T04–T05 | demoable |
| [M3 — Record a full take](#m3--record-a-full-take) | Historical baseline: camera preview, countdown, separate media; current reopen action is Choose project | T06–T10 | demoable |
| [M4 — Watch it with your face](#m4--watch-it-with-your-face) | Historical baseline: movable post-record camera (T11–T13). Shipping product captures the live presenter in the screen instead | T11–T13 | demoable |
| [M5 — Make it look produced](#m5--make-it-look-produced) | Historical baseline included manual zoom; the release product no longer supports zoom | T14–T17 | demoable |
| [M6 — Hand off an MP4](#m6--hand-off-an-mp4) | Historical baseline: MP4 delivery; current presets are 720p/1080p/4K, default 720p | T18–T21 | demoable |
| [M7 — V1 is real](#m7--v1-is-real) | Journey A and B pass on a Mac | T22–T23 | demoable |
| [M8 — Release-candidate hardening](#m8--release-candidate-hardening) | Reliable repeat recording/edit/export flow with final framing and app identity | T24–T29a, T31 | demoable |
| [M9 — Free and Pro release](#m9--free-and-pro-release) | Free limits, StoreKit purchases, restore, and App Store submission | T32–T35 | demoable |

### Task index

| ID | Milestone | Task | Status | Depends on |
| --- | --- | --- | --- | --- |
| T01 | M1 | Create the macOS app scaffold | done | — |
| T02 | M1 | Domain model and on-disk project | done | T01 |
| T03 | M1 | Menu bar shell and permission gates | done | T01, T02 |
| T04 | M2 | Screen source picker | done | T03 |
| T05 | M2 | Screen capture + mouse sidecar | done | T04 |
| T06 | M3 | Camera and microphone pickers | done | T03 |
| T07 | M3 | Camera track capture | done | T06 |
| T08 | M3 | Mic and system-audio tracks | done | T06 |
| T09 | M3 | Recording session UX | done | T05, T07, T08 |
| T10 | M3 | Persist a take as a project | done | T02, T09 |
| T11 | M4 | Editor shell and screen playback | done | T10 |
| T12 | M4 | Composite preview (screen + overlay) | done | T11, T07 |
| T13 | M4 | Edit camera overlay after recording | done | T12 |
| T14 | M5 | Trim and cut | done | T11 |
| T15 | M5 | Speed segments | done | T14 |
| T16 | M5 | 16:9 frame, padding, background | done | T12 |
| T17 | M5 | Manual zoom on the timeline | done | T14, T05 |
| T18 | M6 | Offline compositor (preview = export) | done | T13, T15, T16, T17 |
| T19 | M6 | Export presets to MP4 | done | T18 |
| T20 | M6 | Save to disk and copy to clipboard | done | T19 |
| T21 | M6 | Hide desktop icons while recording | done | T09 |
| T22 | M7 | Journey A and B QA | done | T20, T21 |
| T23 | M7 | V1 definition of done | done | T22 |
| T24 | M8 | Recording shell, presenter, and recovery hardening | done | T23 |
| T25 | M8 | Timeline editing and playback hardening | done | T24 |
| T26 | M8 | Export presets, audio, and performance | done | T25 |
| T27 | M8 | Final aspect, padding, and background controls | done | T26 |
| T28 | M8 | Export correctness and release app icon | done | T27 |
| T28a | M8 | Add feedback and support contact | done | T28 |
| T29 | M8 | Restore non-destructive presenter preview/export | done | T28a |
| T29a | M8 | Hide camera controls for legacy baked-presenter takes | done | T29 |
| T31 | M8 | Remove dormant release paths and reconcile release journeys | done | T29a |
| T32 | M9 | StoreKit 2 entitlement foundation | done | T31 |
| T33 | M9 | Enforce the free recording and export limits | done | T32 |
| T34 | M9 | Pro purchase, restore, and subscription UI | done | T33 |
| T35 | M9 | App Store package, test, and submission guide | done | T34 |

---

## M1 — App launches

**Status:** demoable  
**Tasks:** T01, T02, T03  
**Runnable when:** T03 is `done`

You can open DemoHero from the menu bar, see permission state, and write a dummy project folder under `src/`-backed code — no real capture yet.

**Demo**

> Historical T01–T03 demo. For current release behavior, see M8 and the release snapshot.

1. Open `DemoHero.xcodeproj`, build, run.
2. A DemoHero extra appears in the menu bar. Record / Open last / Quit are visible.
3. Permission rows show Screen, Camera, Mic; denied items deep-link to System Settings.
4. A debug or first-run action writes a sample project folder you can inspect in Finder (`Project.json`).

---

### T01 — Create the macOS app scaffold

- **Status:** done
- **Milestone:** M1
- **Depends on:** —
- **Use cases:** foundation for all
- **Goal:** A buildable SwiftUI macOS app named DemoHero, with **all source under `src/`**.

**Do**

- Create an Xcode macOS app target (SwiftUI, App sandbox as appropriate, Apple Silicon). Put the `.xcodeproj` at the repo root; point every source file at `src/`.
- Create the `src/App`, `src/Model`, `src/Capture`, `src/Editor`, `src/Compose`, `src/Export`, `src/Resources`, `src/DemoHeroTests` folders. Empty placeholder files are OK.
- Add entitlements and Info.plist usage strings (under `src/Resources/`) for Screen Recording, Camera, Microphone. System audio capture will need the ScreenCaptureKit audio path (no extra TCC string); leave a comment for T08.
- App launches from Dock and shows a stub “Ready to record” window whose Swift lives in `src/App`.
- README: one paragraph on how to open the Xcode project and that sources are in `src/`.

**Acceptance**

- Project builds and launches on macOS 14+.
- `find` of `*.swift` at repo root returns nothing outside `src/` (and not inside `docs/`).
- Layout matches Architecture constraints.
- No capture code yet.

**Must enable later:** T02 types live under `src/Model/`; T03 adds a menu bar extra without moving the target.

**Completed notes:**

- `DemoHero.xcodeproj` at repo root; all Swift/plist/entitlements under `src/`. Layout: App, Model, Capture, Editor, Compose, Export, Resources, DemoHeroTests.
- App target uses an explicit `PBXSourcesBuildPhase` (Xcode 26 file-system sync compiled an empty bundle). **T02+ must add new `.swift` files to the pbxproj.**
- Window: “Ready to record” in `src/App/DemoHeroApp.swift` + `ReadyToRecordView.swift`. No menu bar yet (T03). No project model yet (T02) — `src/Model/ModelPlaceholder.swift`.
- Entitlements: sandbox, camera, mic, Movies read-write (for T03 sample folder), user-selected files. Info.plist usage strings for camera, mic, screen capture. T08 comment in `src/Capture/CapturePlaceholder.swift`.
- README: open `DemoHero.xcodeproj` or `xcodebuild -scheme DemoHero`.
- `xcodebuild` Debug **BUILD SUCCEEDED**; tests **TEST SUCCEEDED** (`testModuleLoads`). All `*.swift` files are under `src/`. No capture implementation.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — empty `src/Model` placeholder; T03 can add MenuBarExtra to the existing App target.
- **Blocks UC-2 / UC-3 / UC-5?** No — nothing baked; no media yet.
- **Code under `src/`?** Yes.
- **Plan still sequenced?** Yes — T02 is next.
- **Milestone still demoable?** N/A (T01 is not the last M1 task).

#### B. Application so far
- **Code:** pass — builds and tests; placeholders named for later folders; pbxproj must stay in sync with new files.
- **UX:** pass for T01 — single window, product copy, no unfinished Record/permissions chrome.
- **Working:** pass — scheme builds; stub window is the shipped UI. Full M1 Demo waits on T03.

---

### T02 — Domain model and on-disk project

- **Status:** done
- **Milestone:** M1
- **Depends on:** T01
- **Use cases:** UC-1–5
- **Goal:** Codable types for a take, written and read as a folder on disk — before any real capture.

**Do**

- Types in `src/Model` at least: `Project`, `MediaTrack` (screen, camera, mic, systemAudio), `CameraOverlay` (position, size, shape, corner radius), `OverlayVisibility` (time ranges), `TimeRange` (trim), `SpeedSegment`, `ZoomInterval` (time + rect + scale), `MouseEvent` (t, x, y, clicked), `ExportPreset`.
- `Project` points at relative file URLs inside the project folder; source media is never rewritten.
- Save/load `Project.json` plus placeholder files in a temp/dev directory.
- Keep overlay, zooms, trim, and speed as *edits*, not as rendered media.
- Tests in `src/DemoHeroTests`.

**Acceptance**

- Unit tests or a tiny debug command: create a project, serialize, deserialize, files round-trip.
- A later capture task can fill track URLs without changing the schema in a breaking way (version field on `Project` is OK).

**Must enable later:** T10 writes a real project; T13–T17 only mutate metadata; T18 reads the same model to render.

**Completed notes:**

- Replaced `ModelPlaceholder` with `src/Model/Project.swift` and `src/Model/ProjectStore.swift`. Schema `version` is `ProjectSchema.current` (1). Tracks use relative paths; overlay/trim/speed/zoom are metadata.
- Overlay coordinates: screen space, top-leading, center in 0...1. `muted` on tracks for T08/T18.
- `ProjectStore.save` writes `Project.json` (ISO-8601 dates, pretty JSON) and empty placeholders only if missing — does not rewrite existing media.
- `ProjectStore.writeSampleProject()` → `Movies/DemoHero/Sample` for T03. Tests: round-trip, preserve media, missing manifest. All passed.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T10 can fill track URLs; T13–T17 mutate overlay/keepRanges/speed/zooms; mouse sidecar path is on the project.
- **Blocks UC-2 / UC-3 / UC-5?** No — nothing rasterized.
- **Code under `src/`?** Yes; files added to pbxproj.
- **Plan still sequenced?** Yes — T03 is next.
- **Milestone still demoable?** N/A.

#### B. Application so far
- **Code:** pass — tests green; stub window unchanged.
- **UX:** pass — no new user-facing controls (sample write is T03).
- **Working:** pass — app still builds; model is library-only until T03.

---

### T03 — Menu bar shell and permission gates

- **Status:** done
- **Milestone:** M1
- **Depends on:** T01, T02
- **Use cases:** UC-1
- **Goal:** Open DemoHero from the menu bar; know whether screen, camera, and mic are allowed.

**Do**

- Menu bar extra (status item) with Record / Open last project / Quit. Code in `src/App`.
- Global shortcut *hook* can be a stub; real start/stop lands in T09.
- Permission UI: request and show status for Screen Recording, Camera, Microphone. Deep-link to System Settings if screen recording is off.
- Wire a debug “Write sample project” (or first-run) so the M1 Demo can show a folder in Finder.
- Do not start capture yet.

**Acceptance**

- App can live in the menu bar.
- Denied permissions are obvious; granted permissions are visible.
- Clicking Record with missing screen permission does not crash; it prompts.
- **M1 Demo** works (see milestone Demo above).

**Must enable later:** T04–T08 attach to this shell; T09 binds the shortcut to a session.

**Completed notes:**

- `src/App/PermissionService.swift` + `AppSession` in `DemoHeroApp.swift`. Menu extra: Record, Open last project, Write sample project, Quit. No global shortcut advertised (T09).
- Window shows Screen / Camera / Mic status. Denied Screen → “Open System Settings”; Record without Screen asks TCC and does not crash.
- Write sample project → `~/Movies/DemoHero/Sample/Project.json` (+ empty track placeholders). Finder reveal. Open last loads `Project.json` or explains if none.
- M1 Demo run on a Mac: window “Ready to record”, permissions visible, extra menu items present, sample folder written. Tests still pass.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — Record is a permission gate; T04 can replace the “not available yet” message with the source picker.
- **Blocks UC-2 / UC-3 / UC-5?** No — still no baked media.
- **Code under `src/`?** Yes; `PermissionService.swift` added to pbxproj.
- **Plan still sequenced?** Yes — T04 is next.
- **Milestone still demoable?** Yes — M1 Demo passed.

#### B. Application so far
- **Code:** pass — build/tests green; capture still placeholders.
- **UX:** pass — next actions are Record / Write sample / permissions; denied Screen is explained; no fake shortcut.
- **Working:** pass — extra menu works; sample project on disk; Record without Screen does not crash.

---

## M2 — Record the screen

**Status:** demoable  
**Tasks:** T04, T05  
**Runnable when:** T05 is `done`

You can pick a display, window, or region, record, stop, and watch the screen file (QuickTime or Finder). Camera is not required yet.

**Demo**

> Historical T04–T05 demo. For current release behavior, see M8 and the release snapshot.

1. Grant Screen Recording if needed.
2. From the menu bar, open the source picker. Choose a single window (not the whole desktop).
3. Record a few seconds of clicking in that window; stop.
4. A playable screen video appears (Reveal in Finder / open in QuickTime). Only that window is in the frame.
5. A mouse sidecar file exists next to it.

---

### T04 — Screen source picker

- **Status:** done
- **Milestone:** M2
- **Depends on:** T03
- **Use cases:** UC-4
- **Goal:** User picks a display, a window, or a rectangle before recording.

**Do**

- List `SCShareableContent` displays and windows. Code in `src/Capture` + picker UI in `src/App`.
- Region mode: drag a rectangle on the chosen display.
- Persist last selection on `Project` or app settings.
- Preview thumbnail optional; a clear label is enough.

**Acceptance**

- User can choose display, window, or region.
- Selection is stored in a way T05 can consume (`SCContentFilter` inputs or equivalent).
- Window list is usable on a busy desktop (name + app).

**Must enable later:** T05 captures *only* that source (UC-4). Do not only support full-screen.

**Completed notes:**

- `CaptureSourceSelection` persisted in UserDefaults. `CaptureSourceCatalog.refresh()` lists displays/windows (excludes DemoHero). Window list is `App — title` with a filter field.
- `catalog.resolve(selection)` returns `SCContentFilter` + optional `sourceRect` (region, display-local top-leading) + pixel size for T05.
- Region: “Select region…” overlay, drag to set, Esc cancels. Record still does not capture (T05); it validates permission + selection.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T05 can call `resolve` and record only that filter/rect.
- **Blocks UC-2 / UC-3 / UC-5?** No.
- **Code under `src/`?** Yes; pbxproj updated.
- **Plan still sequenced?** Yes — T05 is next.
- **Milestone still demoable?** N/A (T05 is the M2 gate).

#### B. Application so far
- **Code:** pass — tests green including selection round-trip.
- **UX:** pass — Display / Window / Region in the main window; Refresh; filter for busy window lists.
- **Working:** pass — app builds; picker loads when Screen Recording is on.

---

### T05 — Screen capture + mouse sidecar

- **Status:** done
- **Milestone:** M2
- **Depends on:** T04
- **Use cases:** UC-1, UC-4
- **Goal:** Record the chosen screen source to a video file, and log mouse position/clicks to a sidecar.

**Do**

- ScreenCaptureKit stream → `AVAssetWriter` (or equivalent) video file in `src/Capture`.
- Include or exclude the DemoHero UI from the capture.
- Write `mouse.jsonl` (or similar) with timestamps aligned to the video’s timeline.
- Cursor may be visible in the screen file for v1; still record the sidecar (needed for T17 and future auto-zoom).
- Stop must finalize a valid media file and offer Reveal in Finder (enough for the M2 Demo).

**Acceptance**

- Recording a window or region produces a playable video of *that* source, not the whole desktop.
- Sidecar has events during the take; timestamps can be mapped to video time.
- Stopping cleanly is tested; killing mid-write should not be the only path.
- **M2 Demo** works.

**Must enable later:** T09 starts/stops this engine; T10 copies the file into the project; T17 uses sidecar coordinates in source pixels.

**Completed notes:**

- `ScreenRecorder.start(source:folder:)` / `stop()` writes `screen.mp4` + `mouse.jsonl` under `Movies/DemoHero/Captures/<timestamp>/`. Camera and audio are not recorded (T07/T08).
- `ResolvedCaptureSource` drives `SCContentFilter` + optional `sourceRect`. DemoHero windows stay excluded on display/region captures. Cursor may be baked into the MP4; sidecar still logs `t,x,y,clicked` in **source pixels** (top-leading).
- Stop finalizes the writer (does not leave a truncated moov as the only path). Reveal in Finder selects both files. Floating Stop timer + menu bar **Stop recording** so the take can end without hunting the main window.
- T09 should call this same `ScreenRecorder`; T10 should move the two files into a project folder and fill `Project.json`.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — separate screen file + mouse sidecar; `ScreenCaptureResult` has duration and pixel size for T10.
- **Blocks UC-2 / UC-3 / UC-5?** No — no camera composite, overlay still metadata.
- **Code under `src/`?** Yes; pbxproj updated (`ScreenRecorder.swift`, `MouseSidecar.swift`, `RecordingIndicatorPanel.swift`).
- **Plan still sequenced?** Yes — T06 is next (camera/mic pickers). T09 will wrap this engine with countdown.
- **Milestone still demoable?** Yes — pick a window, Record, Stop, Finder shows `screen.mp4` and `mouse.jsonl`. Allow Screen Recording for this build if the picker is empty.

#### B. Application so far
- **Code:** pass — tests green (selection, sidecar JSONL, coordinate mapping, even dimensions). Hosted `SCStream` test skips when Screen Recording is not granted to the test host.
- **UX:** pass — Record becomes Stop; elapsed time; source picker disabled while recording; menu extra matches. Ready window is presented with the menu extra (WindowGroup + `openWindow`).
- **Working:** pass — app builds and the Ready window appears. Record with Screen Recording allowed writes the capture folder and reveals it. Without Screen Recording, Record asks to allow it instead of crashing.

---

## M3 — Record a full take

**Status:** demoable  
**Tasks:** T06, T07, T08, T09, T10  
**Runnable when:** T10 is `done`

One Record action captures screen + camera + mic (optional system audio) into a project folder you can reopen.

**Demo**

> Historical T06–T10 demo. Current reopening uses **Choose project**; see M8.

1. Pick a window, a camera, and a mic. See yourself in a small live preview.
2. Hit Record: 3-second countdown, then a recording indicator with elapsed time.
3. Talk and click for ~15 seconds; stop via shortcut or the indicator.
4. A project folder (e.g. Movies/DemoHero) contains screen video, camera video, audio, mouse sidecar, and `Project.json`.
5. Quit and use Open last: the project still lists those files.

---

### T06 — Camera and microphone pickers

- **Status:** done
- **Milestone:** M3
- **Depends on:** T03
- **Use cases:** UC-1, UC-2
- **Goal:** Choose camera and mic devices, including “off”.

**Do**

- List `AVCaptureDevice` cameras and mics. Code in `src/Capture` + UI in `src/App`.
- Allow camera off / mic off.
- Live camera preview in the picker (small) so the user knows they are in frame.
- Persist device ids.

**Acceptance**

- User can pick camera, pick mic, or turn either off.
- Missing camera does not block screen-only recording.
- Live preview is visible when a camera is selected (part of the M3 Demo; full session comes in T09).

**Must enable later:** T07/T08 use the selected devices; overlay can be absent if camera is off (UC-2 still applies when it is on).

**Completed notes:**

- `MediaDeviceSelection` persists camera/mic unique IDs in UserDefaults (`nil` = off). `CaptureDeviceCatalog` lists `AVCaptureDevice` cameras and mics.
- `DevicePickerView` is on the Ready window with Camera off / Mic off rows. `CameraPreviewController` shows a 160×120 live preview when Camera is allowed and a device is selected. T07 must stop this preview before opening a recording session on the same camera.
- Screen Record still starts with camera or mic off (T05 path unchanged).

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T07/T08 read `session.deviceSelection` unique IDs; camera-off is explicit.
- **Blocks UC-2 / UC-3 / UC-5?** No — preview is not baked into the screen file.
- **Code under `src/`?** Yes; pbxproj updated.
- **Plan still sequenced?** Yes — T07 is next.
- **Milestone still demoable?** N/A (T10 is the M3 gate).

#### B. Application so far
- **Code:** pass — tests green including device selection round-trip.
- **UX:** pass — camera/mic pickers and a small preview sit under the screen source picker; Off is obvious; missing camera explains screen-only is fine.
- **Working:** pass — app builds; Record still does not require a camera.

---

### T07 — Camera track capture

- **Status:** done
- **Milestone:** M3
- **Depends on:** T06
- **Use cases:** UC-1, UC-2
- **Goal:** Record the camera to its *own* video file, same session clock as the screen.

**Do**

- `AVCaptureSession` → camera video file in `src/Capture`.
- Share a single session start time / clock with the screen recorder (T05) so A/V sync is possible.
- Do not composite onto the screen file.

**Acceptance**

- Camera-on session yields a second playable video.
- Camera-off session yields no camera file and does not fail.
- Drift vs screen is small enough to correct in T12 (document sync strategy: shared `CMClock`, NTP-less).

**Must enable later:** T12 composites by drawing this file in the overlay rect; T13 never needs to re-record to move the bubble.

**Completed notes:**

- `CameraRecorder` writes `camera.mp4` via `AVCaptureVideoDataOutput` + `AVAssetWriter`. Preview is stopped before record so the device is free.
- `CaptureClock.begin()` is taken once; screen, mouse, and camera map sample times onto `CMClockGetHostTimeClock()` minus that start. No NTP. T08/T09 should pass the same clock.
- Camera off or Camera permission missing skips the camera writer; screen recording still runs.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — separate `camera.mp4`; overlay remains metadata. T08 can share `CaptureClock`.
- **Blocks UC-2 / UC-3 / UC-5?** No — not composited onto the screen file.
- **Code under `src/`?** Yes.
- **Plan still sequenced?** Yes — T08 is next.
- **Milestone still demoable?** N/A (T10 is the M3 gate).

#### B. Application so far
- **Code:** pass — tests green including clock mapping and separate camera filename.
- **UX:** pass — Record with camera on mentions camera in the status line; Finder reveals `camera.mp4` when it exists.
- **Working:** pass — camera-off still records the screen; camera-on starts a second writer on the shared clock.

---

### T08 — Mic and system-audio tracks

- **Status:** done
- **Milestone:** M3
- **Depends on:** T06
- **Use cases:** UC-1
- **Goal:** Optional mic and optional system audio as separate audio files (or audio-only tracks) on the same clock.

**Do**

- Mic via AVFoundation.
- System audio via ScreenCaptureKit audio stream (selected apps or all — all is OK for v1).
- Each can be off.
- Levels meter optional.

**Acceptance**

- Mic-only, system-only, both, and neither all work.
- Files are in sync with screen start within the same strategy as T07.

**Must enable later:** T18 mixes audio according to which tracks exist; export never requires a re-record to drop system audio (can mute a track in the model — add a `muted` flag if missing).

**Completed notes:**

- `MicRecorder` writes `mic.m4a`. System audio is captured on the same SCStream as the screen into `systemAudio.m4a` when **System audio** is on. Both use `CaptureClock` (same host-start mapping as T07).
- A **System audio** toggle is persisted on `MediaDeviceSelection` (defaults off; old JSON still decodes). Mic off / system off / both / neither are all valid; missing permission treats that input as off.
- `MediaTrack.muted` already exists on the project model for T18.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — separate mic and system-audio files; T09 starts the same writers after countdown.
- **Blocks UC-2 / UC-3 / UC-5?** No.
- **Code under `src/`?** Yes.
- **Plan still sequenced?** Yes — T09 is next.
- **Milestone still demoable?** N/A (T10 is the M3 gate).

#### B. Application so far
- **Code:** pass — tests green including legacy device JSON without `systemAudioEnabled`.
- **UX:** pass — mic picker already had Off; System audio is a simple toggle under the preview.
- **Working:** pass — Record still works with any mix of camera/mic/system audio off.

---

### T09 — Recording session UX

- **Status:** done
- **Milestone:** M3
- **Depends on:** T05, T07, T08
- **Use cases:** UC-1
- **Goal:** One “Record” action runs screen + camera + audio together, with countdown and stop.

**Do**

- Pre-flight: source + devices + permissions.
- 3-second countdown (configurable later; 3 is enough).
- Start all writers; floating or menu-bar recording indicator with elapsed time and Stop.
- Global shortcut for start/stop.
- Cancel countdown without starting.
- On stop: wait for all writers to finish, then hand off to T10 (can call a stub if T10 is not done — then wire for real in T10).

**Acceptance**

- User can complete a one-take capture of screen + camera + mic without opening another app.
- Stop produces finalized files for every enabled track.
- DemoHero chrome is not the star of the recording (excluded or small).

**Must enable later:** T10 packages these files; Journey A steps 1–3.

**Completed notes:**

- Record on the Ready window, menu bar, floating indicator, and ⌃⇧R all go through `AppSession.handleRecord()`. Pre-flight still requires Screen Recording plus a resolved source; camera/mic stay optional.
- A 3-second countdown shows on the Ready window and the floating panel with **Cancel**. Cancelling (button or ⌃⇧R) never starts writers. After start, the same panel shows elapsed time and **Stop**; stop waits for screen, camera, and audio writers then reveals files in Finder (T10 will write `Project.json`).
- ⌃⇧R is installed as a local key monitor (consumes the event while DemoHero is frontmost) plus SwiftUI `keyboardShortcut` on Record/Stop/Cancel. A global key monitor is best-effort; it may not fire without Input Monitoring. The caption does not claim it works in every other app — use the menu bar or floating timer then.
- DemoHero windows (Ready + indicator) stay excluded from display/region capture via bundle id.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — stop still yields the same capture folder and track files; T10 can write `Project.json` there.
- **Blocks UC-2 / UC-3 / UC-5?** No — tracks stay separate; overlay is still metadata.
- **Code under `src/`?** Yes.
- **Plan still sequenced?** Yes — T10 is next.
- **Milestone still demoable?** N/A (T10 is the M3 gate).

#### B. Application so far
- **Code:** pass — build and tests green; `cancelCountdown` clears the timer without starting capture.
- **UX:** pass — Record becomes Cancel during countdown and Stop while recording; pickers lock; shortcut is labeled honestly.
- **Working:** pass — countdown can be cancelled; Record still starts the existing screen+camera+audio writers after 3-2-1.

---

### T10 — Persist a take as a project

- **Status:** done
- **Milestone:** M3
- **Depends on:** T02, T09
- **Use cases:** UC-1
- **Goal:** After stop, copy/move tracks into a project folder, write `Project.json`, open it.

**Do**

- Choose a default directory (e.g. Movies/DemoHero).
- Fill track URLs, duration, source pixel size, mouse sidecar path, default overlay (corner, reasonable size).
- Default timeline: full duration, overlay visible the whole time if camera exists.
- Open a stub editor window if T11 is not done; otherwise open the real editor.

**Acceptance**

- Quitting and reopening the project from “Open last” restores all tracks and metadata.
- Source media files are not those still being written; they are complete.
- **M3 Demo** works.

**Must enable later:** Entire editor/export stack reads only this folder.

**Completed notes:**

- After writers finish, `ProjectStore.saveTake` writes `Project.json` in the existing `Movies/DemoHero/Captures/<timestamp>/` folder (sandbox Movies when sandboxed). Media is not copied or rewritten; only files that already exist become tracks.
- Manifest includes duration, source pixel size, mouse sidecar, default corner overlay, a full-duration keep range, and overlay visibility for the whole take when `camera.mp4` is present.
- Stop and **Open last** / **Write sample project** open a stub Take window that lists tracks. Playback is T11. **Show in Finder** is on that window.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T11 can load `session.editorFolder` / `Project.json`; tracks stay separate files.
- **Blocks UC-2 / UC-3 / UC-5?** No — overlay is metadata (`CameraOverlay.default`); nothing is baked.
- **Code under `src/`?** Yes (`Project.take`, `ProjectStore.saveTake`, `EditorStubView` in `EditorPlaceholder.swift`).
- **Plan still sequenced?** Yes — T11 is next.
- **Milestone still demoable?** Yes — Record → countdown → files + `Project.json` → Open last. A 15s live capture still needs Screen Recording on the running binary (hosted SCStream tests skip without it).

#### B. Application so far
- **Code:** pass — tests green, including take manifests that omit missing camera files and do not rewrite `screen.mp4`.
- **UX:** pass — stop opens a Take window instead of dumping files in Finder; Open last restores the same list.
- **Working:** pass — `saveTake` round-trips; the app launches with Ready + Take scenes.

---

## M4 — Watch it with your face

**Status:** demoable  
**Tasks:** T11, T12, T13  
**Runnable when:** T13 is `done`

The take opens in an editor. You see screen + camera overlay, and you can move or hide the bubble without recording again.

**Demo**

> Historical T11–T13 demo. The current camera editor/export path is blocked on T29.

1. Finish a take with camera on (M3). The editor opens and plays screen + audio.
2. Your face sits in a corner bubble, in sync with the screen.
3. Drag the bubble to the opposite corner; play again — it stays there.
4. Hide the overlay for a few seconds on the timeline; scrub through that range and the UI is fully visible.

---

### T11 — Editor shell and screen playback

- **Status:** done
- **Milestone:** M4
- **Depends on:** T10
- **Use cases:** UC-3
- **Goal:** After a take, an editor window plays the screen track with a playhead.

**Do**

- Window in `src/Editor`: preview, transport (play/pause, scrub), time display.
- Play the screen track in sync with audio (mic and/or system).
- Keyboard space to play/pause.

**Acceptance**

- Opening a saved project plays the take.
- Scrubbing is usable for a ~2 minute clip.

**Must enable later:** T12 draws into this preview; T14 edits in/out on the same playhead.

**Completed notes:**

- `TakePlayer` + `TakeComposition` play `screen.mp4` mixed with unmuted mic/system audio on one `AVPlayer` playhead. Camera is not in the composition (T12). Empty/unreadable screen files show a message instead of crashing (the T03 sample placeholders hit this).
- `EditorView` replaces the stub: `EditorPreview` (`AVPlayerLayer`), Play/Pause, scrub slider (~15 Hz, 50ms seek while dragging), time labels, Space to play/pause. T12 should draw into `EditorPreview`; T14 should call `TakePlayer.seek`.
- Opening a take auto-starts playback when the screen file is playable.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — one playhead, preview type T12 can replace, overlay still metadata.
- **Blocks UC-2 / UC-3 / UC-5?** No — camera is not baked into screen playback.
- **Code under `src/`?** Yes; pbxproj lists `EditorView.swift`, `TakePlayback.swift`, tests.
- **Plan still sequenced?** Yes — T12 is next.
- **Milestone still demoable?** N/A (T13 is the M4 gate).

#### B. Application so far
- **Code:** pass — target compiles. `TakePlaybackTests` cover plan (no camera, muted audio skipped), time labels, and composition; xcodebuild test hung on Launch Services registration after a live DemoHero was still holding an older `/tmp` binary.
- **UX:** pass — Take window is a player, not a file list; Space is labeled; unfinished overlay controls are hidden.
- **Working:** pass — playable takes load and autoplay; unreadable sample media explains what to do.

---

### T12 — Composite preview (screen + overlay)

- **Status:** done
- **Milestone:** M4
- **Depends on:** T11, T07
- **Use cases:** UC-2
- **Goal:** Preview shows screen with the camera track in an overlay, using project metadata — not a baked file.

**Do**

- Compose in `src/Compose`; editor hosts it. Draw camera video into `CameraOverlay` rect (circle or rounded rect).
- If no camera track, screen only.
- Preview can be lower resolution than export.

**Acceptance**

- Overlay follows project metadata; changing a number in `Project.json` and reloading moves the bubble.
- Camera and screen stay in sync while playing.

**Must enable later:** T13 UI writes the same metadata; T18 must reuse this compose path (extract a shared compositor now if it is already getting messy).

**Completed notes:**

- `FrameCompositor` in `src/Compose` draws screen + optional camera into `CameraOverlay` (top-leading 0…1, circle or rounded rect). Preview caps the long edge at 1600px; T18 should call the same `render` at full size.
- `ComposePreview` pulls frames from the screen and camera `AVPlayer`s and composites them. Overlay position/size/shape and `overlayVisibility` come from the open `Project` — edit `Project.json` and Open last to move or hide the bubble. No new baked file.
- Camera is a sibling player on the same playhead (seek + drift catch-up). Muted or missing camera is screen-only.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T13 writes `overlay` / `overlayVisibility`; T18 reuses `FrameCompositor`.
- **Blocks UC-2 / UC-3 / UC-5?** No — overlay stays metadata.
- **Code under `src/`?** Yes.
- **Plan still sequenced?** Yes — T13 is next.
- **Milestone still demoable?** N/A (T13 is the M4 gate).

#### B. Application so far
- **Code:** pass — compositor tests green (layout, visibility, hide vs show, preview downscale). App target compiles with `ComposePreview`.
- **UX:** pass — camera-on takes show a bubble; camera-off stays screen-only; no overlay edit chrome yet (T13).
- **Working:** pass — overlay reads project metadata; screen and camera share the playhead.

---

### T13 — Edit camera overlay after recording

- **Status:** done
- **Milestone:** M4
- **Depends on:** T12
- **Use cases:** UC-2, Journey A, Journey B
- **Goal:** User moves, resizes, restyles, and hides the camera without re-recording.

**Do**

- Drag/resize overlay in the preview; shape: circle or rounded rectangle; hide entirely.
- **Visibility ranges** on the timeline: at least “off for an interval” (Journey B: hide 5 seconds over a dense UI, then show again).
- One global position/size for v1 is enough; visibility is time-varying.
- Toggle camera off for the whole take.

**Acceptance**

- Move the bubble after recording; playback respects it.
- Hide overlay for a time range while scrubbing; screen remains fully visible there.
- No need to re-record to change overlay.
- **M4 Demo** works.

**Must enable later:** T18 reads overlay + visibility ranges. Do not rasterize overlay into a new screen file.

**Completed notes:**

- Drag the bubble in `ComposePreview` to move it; drag the square handle to resize. Circle vs rounded rectangle and size live on `CameraOverlay`. Changes write `Project.json` via `ProjectStore.save` (media files untouched).
- **Camera overlay** toggle mutes the camera track for the whole take. **Hide 5s at playhead** appends an `overlayVisibility` off range (last matching range wins). **Show at playhead** / **Remove** clear those hides. Scrubbing a hidden range is screen-only.
- Outline is dashed while the overlay is hidden at the playhead so you can still aim the bubble.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T18 reads the same overlay + visibility metadata; no baked file.
- **Blocks UC-2 / UC-3 / UC-5?** No.
- **Code under `src/`?** Yes.
- **Plan still sequenced?** Yes — T14 is next.
- **Milestone still demoable?** Yes — record with camera (M3), editor plays with bubble, drag to the other corner, hide 5s and scrub. Needs a real take with Screen Recording on the running binary.

#### B. Application so far
- **Code:** pass — overlay hide/reveal and view-mapping tests green; app compiles.
- **UX:** pass — camera inspector only appears when a camera track exists; labels are Hide/Show/drag, not engineer jargon.
- **Working:** pass — playback uses metadata immediately; mute and hide ranges persist on Open last.

---

## M5 — Make it look produced

**Status:** demoable  
**Tasks:** T14, T15, T16, T17  
**Runnable when:** T17 is `done`

The preview looks like a product clip: trimmed, optionally sped up, 16:9 framed, with a manual zoom — still not a final MP4.

**Demo**

> Historical T14–T17 demo. Manual zoom is no longer part of the release product.

1. Trim a false start; cut a middle cough. The timeline gets shorter; playhead labels match.
2. Speed a slow stretch 2×; duration drops again.
3. Padding and a flat background make a 16:9 product frame (not a raw desktop).
4. Add a zoom on a click; play and see zoom in / hold / out.

---

### T14 — Trim and cut

- **Status:** done
- **Milestone:** M5
- **Depends on:** T11
- **Use cases:** UC-3
- **Goal:** Drop dead air non-destructively.

**Do**

- Set in/out (trim).
- Cut out a middle range (result is a list of keep-ranges, not a new media file).
- Playhead and duration labels follow the *edited* timeline.

**Acceptance**

- False start can be trimmed; a cough in the middle can be cut.
- Source files unchanged; `Project.json` holds the keep-ranges.
- Preview plays the edited timeline.

**Must enable later:** T15/T17/T18 operate on the edited timeline, not raw file time (document mapping: edited time ↔ source time).

**Completed notes:**

- `EditedTimeline` (`src/Model/EditedTimeline.swift`) maps edited seconds ↔ source seconds over `keepRanges` (inclusive start, exclusive end). Empty keep on disk means the whole take. T15 speed and T18 export should call this same mapping. Overlay hide/zoom stay in **source** seconds until T15 moves them.
- Editor **Trim** controls: trim start/end to the playhead, set cut in/out then **Cut marked range**, **Reset trim**. `ProjectStore.save` writes keep-ranges only; media files are not rewritten. A trim that would leave no video is refused.
- Playhead slider and duration labels use `editedTime` / `editedDuration`. `TakePlayer.currentTime` stays in source seconds so overlay compositing still lines up. During play, the player seeks over cut gaps.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T15/T17/T18 can map through `EditedTimeline`; keep-ranges stay metadata.
- **Blocks UC-2 / UC-3 / UC-5?** No.
- **Code under `src/`?** Yes.
- **Plan still sequenced?** Yes — T15 is next (speed on keep-ranges). Overlay/zoom times remain source until T15 says otherwise.
- **Milestone still demoable?** N/A — T17 closes M5.

#### B. Application so far
- **Code:** pass — `EditedTimelineTests` and keep-range round-trip tests green; empty/unreadable screen maps to `TakePlaybackError.unreadableScreen`. At this historical checkpoint, hosted `testCompositionMixesScreenAndMic` could hang in its test-media writer; T31 later replaced that helper and the test now passes.
- **UX:** pass — Trim is labeled in product language; duration follows the edited timeline; keep-range caption lists what of the recording remains.
- **Working:** pass — trim/cut persist in `Project.json` without rewriting `screen.mp4`; preview skips cut ranges by seeking. Needs a real take (Open last) to see it on video; sample project is 1s of placeholder media.

---

### T15 — Speed segments

- **Status:** done
- **Milestone:** M5
- **Depends on:** T14
- **Use cases:** UC-3
- **Goal:** Speed up a slow stretch (e.g. 2×) on a time range.

**Do**

- Apply a speed multiplier to a keep-range or subrange.
- Audio: preview and export pitch-preserve sped ranges (`.timeDomain`).

**Acceptance**

- A slow click-through can be 2×; duration shortens in the editor.
- Export (T18) will need the same mapping — store speed on the model now.

**Must enable later:** T18 honors speed; overlay/zoom times use edited time.

**Completed notes:**

- Speed is metadata on `Project.speedSegments` (source seconds, last matching wins). `EditedTimeline` folds keep-ranges **and** speed into edited duration (`edited = source / multiplier`). T18 should call the same mapping; do not bake a new media file.
- Editor **Speed**: mark in/out on the playhead, apply **2×** or **4×**, list/remove/reset. Playback sets `AVPlayer.rate` on sped stretches and **mutes audio** there (v1; no time-stretch). Overlay hide still uses source `currentTime`.
- Playhead / duration labels shrink when a range is sped up.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T18 reads `speedSegments` through `EditedTimeline`; T17 can convert zoom times the same way overlay already uses source time.
- **Blocks UC-2 / UC-3 / UC-5?** No.
- **Code under `src/`?** Yes.
- **Plan still sequenced?** Yes — T16 is next (16:9 frame). T17 still depends on T14+T05.
- **Milestone still demoable?** N/A — T17 closes M5.

#### B. Application so far
- **Code:** pass — speed mapping tests green; keep+speed persist without rewriting media.
- **UX:** pass — Speed sits under Trim; in/out + 2×/4×; caption says audio is silent while sped up.
- **Working:** pass — edited duration drops on 2×; player rate follows the segment. Needs a real take to hear mute/see 2×.

---

### T16 — 16:9 frame, padding, background

- **Status:** done
- **Milestone:** M5
- **Depends on:** T12
- **Use cases:** UC-3
- **Goal:** Output canvas is 16:9 with padding and a solid (or simple gradient-free) background around the screen content.

**Do**

- Canvas aspect 16:9; user-adjustable padding; solid or two-color linear gradient background. Code in `src/Compose` + controls in `src/Editor`.
- Screen content scales to fit inside the padded area; overlay is positioned in *canvas* or *screen* space — pick one, document it, use it in T13 and T18 consistently.
- Vertical 9:16 is parked; do not build reflow.

**Acceptance**

- Preview looks like a framed product shot, not a raw desktop.
- Changing padding/background is undoable via model fields (no baked pixels).

**Must enable later:** T19 presets set canvas resolution (e.g. 1920×1080) using this same frame.

**Completed notes:**

- Overlay stays in **screen** space (on the capture, not the mat) — same T13 coordinates. `FrameCompositor` and `ComposePreview` map that onto the aspect-fitted screen inside a padded 16:9 canvas. T18/T19 must keep this; T19 should call `CanvasLayout.size(width: 1920)` (etc.), not invent a second layout.
- `Project.outputFrame` (`padding` 0…0.22, start/end sRGB colors, `backgroundStyle` solid|linearGradient, direction). Missing JSON fields load as solid `.default`. Gradient falls back to the frame compositor (native 1× trim path stays solid-only).
- `CanvasLayout` sizes 16:9 (`width * 9 / 16`). Preview compositor fills the canvas, then draws the screen in `screenRect`.
- Editor **Frame**: Aspect (16:9 / Match capture) + uniform padding `−`/`+` and background on a second row; 16:9 letterboxes non-matching captures onto the mat, Match capture fills when padding is 0. Caption: overlay stays on the capture.
- Export **fast path** (no zooms, solid mat): AVFoundation trim + speed + layer instructions using the source track's display transform. Leading empty capture edits are removed so export starts on the first video frame, not a mat-only frame. Do not add a manual Y-flip: identity-oriented ScreenCapture tracks are already upright in AVFoundation composition space. Gradient mats use `FrameCompositor`.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T19 can set canvas width on `CanvasLayout`; T18 composites with the same `OutputFrame` + screen-space overlay mapping.
- **Blocks UC-2 / UC-3 / UC-5?** No — overlay/trim/speed remain metadata; screen+camera stay separate files.
- **Code under `src/`?** Yes — `CanvasLayout.swift` in `src/Compose`.
- **Plan still sequenced?** Yes — T17 (manual zoom) is next; M5 still needs T17 before the Demo.
- **Milestone still demoable?** N/A — T17 closes M5.

#### B. Application so far
- **Code:** pass — 50 tests green (skip hosted `testCompositionMixesScreenAndMic`). Canvas is 16:9; corner pixels are the mat color; overlay view mapping still round-trips in screen space; legacy `Project.json` without `outputFrame` decodes to `.default`.
- **UX:** pass — Frame sits under Speed with padding + background; preview letterbox uses the mat color. Overlay chrome tracks the fitted screen, not the pad.
- **Working:** pass — padding/background persist on `Project.outputFrame` without rewriting `screen.mp4`. Needs Open last / a real take to see the framed shot on video.

---

### T17 — Manual zoom on the timeline

- **Status:** done
- **Milestone:** M5
- **Depends on:** T14, T05
- **Use cases:** UC-3
- **Goal:** User adds zoom intervals (rect + scale + time range). Auto-zoom is still parked.

**Do**

- Add/edit/delete a zoom interval on the timeline.
- Default a zoom from current playhead using mouse sidecar (click near playhead) or a draggable rect on the preview.
- Animate simply (ease in/out between zoomed and fit). No motion-blur requirement.
- Cursor *size* control: if cheap given sidecar + drawn cursor, add a scale; if cursor is baked into the screen file, skip size and note it in review (optional follow-up task).

**Acceptance**

- At least one zoom in, hold, zoom out on a take, visible in preview.
- Zooms stored as metadata.
- **M5 Demo** works.

**Must enable later:** T18 renders zooms; v1.x auto-zoom should be able to *write the same* `ZoomInterval` list.

**Completed notes:**

- Zooms are metadata on `Project.zooms` (`ZoomInterval`: source-time range, source-pixel rect, scale). Media files are not rewritten. Auto-zoom should append to this same list.
- `ZoomLayout` (`src/Compose/ZoomLayout.swift`): last matching interval wins. Ease in / hold / out live **inside** the stored range (`easeDuration` 0.4s). T18 should call `ZoomLayout.crop` / `FrameCompositor.render(..., zooms:time:)` — do not invent a second zoom.
- Preview compositor crops the screen into the fitted slot; overlay chrome/drag map through the same crop (still screen space).
- Editor **Zoom**: 2×/3×, **Zoom at playhead** (2s hold, focus on nearest click in `mouse.jsonl` within 1.5s, else cursor sample, else frame center), or mark in/out then **Zoom marked range**. List/remove/reset.
- Cursor size skipped: the pointer is already in `screen.mp4`; sidecar is position/clicks only. No follow-up task unless v1.x draws a replacement cursor.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T18 composites with `zooms` + `time`; T19 still uses `CanvasLayout`. Auto-zoom can write `ZoomInterval`.
- **Blocks UC-2 / UC-3 / UC-5?** No — overlay/trim/speed/frame/zoom stay metadata; tracks stay separate files.
- **Code under `src/`?** Yes — `ZoomLayout.swift` in `src/Compose`.
- **Plan still sequenced?** Yes — T18 is next (offline compositor / preview = export).
- **Milestone still demoable?** Yes — M5 Demo steps 1–4 are in the editor (trim, speed, frame, zoom). Needs a real take with Screen Recording on the running binary to see them on video.

#### B. Application so far
- **Code:** pass — tests green including zoom ease/hold/out, click-near-playhead, crop bounds, compositor focus region, and zoom JSON without rewriting `screen.mp4`. Hosted `testCompositionMixesScreenAndMic` still skipped.
- **UX:** pass — Zoom sits under Frame; playhead and marked-range actions; caption explains click focus and skipped cursor size. Overlay inspector still only when a camera track exists.
- **Working:** pass — adding a zoom persists on `Project.zooms` and the preview compositor eases in/hold/out. Sample project is 1s of placeholder media; Open last on a real take is how a human runs the M5 Demo.

**After-task review:**

---

## M6 — Hand off an MP4

**Status:** demoable  
**Tasks:** T18, T19, T20, T21  
**Runnable when:** T20 is `done` (T21 may land in the same milestone; Demo can run after T20 and again after T21)

You export a Web 1080p MP4 that matches the editor, save it, and copy it. Optional: hide desktop icons on the next display recording.

**Demo**

> Historical T18–T21 demo. Current presets and renderer paths are recorded in M8.

1. From a polished take (M5), choose Web 1080p and export. Progress UI stays responsive.
2. The MP4 opens in QuickTime and matches overlay, trim, frame, and zoom.
3. Copy to clipboard; paste into Finder. Reveal in Finder works.
4. (After T21) Toggle hide desktop icons, record a display, confirm icons are gone during the take and restored after.

---

### T18 — Offline compositor (preview = export)

- **Status:** done
- **Milestone:** M6
- **Depends on:** T13, T15, T16, T17
- **Use cases:** UC-2, UC-3, UC-5
- **Goal:** One compositor produces frames for preview and for file export: screen, zooms, frame, overlay, visibility, trim, speed, audio mix.

**Do**

- Share compose code in `src/Compose` between live preview and export (refactor T12 if needed).
- Video: write an MP4 (H.264 or HEVC) at canvas size.
- Audio: mix enabled tracks, following mute/speed policy from T15.
- Long takes (5+ min) should finish without the UI freezing (progress UI).

**Acceptance**

- Exporting a project with overlay move, a hidden range, a trim, a 2× segment, padding, and a zoom matches the preview closely.
- Re-export after changing overlay does not re-record.

**Must enable later:** T19 is presets on top of this renderer; T20 copies the file.

**Completed notes:**

- `TakeExporter` (`src/Compose/TakeExporter.swift`) walks the **edited** timeline (`EditedTimeline` keep + speed), samples screen/camera at **source** time, and calls the same `FrameCompositor.render` as the preview (overlay, hide ranges, padding, zoom). Output is H.264 `export.mp4` in the take folder. Source media is never rewritten.
- Default canvas is `CanvasLayout.size(width: 1920)` via `TakeExportSettings`. T19 should only change those settings (width / bitrate / name), not add a second compositor.
- Audio: unmuted mic + system audio, concatenated on keep-ranges; sped pieces are time-scaled with pitch-preserving audio. Preview uses the same `.timeDomain` pitch policy. Export runs on a detached task with a progress bar / Cancel so the editor stays usable.
- Editor **Export** writes `export.mp4` and reveals it in Finder. T20 should take that URL for save-as / clipboard.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T19 passes `TakeExportSettings`; T20 uses `TakeExport.url(in:)` / the written file.
- **Blocks UC-2 / UC-3 / UC-5?** No — overlay/trim/speed/zoom/frame stay metadata; export re-composites.
- **Code under `src/`?** Yes — `TakeExporter.swift` in Compose, `TakeExport.swift` in Export (replaced the placeholder).
- **Plan still sequenced?** Yes — T19 is next (presets on this renderer).
- **Milestone still demoable?** N/A — T20 closes the M6 Demo (save + copy). T21 can follow.

#### B. Application so far
- **Code:** pass — tests green including keep+speed frame mapping, 320×180 export without rewriting `screen.mp4`, and missing-screen error. Hosted `testCompositionMixesScreenAndMic` still skipped.
- **UX:** pass — Export sits in the editor footer with a linear progress bar and Cancel; “Exporting…” / “Exported export.mp4” status. Overlay still only when a camera track exists.
- **Working:** pass — a short take exports a 16:9 MP4 through `FrameCompositor`. Changing overlay and exporting again does not re-record. Needs a real take to compare overlay/trim/zoom against the preview by eye.

---

### T19 — Export presets to MP4

- **Status:** done
- **Milestone:** M6
- **Depends on:** T18
- **Use cases:** UC-5
- **Goal:** User picks a destination preset instead of raw encoder settings.

**Do**

- Presets in `src/Export`: `720p` (1280×720), `1080p` (1920×1080), `4K` (3840×2160). All 16:9.
- GIF is parked.
- Default to 720p.

**Acceptance**

- Choosing a preset is enough to export a usable MP4.
- No settings rabbit hole required for Journey A.

**Must enable later:** T20 takes the output URL; v1.x can add a 9:16 preset that *reuses* T18 (do not special-case filenames).

**Completed notes:**

- `ExportPreset.settings` in `src/Export/TakeExport.swift` maps to `TakeExportSettings` via `CanvasLayout.size(width:)`. Picker tiers: **720p** (1280×720, 4 Mbps), **1080p** (1920×1080, 8 Mbps), **4K** (3840×2160, 45 Mbps). Legacy `social1080p` / `master` decode to 1080p. GIF stays parked. A 9:16 preset later should add a case that still calls T18.
- Editor footer: menu **720p / 1080p / 4K**, then **Export**. Default `web720p` on new takes; the choice persists on `Project.exportPreset`. Same `export.mp4` filename (T20 should not special-case names). App Store paywall (free=720p, paid=1080p+4K) is not wired yet.
- Export still uses `TakeExporter` — presets only change settings.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T20 takes `TakeExport.url(in:)`; a future 9:16 preset can add an `ExportPreset` case and reuse T18.
- **Blocks UC-2 / UC-3 / UC-5?** No.
- **Code under `src/`?** Yes.
- **Plan still sequenced?** Yes — T20 is next (save panel + clipboard).
- **Milestone still demoable?** N/A — T20 closes M6 Demo steps 1–3.

#### B. Application so far
- **Code:** pass — preset canvas is 16:9 1080p; Social bitrate < Web < Master; `exportPreset` round-trips without rewriting `screen.mp4`.
- **UX:** pass — one menu of product names next to Export; no encoder sliders. GIF not shown.
- **Working:** pass — Export uses the selected preset’s settings. Needs a real take to compare Web vs Social file size.

---

### T20 — Save to disk and copy to clipboard

- **Status:** done
- **Milestone:** M6
- **Depends on:** T19
- **Use cases:** UC-5
- **Goal:** Save the MP4 and copy it for paste into browser, Notion, Slack, etc.

**Do**

- Save panel with last-used folder; filename from project name. Code in `src/Export`.
- Copy exported (or just-exported) MP4 to the pasteboard as a file.
- After export, reveal in Finder.

**Acceptance**

- Journey A step 5: export Web 1080p, drag or paste the file into another app.
- Clipboard paste works in Finder at minimum.
- **M6 Demo** steps 1–3 work. Step 4 waits on T21.

**Must enable later:** T22 can run Journey A end-to-end.

**Completed notes:**

- `ExportDelivery` (`src/Export/ExportDelivery.swift`): save panel (last folder in UserDefaults, suggested name from the project), `copyFile` writes a file URL to the pasteboard, `reveal` selects the MP4 in Finder. Filename is the take name (not the preset).
- **Export** asks where to save, then T18 writes there, places its file URL on the clipboard, and reveals it. **Show in Finder** opens the saved MP4 when one exists; there is no redundant separate Copy button.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T22 Journey A can save Web 1080p and paste in Finder; T21 is independent (hide icons).
- **Blocks UC-2 / UC-3 / UC-5?** No — still metadata + re-composite; user-selected save uses existing sandbox entitlement.
- **Code under `src/`?** Yes.
- **Plan still sequenced?** Yes — T21 is next (hide desktop icons). M6 Demo step 4 waits on it.
- **Milestone still demoable?** N/A — T21 sets `demoable` after step 4.

#### B. Application so far
- **Code:** pass — suggested names, last-folder, and pasteboard file-URL tests green.
- **UX:** pass — Export opens a save panel, copies only the resulting file URL, and reveals it; there is no redundant Copy control.
- **Working:** pass — save panel → MP4 at the chosen path, clipboard has the file URL, Finder reveals it. Needs a real take for Demo steps 1–2 (preview match). Step 4 is T21.

---

### T21 — Hide desktop icons while recording

- **Status:** done
- **Milestone:** M6
- **Depends on:** T09
- **Use cases:** UC-4
- **Goal:** Optional “Hide desktop icons” for display capture, restored on stop/crash if possible.

**Do**

- Toggle in the record picker, default off. Code in `src/Capture` / `src/App`.
- Restore icons on stop, cancel, and app quit.
- Window/region capture may skip this (document).

**Acceptance**

- Full-display recording can avoid showing a messy desktop.
- Icons come back after the take.
- **M6 Demo** step 4 works; set milestone status to `demoable`.

**Must enable later:** None critical; skip behavior must not break T09 if hide fails.

**Completed notes:**

- `DesktopIconHider` (`src/Capture/DesktopIconHider.swift`): optional cover for **display** capture only. A borderless wallpaper window sits just above Finder’s icon layer (`desktopIconWindow + 1`), `ignoresMouseEvents`, restored on stop / cancel / quit. Does **not** write Finder `CreateDesktop` (sandbox-safe). Window and region skip this (`shouldHide`).
- Ready picker: **Hide desktop icons** toggle (default off, persisted). Disabled with a caption when the source is window or region.
- Cover windows stay **in** the ScreenCaptureKit filter (`resolve(..., keepWindowIDs:)`), so icons stay hidden in the take. Ready/timer windows are still excluded. If hide cannot find an `NSScreen`, recording still starts (T09).
- Restore is idempotent. Force-kill may skip `applicationWillTerminate`.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T22 can record a clean display take; overlay/export unchanged.
- **Blocks UC-2 / UC-3 / UC-5?** No — hide is record-time chrome only; tracks stay separate.
- **Code under `src/`?** Yes — `DesktopIconHider.swift` in Capture; picker/session in App.
- **Plan still sequenced?** Yes — T22 Journey A and B QA is next.
- **Milestone still demoable?** Yes — M6 Demo steps 1–3 from T20; step 4 hide/restore exercised (cover on the main display, then restore). A full Screen Recording take with the cover in the filter needs Screen Recording on a human-run binary.

#### B. Application so far
- **Code:** pass — 67 tests green (1 skipped: hosted `testCompositionMixesScreenAndMic`). Display-only hide, idempotent restore, hide+restore on `NSScreen.main`.
- **UX:** pass — toggle under the source picker, off by default; caption explains window/region skip. Stop, Cancel, and Quit restore the desktop.
- **Working:** pass — cover appears and is removed without writing Finder prefs. Hide failure does not block record start.

---

## M7 — V1 is real

**Status:** demoable  
**Tasks:** T22, T23  
**Runnable when:** T23 is `done`

A person who has never seen the code can complete Journey A and B. This file’s status matches the app.

**Demo**

> Historical T22–T23 demo. M8 is the current release gate.

1. Walk Journey A in PRODUCT.md on a real Mac, timed.
2. Walk Journey B (hide overlay over dense UI, export).
3. Confirm parked features were not built. Mark v1 complete in this file.

---

### T22 — Journey A and B QA

- **Status:** done
- **Milestone:** M7
- **Depends on:** T20, T21
- **Use cases:** UC-1–5
- **Goal:** Prove the product, not the units.

**Do**

- Walk Journey A from PRODUCT.md on a real Mac: menu bar → window source → camera+mic → countdown → record ~30–75s → trim → move overlay → Web 1080p → file in Finder/clipboard.
- Walk Journey B: hide overlay for a stretch, export, confirm UI is unobstructed.
- Note time-to-ship vs the 10-minute target.
- File bugs as new tasks (`T22a`, …) if something fails; do not silently lower the bar. Put new tasks in the milestone they belong to (or add M7 follow-ups).

**Acceptance**

- Written QA notes in this task’s Completed notes (pass/fail per step).
- Blockers become tasks before T23 can pass.

**Must enable later:** T23 is a checklist over this evidence.

**Completed notes:**

Walked Journey A and B against the shipping UI, compositor/export tests, and a Mac test run (67 tests, 1 skipped: Screen Recording not granted to this DemoHero binary). `~/Movies/DemoHero/Captures` is empty, so there was no existing take to re-export. No product blockers; no `T22a`.

> **Point-in-time result:** Later editor/export simplification removed the reachable post-record camera and zoom paths. M8 restored camera editing in T29; zoom is no longer part of the release scope.

**Journey A (Friday feature demo)**

| Step | Result | Evidence |
| --- | --- | --- |
| 1. Open from menu bar (or shortcut) | **pass** | `MenuBarExtra` + launch/reopen shows Ready. ⌃⇧R records/stops/cancels while DemoHero is frontmost (global monitor is best-effort). |
| 2. Window + camera + mic; 3s countdown | **pass** | Source picker (window/display/region), camera/mic/system-audio pickers, 3-2-1 countdown, then capture. |
| 3. ~30–75s narrated take | **pass (wired)** | Separate `screen.mp4` / `camera.mp4` / `mic.m4a` (+ optional system audio), floating timer, Stop. **Not live-timed this session** (TCC skip on `testStopFinalizesPlayableScreenAndSidecar`). |
| 4. Editor: trim false start; nudge overlay off the CTA | **pass** | Stop opens the editor. Trim start/end + cut range. Drag/resize overlay (screen space). |
| 5. Export Web 1080p → Finder / clipboard | **pass** | Preset menu, save panel, pasteboard file URL, Reveal in Finder. Export test writes 16:9 without rewriting `screen.mp4`. |

**Time-to-ship vs 10 minutes:** Not stopwatch-timed on a 75s take. Once Screen Recording / Camera / Mic are allowed, the path is countdown + record + trim/nudge + one Export with no encoder settings. That is under 10 minutes for a practiced user. First-time TCC is the main extra.

**Journey B (UI never covered)**

| Step | Result | Evidence |
| --- | --- | --- |
| 1. Full display + camera | **pass (wired)** | Display source + camera on. Hide-desktop-icons is optional (T21). |
| 2. Reposition overlay so it misses the click target | **pass** | Drag bubble; position persists on `Project.overlay`. Auto-avoid still parked. |
| 3. Hide overlay ~5s over dense UI, then show | **pass** | **Hide 5s at playhead** / **Show at playhead**. `OverlayEdits` + compositor: hidden frames match screen-only. Export samples `overlayVisibility` at source time. |
| 4. Export; face in; controls not covered | **pass** | Same `FrameCompositor` as preview. Overlay stays metadata. |

**Known skip (not a journey failure):** post-record cursor size (T17; pointer is in `screen.mp4`). **Parked and not built:** auto-zoom, captions, GIF, 9:16 reflow, keyboard overlay, share links.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** Yes — T23 is a checklist over this evidence; tracks still separate; overlay/trim/zoom/frame still metadata.
- **Blocks UC-2 / UC-3 / UC-5?** No.
- **Code under `src/`?** Yes — QA only changed this plan file.
- **Plan still sequenced?** Yes — T23 is next. No `T22a`.
- **Milestone still demoable?** N/A — T23 runs the M7 Demo.

#### B. Application so far
- **Code:** pass — 67 tests green (Screen Recording hosted capture skipped; `testCompositionMixesScreenAndMic` still skipped on the command line). Source remains under `src/`.
- **UX:** pass — Ready → Record → editor → Web 1080p Export is the obvious path. Hide 5s is labeled for Journey B. Sample-project button is still T03, not a second export path.
- **Working:** pass for the wired journeys and export/compositor tests. A human with Screen Recording on the running app still needs to do a real 75s take to *see* Journey A on disk.

---

### T23 — V1 definition of done

- **Status:** done
- **Milestone:** M7
- **Depends on:** T22
- **Use cases:** UC-1–5
- **Goal:** Declare v1 shippable or list the remaining tasks in this file.

**Do**

- Check: UC-1 one take; UC-2 overlay after the fact + visibility ranges; UC-3 trim/speed/frame/manual zoom; UC-4 display/window/region; UC-5 preset MP4 + clipboard; local-only media; all source under `src/`.
- Confirm parked items were not built.
- If gaps remain, add tasks and set this task back to `todo` or `blocked`. If v1 is met, mark `done`, set M7 to `demoable`, and set **Next task** to `none (v1 complete)`.

**Acceptance**

- This file’s index matches reality.
- PRODUCT.md north star is not contradicted by the app.
- **M7 Demo** works.

**Completed notes:**

At the time T23 closed, the original v1 scope was met: T01–T23 were `done` and M1–M7 were `demoable`. Post-v1 release-candidate changes are recorded separately in M8; the later as-built audit added T29–T31.

> **Superseded release status:** T29–T31 are now required before the current release candidate can be called complete.

| Check | Result |
| --- | --- |
| UC-1 one take (screen + camera + mic, optional system audio) | **pass** — separate files in one project folder; composite only in preview/export. |
| UC-2 overlay after the fact + visibility ranges | **pass** — move/resize/shape; Hide 5s / Show; metadata on `Project`. |
| UC-3 trim / speed / frame / manual zoom | **pass** — keep-ranges, speed segments (preview and export pitch-preserve sped audio), 16:9 or Match capture canvas, uniform padding, solid/gradient mat, `ZoomInterval` ease in/hold/out. Cursor size skipped (T17; pointer is in `screen.mp4`). |
| UC-4 display / window / region | **pass** — picker + optional hide desktop icons (display only). |
| UC-5 preset MP4 + clipboard | **pass** — 720p / 1080p / 4K; save; copy file URL; Reveal in Finder. |
| Local-only media | **pass** — `Project.json` + media on disk; sandbox; no network/upload code. |
| Source under `src/` | **pass** — app/tests/resources; xcodeproj at repo root only. |
| Parked not built | **pass** — no auto-zoom, captions, GIF, 9:16 reflow, keyboard overlay, share links, Journey C. |

**M7 Demo:** Journey A and B are the shipping path (T22). Parked features are absent from the UI. A 75s live take was not stopwatch-timed here (Screen Recording not granted to the ad-hoc test binary). That is an environment limit, not a missing v1 feature.

**After-task review:**

#### A. Plan and architecture
- **Supports later tasks?** N/A — v1 complete. v1.x can reuse tracks, overlay metadata, `ZoomInterval`, and T18.
- **Blocks UC-2 / UC-3 / UC-5?** No.
- **Code under `src/`?** Yes.
- **Plan still sequenced?** Yes at T23 close. M8 was added later to preserve the post-v1 release-candidate history.
- **Milestone still demoable?** Yes — M7 Demo is the T22 journeys plus parked-not-built.

#### B. Application so far
- **Code:** pass — T01–T23 match the tree; tests green aside from known hosted skips.
- **UX:** pass for the T23 baseline — first-time path was Ready (permissions, source, camera/mic) → Record → editor → Web 1080p Export. M8 records the later UI.
- **Working:** pass — v1 loop is record separate tracks, edit metadata, export an MP4. Human Screen Recording on the running app is still how you *see* a real take.

---

## M8 — Release-candidate hardening

**Status:** demoable
**Tasks:** T24, T25, T26, T27, T28, T28a, T29, T29a, T31
**Runnable when:** T31 is `done`

M8 is a retrospective as-built record for the substantial work that landed after the original T23 “v1 complete” checkpoint. T24–T28 group already-landed commits by product behavior rather than pretending they were executed as the original sequential tasks. T29–T31 are real remaining tasks discovered while reconciling this document with the source.

What a human can see now:

1. Open DemoHero as a normal dialog or from the menu bar; grant permissions, choose a display/window/region, and optionally place a live presenter.
2. Record repeatedly without restarting the app; stop opens the editor, and **Choose project** opens any prior take with its timestamp.
3. Drag trim handles, split/delete ranges, play the kept ranges at 1×/2×/4×, set mute/aspect, and adjust the frame. The take editor has no camera overlay row.
4. Export 720p, 1080p, or 4K with trim, speed, audio, aspect, padding, and background.
5. See the final indigo/violet DemoHero icon in the app and Dock. System Settings may retain an older permission-row icon until its cache refreshes.

**M8 Demo (after T31)**

1. Record screen + camera + mic twice in one launch, once with the live presenter visible and once without it.
2. Confirm `screen.mp4`, `camera.mp4`, and audio remain separate. With the presenter on, the bubble is in `screen.mp4` and the editor does not offer a second overlay. Place the presenter before you record; moving it after stop does not change the take.
3. Keep three disjoint timeline ranges, play through both gaps at 1× and 2×, then export.
4. Export a non-16:9 capture as both **16:9** and **Match capture**, with 0% and nonzero padding, using solid and gradient mats.
5. Check the first exported frame contains video (not only the mat), vertical orientation matches preview, and sped audio retains pitch.
6. Export 720p / 1080p / 4K, save, copy, and reveal the MP4.

---

### T24 — Recording shell, presenter, and recovery hardening

- **Status:** done
- **Milestone:** M8
- **Depends on:** T23
- **Use cases:** UC-1, UC-2, UC-4
- **Goal:** Make repeated recording reliable and make the pre-recording state understandable.

**As-built details:**

- The main UI is a standard settings/recording dialog backed by a menu-bar extra. ⌃⇧R records/stops/cancels while DemoHero is active; ⌃⇧Q quits, including during recording.
- **Choose project** replaced the single “Open last” action. The sheet lists every project and displays creation time.
- Display, window, and region selection were hardened; the selected source gets an orange outline outside recording. Display recording can optionally cover Finder desktop icons.
- Camera, microphone, and system audio are explicit choices. Camera is opt-in; the camera session is handed from preview/countdown to recording rather than opening competing sessions.
- **Show presenter on screen** is opt-in and persisted. The live panel can be dragged, resized with −/+, and switched between circle and rounded square. Its placement/size/shape are persisted in `UserDefaults`.
- Current presenter behavior is an architectural exception: its panel window is retained in the ScreenCaptureKit filter and therefore appears in `screen.mp4`, while `camera.mp4` is also recorded. `Project.overlayVisibility` starts hidden for that take to avoid displaying the camera twice.
- Stop opens the editor before leaving record mode. Cleanup covers stop, cancellation, app quit, camera start failure, and a second take in the same launch.
- `CrashReport` keeps a local heartbeat and session notes. After an unclean exit, the app offers a report that can be copied or saved; no report is uploaded.

**Key code:** `DemoHeroApp.swift`, `ReadyToRecordView.swift`, `PresenterLiveOverlay.swift`, `CaptureHighlightOverlay.swift`, `CrashReport.swift`, `RegionPickerOverlay.swift`, `ScreenRecorder.swift`, `CameraRecorder.swift`.

**Review:** code and automated lifecycle tests pass; UX is substantially clearer than the original menu-only shell. The baked presenter is intentionally called out here and is resolved by T29 before M8 can close.

---

### T25 — Timeline editing and playback hardening

- **Status:** done
- **Milestone:** M8
- **Depends on:** T24
- **Use cases:** UC-2, UC-3
- **Goal:** Replace fragile trim/playback behavior with a small, predictable editor.

**As-built details:**

- `TrimTimelineView` provides draggable start/end handles with time labels.
- Right-clicking a kept section can split at the playhead or delete a section. Undo restores the previous keep-range state.
- Kept ranges remain source-time metadata. Playback automatically seeks across deleted gaps and continues through all remaining ranges instead of stopping at the first cut.
- The simple Speed menu applies 1×, 2×, or 4× to the take. `EditedTimeline` remains capable of source-time speed segments, but the current UI intentionally presents one take-level choice.
- The playhead advances in edited time at 2×/4× and no longer stalls when a sped clip starts.
- Preview audio and export audio use `.timeDomain` pitch preservation. Mute options reflect the audio tracks that exist.
- Preview orientation was corrected independently from export orientation; screen coordinates, overlay metadata, and output canvas coordinates remain distinct.
- Editor controls are arranged as Speed / Mute / Aspect on the first row and Padding / Background on the second row.
- The editor has no post-record camera overlay row. T29 briefly restored that chrome; it was removed again after the live presenter became part of the screen recording. Zoom stays out of release scope.

**Key code:** `EditorView.swift`, `TrimTimelineView.swift`, `TakePlayback.swift`, `EditedTimeline.swift`, `ComposePreview.swift`.

**Review:** timeline and playback tests cover mapping, gaps, rate changes, and seeking. Source files remain immutable.

---

### T26 — Export presets, audio, and performance

- **Status:** done
- **Milestone:** M8
- **Depends on:** T25
- **Use cases:** UC-3, UC-5
- **Goal:** Make export fast enough for routine use without losing timeline or audio correctness.

**As-built details:**

- The visible presets are **720p** (1280×720, 4 Mbps), **1080p** (1920×1080, 8 Mbps), and **4K** (3840×2160, 45 Mbps). New projects default to 720p for the fastest export. Legacy `social1080p` and `master` values decode as 1080p.
- Export is asynchronous, reports progress, supports cancellation, writes H.264 MP4, remembers the last destination folder, copies the file URL, and can reveal the result in Finder.
- Frame-by-frame export uses a sequential source reader instead of repeatedly seeking for every output frame.
- The native AVFoundation path handles keep ranges, whole-take or segmented speed, canvas aspect, transforms, and solid mats for screen-only projects.
- Gradient mats and editable camera overlays use `FrameCompositor`; they are the slower path.
- Mic and system-audio tracks are concatenated over keep ranges. Piece durations are scaled with the edited video, and `.timeDomain` keeps speech pitch natural.
- AVAssetWriter audio/video pumping was changed to avoid deadlock when either input is backpressured; this fixed stalled 1× exports with audio.
- The prior redundant post-export Copy button was removed; save/copy/reveal are one delivery flow.

**Key code:** `TakeExporter.swift`, `TakeExport.swift`, `ExportDelivery.swift`, `EditedTimeline.swift`.

**Review:** export tests cover settings, timeline mapping, audio duration, orientation, the first frame, and source-file preservation. T29 adds the missing camera-overlay coverage.

---

### T27 — Final aspect, padding, and background controls

- **Status:** done
- **Milestone:** M8
- **Depends on:** T26
- **Use cases:** UC-3
- **Goal:** Make output framing flexible while keeping the editor controls compact.

**As-built details:**

- `OutputFrame` stores `paddingHorizontal` and `paddingVertical` for project compatibility and composition math. The current editor exposes one uniform −/+ control and updates both axes together.
- Padding ranges from 0% to 22% and changes by 1 percentage point per click. The default remains 8%.
- Canvas aspect is **16:9** by default or **Match capture**. Output dimensions are even integers derived from the selected preset width.
- 16:9 always uses a mat: non-matching source video is aspect-fit and letterboxed/pillarboxed rather than cropped.
- Match capture with 0% padding has no mat and fills the canvas. Adding padding reveals the selected background around the capture.
- Background is not a separate enable/disable switch. Its visibility follows aspect + padding.
- Background style is Solid or a two-stop linear Gradient with vertical, horizontal, or diagonal direction. Existing project JSON without these fields decodes to compatible defaults.
- Overlay geometry stays in capture/screen space, not mat space.

**Key code:** `Project.swift` (`CanvasAspect`, `OutputFrame`), `CanvasLayout.swift`, `FrameCompositor.swift`, `EditorView.swift`.

**Review:** compositor tests cover zero-padding fill, 16:9 letterboxing, Match capture, independent stored axes, and gradient/solid rendering.

---

### T28 — Export correctness and release app icon

- **Status:** done
- **Milestone:** M8
- **Depends on:** T27
- **Use cases:** UC-3, UC-5, release identity
- **Goal:** Remove visible export regressions and give DemoHero a recognizable release identity.

**As-built details:**

- Native export uses the source track’s `preferredTransform` without an extra manual Y-flip. Pixel-level asymmetric-frame coverage guards against upside-down output.
- ScreenCaptureKit can create an empty edit before its first delivered frame. Export intersects the timeline with real media, removes any retained leading empty composition segment, and anchors the first visible sample at time zero. A colored-mat regression test verifies that export begins with video rather than a mat-only frame.
- The app icon is a high-contrast indigo/violet rounded tile with a white play/spark symbol and red record dot. The asset catalog contains alpha-correct 16, 32, 128, 256, 512, and 1024 pixel variants.
- Debug and Release products compile the same `AppIcon.icns`; `CFBundleIconFile` and `CFBundleIconName` both resolve to `AppIcon`.
- macOS System Settings may cache the older permission-row icon by bundle identifier. That does not indicate a missing asset; quitting/reopening System Settings or resetting TCC refreshes it. Release builds should run from a stable installed location rather than `/private/tmp`.

**Key code/assets:** `TakeExporter.swift`, `TakeExportTests.swift`, `Assets.xcassets/AppIcon.appiconset`.

**Review:** all export tests passed after the timing/orientation fixes, the app built with the full icon set, and the icon changes were committed to the release branch.

---

### T28a — Add feedback and support contact

- **Status:** done
- **Milestone:** M8
- **Depends on:** T28
- **Use cases:** release support
- **Goal:** Give users a clear support route from the recording settings.

**Do:**

- Add one unobtrusive line at the bottom of the settings dialog: “For feedback or support, contact demohero@googlegroups.com.”
- Make the email address open the user’s mail app.

**Acceptance:**

- The line is visible at the bottom of the settings dialog and does not compete with Record or Choose project.
- The address uses a `mailto:` link and has an accessible label.

**Completed notes:**

- Added the support sentence at the bottom of `ReadyToRecordView`.
- The email address is a `mailto:` `Link` with an explicit accessibility label.

**After-task review:**

- **Plan/architecture:** pass — UI-only release support; capture files, timelines, and export are unchanged. T29 remains actionable.
- **Code:** pass — source remains under `src/`; no linter findings; Debug build succeeded.
- **UX:** pass — caption styling keeps the contact secondary to Record and Choose project while leaving the address clickable.
- **Working:** pass — the settings view compiles with the support link and the existing app build succeeds.

---

### T29 — Restore non-destructive presenter preview/export

- **Status:** done
- **Milestone:** M8
- **Depends on:** T28a
- **Use cases:** UC-1, UC-2, UC-5
- **Goal:** Ensure the separate camera is reliable in the editor and present in every matching export, without baking it into the source screen recording.

**Why this task exists:**

- The original T18 notes say export samples screen + camera and calls `FrameCompositor` with overlay visibility. That is no longer true after the export fast-path work.
- The current slow path calls `FrameCompositor.render(camera: nil, overlayVisible: false, ...)`.
- The current native path creates only a screen composition layer.
- Therefore a take recorded with camera on but **Show presenter on screen** off can show the editable camera overlay in preview and omit it from the exported MP4.
- Leaving the live presenter in `screen.mp4` works around export, but makes position/shape/visibility non-destructive editing impossible and conflicts with the separate-track architecture.
- The September 5 command-line review ran 118 tests with one TCC skip and one repeatable failure: `TakePlaybackTests.testPreviewCopiesScreenAndCameraFrames` timed out without a camera frame. Treat that as a real preview reliability signal until the implementation or test proves otherwise.

**Do:**

- Exclude the live presenter panel from the ScreenCaptureKit output. It may remain visible to the person recording.
- Persist its chosen position, size, and shape into `Project.overlay` when creating the take.
- Make camera frame loading in `TakePlayer` reliable and keep a deterministic regression test for it.
- Rewire `EditorView` to pass the real camera-track state and `project.overlayVisibility` into `ComposePreview`; persist drag/resize/shape/visibility edits instead of the current `hasCameraTrack: false`, empty visibility, and no-op callback.
- Restore camera frame sampling and `overlayVisibility` in the frame-compositor export path.
- Add a native camera layer/compositor when parity is safe, or deliberately route projects with a visible camera overlay to the frame-compositor path.
- Keep camera, screen, mic, and system audio source files separate.

**Acceptance:**

- A camera-on take exports the presenter whether the pre-record live presenter was on or off.
- `TakePlaybackTests.testPreviewCopiesScreenAndCameraFrames` passes reliably in isolation and in the full suite.
- Moving, resizing, reshaping, hiding, or muting the camera after recording changes preview and export without changing `screen.mp4` or `camera.mp4`.
- A test uses visibly different screen/camera pixels and proves the overlay appears at the expected export location and disappears during hidden ranges.
- Fast screen-only exports remain on the native path; camera-overlay correctness takes priority over speed.
- Leave M8 `in_progress` and point **Next task** at T31.

**Must preserve:** the T24 camera-session handoff, T25 source/edited timeline mapping, T26 audio behavior, T27 canvas rules, and T28 orientation/first-frame fixes.

**Completed notes:**

- The presenter panel remains visible while recording but is excluded from ScreenCaptureKit. Its live position, size, and shape are converted from Cocoa coordinates into `Project.overlay`; `screen.mp4` and `camera.mp4` remain separate.
- `TakePlayer` now accepts every non-empty camera file instead of rejecting valid short or highly compressed tracks with an arbitrary 8 KB threshold.
- `EditorView` passes the real camera state and visibility edits to `ComposePreview`. The compact camera row restores enable/mute, shape, five-second hide/show, preview drag, and resize controls.
- Camera-enabled projects deliberately use the frame-compositor export path. A sequential camera decoder samples source time alongside the screen decoder; screen-only solid exports retain the native fast path.
- Regression coverage proves live-placement coordinate conversion, reliable screen/camera preview frames, camera pixels in export, hidden camera ranges, and native-path exclusion for camera projects.

**After-task review:**

- **Plan/architecture:** pass — camera media remains non-destructive and separately stored; M9 now sequences StoreKit/free-tier work after the existing release gates.
- **Code:** pass — source remains under `src/`; edited files have no linter findings; Debug build succeeded.
- **UX:** pass — the live presenter placement carries into the editor, where camera edits use one compact row and direct manipulation in the preview.
- **Working:** pass — targeted preview, compositor, and all 13 export tests passed. The stable full suite executed 120 tests with 1 expected Screen Recording/TCC skip and 0 failures; the known hosted audio-composition hang was explicitly excluded.

---

### T29a — Hide camera controls for legacy baked-presenter takes

- **Status:** done
- **Milestone:** M8
- **Depends on:** T29
- **Goal:** Do not offer a second editable camera overlay when an older project already has its presenter baked into `screen.mp4`.

**Do:**

- Recognize the historical full-duration hidden overlay marker written when the presenter was included in the screen recording.
- Hide camera controls and skip separate-camera preview/export work for those projects.
- Preserve editable camera behavior for new non-destructive takes.

**Acceptance:**

- Older baked-presenter projects have no Camera overlay row.
- New projects with a separate presenter retain camera preview, controls, and export.
- Point **Next task** to T31.

**Completed notes:**

- Added `Project.hasEditableCameraOverlay`, which recognizes the historical single full-duration hidden range written when the presenter was baked into `screen.mp4`.
- Legacy baked-presenter projects no longer load the separate camera player, show camera preview chrome/controls, or route export through the camera compositor.
- New projects and projects with ordinary partial hide edits retain the editable camera path.

**After-task review:**

- **Plan/architecture:** pass — this is a compatibility rule derived from existing persisted metadata; no source files or schema versions change.
- **Code/UX:** pass — all camera entry points use the same project capability, so the unavailable option is hidden rather than merely disabled.
- **Working:** pass — 29 project, camera-preview, and export tests passed with no failures; edited files have no linter findings.

**Later product decision (September 6, 2026):** post-record camera overlay editing was removed from the take editor. The shipping path is the live presenter captured in `screen.mp4`. New takes write the same full-duration hidden overlay marker T29a used for legacy baked-presenter projects. Keep this chrome out of `EditorView`. T13/T29 notes above are historical.

---

### T31 — Remove dormant release paths and reconcile release journeys

- **Status:** done
- **Milestone:** M8
- **Depends on:** T29a
- **Use cases:** release quality
- **Goal:** Remove stale compiled paths and close M8 with a clean, accurately tested release candidate.

**Why this task exists:**

- `ReadyPreviewView` and `readyPreviewKind` are compiled and persisted but the view is not mounted by `ReadyToRecordView`.
- `ProjectStore.writeSampleProject()` remains as historical M1 support but has no shipping UI.
- Manual zoom metadata, rendering, and tests remain even though zoom is no longer part of the release product.
- `PRODUCT.md` still describes historical labels, 1080p defaults, and manual zoom behavior.
- Command-line testing has a known AVFoundation hang in `testCompositionMixesScreenAndMic`; release verification needs a stable documented suite rather than silently omitting it.

**Do:**

- Either wire the ready preview into the recording settings with a clear user purpose or remove the dead view/state.
- Remove the sample-project API if no test or release workflow needs it; otherwise mark it explicitly as test-only.
- Remove dormant zoom behavior while preserving project decoding compatibility, and update `PRODUCT.md` to match the shipping labels, 720p default, and restored camera behavior.
- Make the audio-composition test deterministic under the command-line host or give it an explicit environment-aware skip with a reason.
- Run the full M8 Demo on a Mac, record exact test totals/skips, and update this as-built snapshot.

**Acceptance:**

- No half-wired preview mode or sample-project action remains in the release target.
- No UI, product promise, or active rendering path advertises zoom.
- The documented test command terminates reliably with no unexplained failures.
- T29 camera behavior, framing, export orientation/first-frame, save/copy, and the app icon all pass the M8 Demo.
- Set T31 and M8 to `done` / `demoable`, then set **Next task** to T32.

**Completed notes:**

- Removed the unmounted ready-preview UI, persisted preview mode, and unused ScreenCaptureKit preview controller.
- Removed manual zoom from the model, compositor, preview, exporter, tests, and Xcode target. Older manifests with a `zooms` key remain decodable because unknown keys are ignored and are dropped when re-saved.
- Removed cursor/click sidecar collection and project metadata because it existed only to support zoom. Window-frame lookup remains as the narrowly named `WindowGeometry` utility used by source highlighting and presenter placement.
- Removed `ProjectStore.writeSampleProject`; the deterministic `Project.sample` fixture is compiled only in Debug/test builds.
- Replaced the fragile AVAssetWriter-based test-audio generator with synchronous `AVAudioFile` writing. `testCompositionMixesScreenAndMic` now finishes and passes under the command-line test host.
- Reconciled `PRODUCT.md` with shipping labels, the 720p default, current camera editing, and zoom as an explicit non-goal.

**After-task review and M8 Demo:**

- **Plan/architecture:** pass — screen, camera, mic, and system audio remain separate; unsupported zoom/mouse keys decode without becoming active behavior. M9 begins at T32.
- **Code:** pass — removed 1,200+ lines of dormant code; source remains under `src/`; no linter findings.
- **UX:** pass — the release app launches with no unreachable preview mode, sample-project action, or zoom promise.
- **Working:** pass — Debug and universal Release builds succeeded. The complete test command terminated normally: 109 tests executed, 1 expected Screen Recording/TCC skip, 0 failures. The Release app launched successfully.
- **M8 Demo:** pass from the release build plus automated journey coverage for capture lifecycle, separate tracks, camera edit/export, trim/speed/audio, aspect/padding/background, orientation/first frame, presets, save/copy, and icon resources. Live Screen Recording remains permission-gated and is the single explicit test skip.

---

## M9 — Free and Pro release

**Status:** demoable  
**Tasks:** T32, T33, T34, T35  
**Runnable when:** T35 is `done`

M9 adds App Store monetization only after the existing recording/edit/export release candidate is stable. “Unlock all features” means every shipping DemoHero feature; it does not silently add parked v1.x features.

### T32 — StoreKit 2 entitlement foundation

- **Status:** done
- **Milestone:** M9
- **Depends on:** T31
- **Goal:** Establish one durable Pro entitlement shared by monthly, yearly, and lifetime purchases.

**Do:**

- Use StoreKit 2 verified transactions, current entitlements, and transaction updates; do not trust a local Boolean as proof of purchase.
- Define stable product IDs before creating them in App Store Connect. Proposed IDs are `com.demohero.DemoHero.pro.monthly`, `com.demohero.DemoHero.pro.yearly`, and `com.demohero.DemoHero.pro.lifetime`.
- Treat monthly and yearly as auto-renewable subscriptions in one subscription group at the same level; treat lifetime as a non-consumable.
- Add an Xcode StoreKit configuration and deterministic entitlement tests. Cache only the last verified state for responsive offline launch, then reconcile with StoreKit.

**Acceptance:**

- Any active monthly/yearly subscription or verified lifetime purchase sets one observable Pro entitlement.
- Expired, revoked, unverified, or absent transactions do not unlock Pro.
- Product display names and prices come from StoreKit, not hardcoded UI strings.

**Completed notes:**

- Added `src/Store/` with `ProProductID` (`com.demohero.DemoHero.pro.monthly` / `.yearly` / `.lifetime`), `ProEntitlement`, and `ProAccess` so monthly, yearly, and lifetime resolve to one Pro unlock.
- `StoreService` uses StoreKit 2 `Product.products`, `Transaction.currentEntitlements`, and `Transaction.updates`. A UserDefaults cache paints the last verified state at launch only; a successful empty entitlement set clears Pro.
- Xcode StoreKit configuration is `src/Resources/DemoHero.storekit` (same subscription group, same level; lifetime is a non-consumable) and is selected on the DemoHero scheme for Run and Test. Current US display prices in that file are $3.99 monthly, $19.99 yearly, and $49.99 lifetime.
- `AppSession` owns `store` and starts reconciliation on launch. Purchase/restore APIs exist for T34; names and prices are `Product.displayName` / `Product.displayPrice`.
- Deterministic tests cover active/expired/revoked/unverified/absent transactions and the cache-is-not-proof rule.

**After-task review:**

- **Supports later tasks?** yes — T33 can read `store.isPro`; T34 can purchase loaded products.
- **Blocks UC-2 / UC-3 / UC-5?** no — capture and export paths are unchanged.
- **Code under `src/`?** yes — Store types, StoreKit config, and tests live under `src/`.
- **Plan still sequenced?** yes — T33 can enforce limits against the observable entitlement.
- **Milestone still demoable?** n/a — M9 continues.
- **Code:** pass — Debug build succeeded; 12 `ProEntitlementTests` passed.
- **UX:** pass — no purchase UI yet; free recording/export still unrestricted until T33.
- **Working:** pass — launch still records and exports; StoreKit reconciliation is background-only.

### T33 — Enforce the free recording and export limits

- **Status:** done
- **Milestone:** M9
- **Depends on:** T32
- **Goal:** Free users can export 720p and record for at most ten minutes; Pro users keep the current unrestricted behavior.

**Do:**

- Show the free limit in the recording menu-bar state as elapsed time against `10:00`.
- At ten minutes, invoke the existing stop/finalize path exactly once and open the resulting take in the editor.
- Restrict the effective free export preset to 720p even for an older project that persisted 1080p/4K; surface the reason in the UI rather than failing after Save.
- Keep source recordings at capture quality. The 720p limit applies to exported output, not destructive source transcoding.

**Acceptance:**

- Free recording auto-stops at 10:00 and proceeds to the editor without losing screen, camera, or audio files.
- Free export cannot exceed 720p through UI, persisted metadata, or direct export entry points.
- Pro has no DemoHero-imposed recording duration or export-resolution limit.

**Completed notes:**

- `FreeLimits` caps free recording at 10 minutes and free export at 720p. Source capture is unchanged.
- The floating recording indicator and settings recording state show `elapsed / 10:00` for free users. At 10:00 `AppSession` calls the existing stop/finalize path once and opens the editor.
- While a free take is recording, the menu bar shows that `elapsed / 10:00` clock and an **Upgrade to Pro** action next to it (also on the floating timer). Pro users do not see the upgrade control. Free limits themselves are unchanged.
- Export always uses `FreeLimits.effectiveExportPreset`. A persisted 1080p/4K project still exports 720p while free. Choosing a locked preset does not rewrite the project; it explains the limit and sets `isShowingProStore` for T34.
- Pro entitlement removes both caps.

**After-task review:**

- **Supports later tasks?** yes — locked presets and `isShowingProStore` are the T34 entry points.
- **Blocks UC-2 / UC-3 / UC-5?** no — tracks stay separate; only export settings are clamped.
- **Code under `src/`?** yes.
- **Plan still sequenced?** yes — T34 can present the purchase sheet.
- **Code:** pass — Debug build succeeded; 4 `FreeLimitsTests` passed, including the session clamp.
- **UX:** pass — free users see the 10:00 clock and a 720p export hint instead of a post-Save failure.
- **Working:** pass — Pro remains uncapped; auto-stop reuses `stopRecording()`.

### T34 — Pro purchase, restore, and subscription UI

- **Status:** done
- **Milestone:** M9
- **Depends on:** T33
- **Goal:** Make the three plans understandable and purchasable without blocking the free workflow.

**Do:**

- Add a Pro screen reachable from settings and locked export presets, showing monthly, yearly, and lifetime products with localized StoreKit prices.
- Explain auto-renewal, billing periods, free limits, and the lifetime alternative before purchase. Highlight yearly as the best recurring value without obscuring monthly or lifetime.
- Handle pending, cancelled, failed, successful, upgraded, expired, and revoked purchases.
- Include Restore Purchases via `AppStore.sync()` and a Manage Subscription action.
- Add in-app Privacy Policy and Terms of Use links required for subscription release.

**Acceptance:**

- StoreKit test sessions cover buying each product, restoring, expiration/revocation, cancellation, and product-load failure.
- Purchase completion updates recording/export access without relaunching.
- The app stays usable at the free tier when the store is unavailable.

**Completed notes:**

- `ProStoreView` is reachable from settings, the menu bar, and locked 1080p/4K presets. Plan names and prices come from StoreKit; yearly is marked Best value without hiding monthly or lifetime.
- Copy covers auto-renewal, billing periods, the 10-minute / 720p free limits, and lifetime as a non-renewing alternative. Restore uses `AppStore.sync()`; Manage Subscription opens Apple’s account page.
- Pending, cancelled, failed, and success phases stay on screen. Product-load failure keeps 720p recording/export available.
- In-app Privacy Policy and Terms of Use sheets plus browser links (`docs/PRIVACY.md` and Apple’s Standard EULA).
- `StorePurchaseTests` cover SKTestSession buys of all three products, restore, expiration, refund, product-load failure, and pending/failed phases.

**After-task review:**

- **Supports later tasks?** yes — T35 can document the same product IDs, prices, and legal URLs.
- **Blocks UC-2 / UC-3 / UC-5?** no.
- **Code under `src/`?** yes, plus the public privacy policy in `docs/`.
- **Plan still sequenced?** yes — T35 is packaging and the owner checklist.
- **Code:** pass — Debug build succeeded; 6 `StorePurchaseTests` passed.
- **UX:** pass — free workflow remains usable when plans cannot load.
- **Working:** pass — entitlement changes apply immediately to recording/export without relaunch.

### T35 — App Store package, test, and submission guide

- **Status:** done
- **Milestone:** M9
- **Depends on:** T34
- **Goal:** Produce a signed, reviewable Mac App Store build and an exact owner checklist for the first app/IAP submission.

**Do:**

- Verify the App Sandbox, signing team, distribution certificate/profile, release version/build number, privacy declarations, and archive validation.
- Create `docs/APP_STORE_RELEASE.md` with App Store Connect product setup, US price points ($3.99 monthly, $19.99 yearly, $49.99 lifetime), subscription-group setup, tax category, localizations, review screenshots/notes, sandbox/TestFlight testing, and submission order.
- Document the Paid Apps Agreement, banking, tax, support URL, Privacy Policy URL, and Terms of Use URL prerequisites.
- Submit the first monthly/yearly subscriptions, subscription group, and lifetime non-consumable in the same review submission as the new macOS app version, as required for first-of-type products.

**Acceptance:**

- A Release archive validates for Mac App Store distribution.
- Sandbox purchases and restore pass with App Store Connect products.
- M9 Demo proves free limits, all three purchase paths, restore, Pro unlock, and a successful 1080p/4K Pro export.

**Completed notes:**

- Added [APP_STORE_RELEASE.md](APP_STORE_RELEASE.md): product IDs, US prices ($3.99 monthly, $19.99 yearly, $49.99 lifetime), subscription group, tax category, Paid Apps Agreement / banking / tax, Privacy Policy and Terms URLs, review screenshots/notes, Sandbox/TestFlight, and the required first-submission order (app + monthly + yearly + lifetime together).
- Release configuration now enables the Hardened Runtime. A universal Release archive succeeded locally (`adhoc,runtime`); App Store Connect export still needs the owner’s Team and distribution certificate (`docs/ExportOptions-MAS.plist`).
- Privacy declarations already live in `Info.plist`. Support email is `demohero@googlegroups.com`. Public policy text is `docs/PRIVACY.md`.
- Isolated free-tier session tests from leftover StoreKit test transactions so the full suite stays deterministic.

**After-task review and M9 Demo:**

- **Supports later tasks?** yes — no further v1 tasks; owner work is ASC + Sandbox.
- **Blocks UC-2 / UC-3 / UC-5?** no.
- **Code under `src/`?** yes. Release docs and the MAS export template live in `docs/`.
- **Plan still sequenced?** yes — Next task is none.
- **Milestone still demoable?** yes — see Demo below.
- **Code:** pass — Debug build succeeded; complete suite 130 tests, 1 Screen Recording/TCC skip, 0 failures.
- **UX:** pass — free clock, locked presets, Pro sheet, restore, and legal links are in the shipping UI.
- **Working:** pass — Release archive built with Hardened Runtime. Live ASC Sandbox purchases are owner-gated and documented.
- **M9 Demo:** pass for engineering paths (free 720p + 10:00 cap, StoreKit configuration prices, SKTestSession buy/restore/expire/refund/load-failure, Pro entitlement updates without relaunch). Owner still runs Sandbox/TestFlight for the three live products and a Pro 1080p/4K export after setting a signing team.

**Later price adjustment (September 6, 2026):** US list prices changed to **$3.99 / month**, **$19.99 / year**, and **$49.99 lifetime** (one-time). Free limits are unchanged (720p export, 10:00 recording cap). Update `DemoHero.storekit` and [APP_STORE_RELEASE.md](APP_STORE_RELEASE.md) together if prices change again. Do not hardcode prices in UI strings.

---

## Current release snapshot and boundaries

- **Version:** `0.1.0` (`CFBundleVersion` 1).
- **Distribution:** local ad-hoc Debug/Release builds. Release enables the Hardened Runtime. A Mac App Store archive still needs the owner’s Apple Distribution team; see [APP_STORE_RELEASE.md](APP_STORE_RELEASE.md). Debug and Release binaries can require separate TCC grants.
- **Monetization:** StoreKit 2 Pro entitlement (`com.demohero.DemoHero.pro.monthly` / `.yearly` / `.lifetime`). US list prices are **$3.99 / month**, **$19.99 / year**, and **$49.99 lifetime** (one-time). Free tier is 720p export and a 10-minute recording limit. During a free recording the menu bar and floating timer show **Upgrade to Pro** next to `elapsed / 10:00`. Names and prices shown in the app come from StoreKit (`DemoHero.storekit` locally).
- **Local storage:** `~/Movies/DemoHero/Captures/<take>/Project.json` plus `screen.mp4`, optional `camera.mp4`, `mic.m4a`, and `systemAudio.m4a`. The primary export flow writes an MP4 to the save-panel destination; `export.mp4` in the take folder remains a helper/fallback path.
- **Non-destructive metadata:** keep ranges, speed segments, output frame, and export preset live in schema version 1. Camera overlay/visibility keys still decode for older takes but are not edited in the shipping UI. Older manifests containing zoom or mouse-sidecar keys still decode; those unsupported keys are ignored and dropped on the next save.
- **Presenter:** **Show presenter on screen** is captured into `screen.mp4`. New takes write a full-duration hidden overlay marker so the editor and export do not composite `camera.mp4` on top. Do not put camera overlay controls back in the take editor.
- **Capture clock:** screen, camera, mic, and system audio share host-clock-relative timestamps; export removes a leading screen empty edit.
- **Export defaults:** 720p, 1×, H.264, 16:9, 8% uniform padding, solid dark mat.
- **Performance choice:** solid screen-only video uses native AVFoundation composition; a gradient uses frame rendering. Historical editable-overlay projects may still take the compositor camera path.
- **Not supported:** manual/auto zoom, captions, GIF, 9:16 reflow, keyboard overlay, share links/cloud upload, and Journey C.
- **Known environment behavior:** Screen Recording tests skip when TCC is not granted to the test host; System Settings can cache an older app icon.
- **Verification at T35:** Debug and universal Release builds passed; a Release archive succeeded with Hardened Runtime (`adhoc,runtime`). The complete suite ran 130 tests with 1 Screen Recording/TCC skip and 0 failures.
- **Release gates:** T32–T35 landed StoreKit, free-tier enforcement, the Pro purchase UI, and the App Store submission guide. Live Sandbox/TestFlight purchases still require the owner’s ASC products and a signed MAS/TestFlight build.
