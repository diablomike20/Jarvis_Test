# J.A.R.V.I.S.

A voice and hand-gesture assistant behind a heads-up interface built to look
like the one in the Iron Man films. It wakes to its name and answers aloud,
tracks your hand through the webcam, and puts 3D CAD models in the interface
for you to turn with your hand and change by voice. Tested on macOS (Windows
and Linux installers included, not yet tested). Control of the Mac's keyboard,
mouse and windows is written but waits on macOS's Accessibility permission,
not yet granted on the test machine.

A live screen recording of the hand-driven model viewer:
[parahuman.net/#jarvis](https://parahuman.net/#jarvis). Full reference:
[docs/MANUAL.md](docs/MANUAL.md).

## What it does

**Voice.** Say “Jarvis” and it wakes; “Jarvis, open Safari” does both in one
breath. It answers aloud in a British voice and stops listening while it
speaks, so it does not transcribe its replies (with one tab open).

**Hands.** The webcam tracks your hand, body and face (14–18 fps measured live
on a MacBook). Only the hand it is already following can drive; a second
person's hand has to wait. Live, the hands have driven the 3D viewer. Desktop
gestures are written but, on a Mac, wait on the Accessibility permission:
point to move the cursor, pinch to click, pinch and move to drag, swipe to
switch apps.

**3D models in the air.** Say “show” and a part's name, and the part appears
in the interface as a hologram. An open hand turns it; pushing toward or away
from the camera zooms. It opens STL meshes, OpenSCAD files (CAD written as
code) and STEP files (the standard format CAD programs exchange), with no CAD
program open.

**CAD edits by voice.** On an OpenSCAD part, “set rim thickness to 3” or “make
the lenses bigger” rebuilds it in place: 0.16 s for a glasses frame. With
Claude Code set up, changes a number cannot express go to Claude, which edits
the OpenSCAD source or scripts a STEP assembly's named parts: on a 306-part
engine assembly, “make the nozzle 20 percent longer” took its length from
3,420 to 3,855 mm in 50 s, throat fixed. Each STEP edit is compared for size
and photographed from three angles for the model to judge; one that looks
wrong is undone and redone. The original file is untouched until “save”.

**Control.** 43 actions: apps, windows, browser tabs, keyboard, mouse, media,
volume, brightness, screenshots, clipboard, Shortcuts and system status. The
voice parser, the gestures, the on-screen command index and the language model
all call the same registry (on a Mac, its keyboard, mouse and window actions
wait on the Accessibility permission).

**Thought.** Common commands are matched on the machine with no model
involved. Anything else goes to a language model (Qwen3 through Ollama, free
and local, or Claude), which can call the same actions as tools. When a
spoken name fits more than one dimension, it asks which one you meant.

## Progress

**22 September 2026, first commit → 24 September:** hand-driven 3D viewer,
CAD edits by voice and through Claude, and installers for three systems (two
not yet run).

Measured during the build:

| | before | after |
|---|---|---|
| Local model reply, “set the volume” (Qwen3 8B) | 8.4 s; “take a screenshot” timed out at 60 s | 1.4 s, same tool chosen (its reasoning step switched off) |
| A question through Claude Code | 11.7 s | ~3 s (2.6–8 s measured), once run apart from the user's own Claude settings |
| OpenSCAD rebuild, glasses frame | 5.9 s on the older CGAL geometry engine | 0.16 s on the newer Manifold engine |
| OpenSCAD rebuild, a large machine assembly | unfinished after 10 minutes (CGAL) | 8 s (Manifold) |
| CAD-mode drift from a hand held still (synthetic hands) | 35 px per frame | 2.5 px per frame, with a one-euro filter (smoothing that eases off as the hand speeds up) |
| Shutdown with the interface open | over 20 s | 0.1 s |
| Spoken commands dropped by the wake window, in 7 replayed from recordings | 2 of 7 | both cases fixed and covered by a regression test; the 7 recordings were not re-run after the fix |

- A common command (“open Safari”, “what's my battery”) is matched in about
  0.01 ms on an Apple M5 Pro. A model round trip takes 1–4 s locally, or ~3 s
  with Claude. Shape edits through Claude take 20–90 s.
- Test run on macOS, 18 October 2026: the seven suites that need no camera,
  microphone, AI model or running server pass 152 checks with no failures.
  The synthetic-hand gesture tests caught two real defects on the first day: a
  pinch threshold that could never fire, and a fist that read as a pinch.
- Actions 41 → 43 and test files 0 → 13 from the first commit to 24 September
  (development history; this repository starts from a 23 September snapshot).

## Install

**macOS or Linux** — paste into a terminal:

```sh
curl -fsSL https://raw.githubusercontent.com/dylanleitenberg-tech/JARVIS/main/install.sh | sh
```

**Windows** — paste into PowerShell:

```powershell
powershell -ExecutionPolicy Bypass -c "irm https://raw.githubusercontent.com/dylanleitenberg-tech/JARVIS/main/install.ps1 | iex"
```

No administrator rights and no Python needed: it installs into your own
folders with a private Python, adds JARVIS to your Applications / Start menu /
app menu, and opens it. Run it again to update. About 750 MB.

**On first open, JARVIS asks for everything it needs** — camera, microphone,
and on a Mac Accessibility and app control — and its SETUP panel shows the
state of each with a button that asks for it. Press SETUP (or `p`) any time.

| | macOS | Windows | Linux |
|---|---|---|---|
| Voice | Chrome or Edge | Chrome or Edge (Edge is built in) | Chrome or Edge |
| Hand tracking | yes | written, untested | written, untested |
| Keyboard, mouse, windows | written; needs Accessibility | written, untested | X11 written, untested; Wayland does not allow it |
| Local AI (free) | [Ollama](https://ollama.com/download), then DOWNLOAD in SETUP | same | same |

Speech is transcribed in the browser, by Google (Chrome) or Microsoft (Edge);
the camera never leaves your machine. For answers beyond the fixed commands,
install Ollama for a free local model, or set `ANTHROPIC_API_KEY` for Claude. `.scad` files need
[OpenSCAD](https://openscad.org); STEP files need the installer's
`--with-step` option (CadQuery, about 1 GB).

Uninstall: `sh ~/Library/Application\ Support/JARVIS/install.sh --uninstall`
(Mac), `sh ~/.local/share/jarvis/install.sh --uninstall` (Linux), or on
Windows set `JARVIS_UNINSTALL=1` and run the install line again. Your model
files are never touched.

Checkout, permissions, AI backends, every gesture and command, troubleshooting:
[docs/MANUAL.md](docs/MANUAL.md).

## Status

- **Run live on macOS:** camera hand tracking, the hand-driven 3D viewer,
  voice in and out, Claude, and the local Qwen3 fallback.
- **Mac keyboard, mouse and window control** (gesture cursor, clicks, app
  switching, CAD mode in other applications): written, waiting on macOS's
  Accessibility permission, which has not been granted on the test Mac. See
  [If control never turns on](docs/MANUAL.md#if-control-never-turns-on).
- **Windows and Linux:** control backends and one-line installers are written;
  neither has been run on a real machine.
- Written with AI coding agents under Dylan Leitenberg's direction.

## License

MIT: see `LICENSE`. Bundled fonts and three.js keep their own licenses: see
`THIRD_PARTY_NOTICES.md`.
