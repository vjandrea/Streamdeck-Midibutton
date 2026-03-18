# Agent Notes

## Repo Reality

This is the source repo for the Stream Deck MIDI plugin, not just a deployed bundle.

- Runtime source lives in `Sources/`.
- The installable plugin bundle lives in `co.uk.clarionmusic.midibutton.sdPlugin/`.
- A Windows-oriented `CMakeLists.txt` now exists at repo root for the runtime.
- The macOS Xcode project still exists at `Sources/macOS/midibutton.xcodeproj`.
- The plugin manifest now declares both `CodePathMac` and `CodePathWin`.

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

- The Windows runtime has not been built successfully yet on this machine.
- The current shell does not inherit MSVC environment variables automatically; `cl` works only after launching `VsDevCmd.bat`.
- No Visual Studio instance/toolchain was available to the agent's plain shell during CMake configure, even though Build Tools are installed on disk.
- End-to-end Windows verification in Stream Deck has not happened yet.

## Current Windows Foundation Status

Implemented so far:

- Root `CMakeLists.txt` added for a Windows build of `midibutton.exe`.
- Shared headers were made less dependent on the macOS prefix header:
  Apple-only `CoreServices` include is now guarded.
  JSON and `DebugPrint` fallbacks are available from shared headers.
- `Sources/Common/ESDUtilitiesWindows.cpp` now includes `windows.h`.
- `co.uk.clarionmusic.midibutton.sdPlugin/propertyinspector/js/pi_win.js` now exists.
- The Windows property inspector is intentionally output-only:
  it hides virtual-port settings and MIDI input-port selection.
- `co.uk.clarionmusic.midibutton.sdPlugin/manifest.json` now includes `CodePathWin` and a Windows `OS` target.
- Missing MMC `_active` icons were added by duplicating the inactive assets so runtime icon swaps do not fail.
- The shared runtime now treats Windows as output-first:
  Windows ignores stale virtual-port and MIDI-input global settings.
  Windows only initializes MIDI output in `DidReceiveGlobalSettings()`.
  Windows no longer sends MIDI-input port payloads back to the property inspector.

## Existing Cross-Platform Assets

- `Sources/Common/ESDUtilitiesWindows.cpp` already exists.
- The project already vendors cross-platform networking and JSON dependencies.
- The runtime uses `rtmidi17`, and headers are already present under `Sources/include/rtmidi17/`.
- The runtime already gates some virtual-port behavior behind `#if defined(__APPLE__)`, which is the right direction for a Windows output-first split.

## Implementation Plan For The Windows Machine

1. Establish a Windows build target before changing behavior.
   Status: mostly done.
   `CMakeLists.txt` exists and targets a Windows runtime build.
   Remaining work is successful configure/build in an MSVC-enabled shell.

2. Make the runtime compile cross-platform.
   Status: partially done.
   Apple-only includes and some Apple-only behavior are now guarded.
   The timer header still comes from `Sources/macOS/timer.h`, but CMake adds that include path explicitly.
   Shared code still needs an actual Windows compile pass to catch remaining compiler issues.

3. Ship a Windows runtime with output-only behavior.
   Status: in progress.
   The property inspector and global-settings flow now align with an output-only Windows MVP.
   Existing action UUIDs remain unchanged.
   MIDI input-driven sync is still deferred.

4. Make the property inspector honest on Windows.
   Status: done for MVP.
   `pi_win.js` exists.
   Virtual-port and MIDI-input controls are hidden/omitted on Windows.

5. Wire the plugin bundle for Windows.
   Status: mostly done.
   `CodePathWin` and Windows `OS` support are present in the manifest.
   The remaining step is to place a built `midibutton.exe` beside the bundle contents.

6. Fix asset/runtime mismatches discovered during inspection.
   Status: done for current runtime behavior.
   Missing MMC active icons were added as placeholder duplicates of inactive icons.

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

- Runtime configures and builds successfully on Windows in an MSVC-enabled shell.
- `midibutton.exe` is copied into `co.uk.clarionmusic.midibutton.sdPlugin/`.
- Plugin bundle installs in Stream Deck on Windows.
- Each action can be added and opened in the property inspector without JS errors.
- Outbound MIDI works for Note, Note Toggle, CC, CC Toggle, Program Change, and MMC.
- Unsupported controls are hidden or clearly unavailable on Windows.
- macOS manifest/runtime wiring remains intact after Windows support is added.

## Immediate Next Step

- Run CMake and the build from a shell bootstrapped with:
  `C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\Common7\Tools\VsDevCmd.bat`
- Once a compile succeeds, fix the remaining compiler errors iteratively instead of broad refactors.

## Working Guidance

- Start by making the build reproducible on Windows. Do not begin with a broad refactor.
- Prefer compatibility shims and guarded platform branches over deep rewrites for the MVP.
- Keep the first Windows milestone small: compile, load, send outbound MIDI, ship.
- If a requested change touches MIDI input synchronization or virtual ports, treat it as phase two unless it is nearly free.
