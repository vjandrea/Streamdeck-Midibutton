
`MIDIbutton` is a plugin to send MIDI messages from the Elgato Streamdeck.


# Description

`MIDIbutton` is a plugin to send MIDI messages (Note On/Off, CC and MMC) from the Elgato Streamdeck. The messages can be customised, and the MIDI port can be renamed. Uses the RtMidi library for MIDI communication.


# Features

- code written in C++
- macOS runtime supported today
- Windows output-first port in progress


![](screenshot.png)


# Installation

In the Release folder, you can find the file `co.uk.clarionmusic.midibutton.streamDeckPlugin`. If you double-click this file on your machine, Stream Deck will install the plugin.


# Windows build status

The repo now contains an initial Windows build path:

- `CMakeLists.txt` builds the plugin runtime as `midibutton.exe`
- `co.uk.clarionmusic.midibutton.sdPlugin/manifest.json` declares `CodePathWin`
- the Windows property inspector hides macOS-only controls such as virtual ports and MIDI input selection

Current Windows scope is an output-first MVP. MIDI input sync and virtual-port behavior remain macOS-only for now.


# Building on Windows

Prerequisites:

- Visual Studio 2022 with Desktop development for C++
- CMake

Example configure and build commands:

```powershell
cmake -S . -B build -G "Visual Studio 17 2022" -A x64
cmake --build build --config Release
```

After building, place the produced `midibutton.exe` inside `co.uk.clarionmusic.midibutton.sdPlugin/` before installing or testing the plugin in Stream Deck on Windows.


# Source code

The Sources folder contains the source code of the plugin.
