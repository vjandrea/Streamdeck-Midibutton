# Agent Notes

## Repo Reality

This is the source repo for the Stream Deck MIDI plugin, not just a deployed bundle.

- Runtime source lives in `Sources/`.
- The installable plugin bundle lives in `co.uk.clarionmusic.midibutton.sdPlugin/`.
- The current build system is macOS-only: `Sources/macOS/midibutton.xcodeproj`.
- The checked-in bundle is also macOS-only today: `co.uk.clarionmusic.midibutton.sdPlugin/manifest.json` has `CodePathMac` and only a macOS `OS` entry.

## Current Goal

Primary task: add a Windows port, starting with an output-first MVP.

Chosen product decisions:

- Windows scope is output-first MVP, not full parity.
- Global MIDI input syncing and full virtual-port behavior are deferred unless they fall out cheaply during the port.
- Keep the existing `uk.co.clarionmusic.midibutton.*` action IDs during the port.
- Rename the plugin namespace to `net.vjandrea.midibutton` only after the Windows build is working.

## Source Layout That Matters

- `Sources/StreamDeckMidiButton.cpp` and `Sources/StreamDeckMidiButton.h`
  Core plugin behavior, action handling, MIDI send/receive, fade logic, MMC icon switching.
- `Sources/Common/main.cpp`
  Stream Deck SDK entrypoint and connection-manager bootstrapping.
- `Sources/Common/ESDConnectionManager.*`
  WebSocket plumbing to Stream Deck.
- `Sources/Common/ESDUtilitiesMac.cpp` and `Sources/Common/ESDUtilitiesWindows.cpp`
  Platform-specific filesystem/plugin-path helpers.
- `co.uk.clarionmusic.midibutton.sdPlugin/propertyinspector/`
  Property inspector HTML/CSS/JS shipped with the plugin bundle.

## Confirmed Windows-Port Blockers

- There is no Windows build system checked in. Only the Xcode project exists.
- `Sources/StreamDeckMidiButton.h` includes `CoreServices/CoreServices.h` unconditionally. That must be guarded or removed for Windows compilation.
- `Sources/macOS/timer.h` is included as `"timer.h"` from the shared runtime. The Xcode project provides that include path implicitly. A Windows build must add the equivalent include path or relocate the header.
- The property inspector loader in `co.uk.clarionmusic.midibutton.sdPlugin/propertyinspector/index.html` references `pi_win.js`, but that file is not in the repo.
- The checked-in manifest does not declare `CodePathWin` or a Windows OS target.
- The runtime references MMC active icons such as `Icons/stop_active.png`, but only inactive MMC icons are currently present in `co.uk.clarionmusic.midibutton.sdPlugin/Icons/`.

## Existing Cross-Platform Assets

- `Sources/Common/ESDUtilitiesWindows.cpp` already exists.
- The project already vendors cross-platform networking and JSON dependencies.
- The runtime uses `rtmidi17`, and headers are already present under `Sources/include/rtmidi17/`.
- The runtime already gates some virtual-port behavior behind `#if defined(__APPLE__)`, which is the right direction for a Windows output-first split.

## Implementation Plan For The Windows Machine

1. Establish a Windows build target before changing behavior.
   Create a build system that works on Windows first.
   Preferred route: add `CMakeLists.txt` for the runtime and support Visual Studio/MSVC.
   Acceptable route: add a Visual Studio solution/project directly if that is faster.

2. Make the runtime compile cross-platform.
   Guard Apple-only includes and code paths.
   Ensure shared sources can include `timer.h` from a stable cross-platform include path.
   Keep `ESDUtilitiesMac.cpp` and `ESDUtilitiesWindows.cpp` selected per platform.

3. Ship a Windows runtime with output-only behavior.
   Prioritize outbound MIDI for Note, Note Toggle, CC, CC Toggle, Program Change, and MMC.
   Keep existing action UUIDs and settings payload shape unchanged unless compilation forces a minimal compatibility shim.
   Defer MIDI input-driven state sync if it complicates the first Windows build.

4. Make the property inspector honest on Windows.
   Add `pi_win.js`.
   Hide or omit unsupported Windows-only deferred controls instead of showing broken options.
   Specifically review any UI for `useVirtualPort`, MIDI input port selection, and other settings tied to macOS-only behavior.

5. Wire the plugin bundle for Windows.
   Add `CodePathWin` and Windows `OS` support in the manifest.
   Keep the macOS bundle working while adding the Windows executable beside it.

6. Fix asset/runtime mismatches discovered during inspection.
   Either add the missing MMC active icons or change runtime/icon behavior so it does not reference nonexistent files.

7. After the Windows build works, do the namespace rename.
   Move from `co.uk.clarionmusic.midibutton.sdPlugin` and `uk.co.clarionmusic.midibutton.*` to the `net.vjandrea.midibutton` namespace in one clean pass.

## Behavioral Notes To Preserve

- Do not casually rename settings keys used by the property inspector and runtime.
- `StreamDeckMidiButton` stores button settings from Stream Deck payloads and uses them directly in runtime behavior.
- CC fade behavior is runtime-side, not inspector-side.
- MMC actions change button images dynamically at runtime.
- Virtual-port behavior is currently treated as macOS-specific in the runtime and should stay out of the Windows MVP unless explicitly implemented.

## Recommended Tooling On Windows

- Visual Studio 2022 with Desktop development for C++
- CMake
- Git
- Stream Deck for Windows installed locally for manual plugin testing

Optional but useful:

- `ninja`
- a second MIDI utility or DAW to verify outbound MIDI messages

## Verification Checklist

- Runtime builds successfully on Windows.
- Plugin bundle installs in Stream Deck on Windows.
- Each action can be added and opened in the property inspector without JS errors.
- Outbound MIDI works for Note, Note Toggle, CC, CC Toggle, Program Change, and MMC.
- Unsupported controls are hidden or clearly unavailable on Windows.
- macOS manifest/runtime wiring remains intact after Windows support is added.

## Working Guidance

- Start by making the build reproducible on Windows. Do not begin with a broad refactor.
- Prefer compatibility shims and guarded platform branches over deep rewrites for the MVP.
- Keep the first Windows milestone small: compile, load, send outbound MIDI, ship.
- If a requested change touches MIDI input synchronization or virtual ports, treat it as phase two unless it is nearly free.
