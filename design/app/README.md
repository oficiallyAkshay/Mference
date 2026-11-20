# Mac app design

Design sketches and a build spec for the Mference Mac app: a menu-bar app, a
first-launch window, a terminal status view, automatic updates and the
changes to the landing site. Nothing here is built yet. The chat itself stays
[Open WebUI](../../docs/OPEN_WEBUI.md) in the browser.

Open the sketches in a browser; they need no build step:

| Sketch | Status |
| --- | --- |
| [menu-bar.html](menu-bar.html) | Approved: the main menu and 24 states |
| [onboarding.html](onboarding.html) | In review: the first-launch window (it plays) and the site changes |

## Surfaces

| Surface | For | Built with |
| --- | --- | --- |
| Menu bar | Everyone: start, stop, switch and install models, open the chat | SwiftUI `MenuBarExtra` in menu style, so macOS draws the Liquid Glass material, dark mode and Reduce Transparency |
| First-launch window | New users | A regular SwiftUI window with a step sidebar |
| `mference status` | Technical users, in the terminal | [Noora](https://github.com/tuist/Noora) live tables and progress bars |
| Landing site | Anyone deciding to download | The existing page on the `add-landing-site` branch, extended |

All four read one status feed from `MferenceServer`, so a fact has one source.

## Menu bar

A plain Mac menu with an SF Symbol and a word on every row. The top row is the
status: the loaded model, its state, and three measured facts (speed of the
last reply, memory in use, and when and how fast it loaded). A load time is
the last load of that model on this Mac; a model never loaded here says so.
Every error names the measured fact behind it and one action that fixes it.
Mference never stops a process it did not start.

States drawn: ready, reading a long prompt, answering with a queue, switching
models, first load of a model, memory pressure, starting, off, stopping, no
models, install list, installing, install paused, install done, download
stopped, not enough memory, install not finished, disk full, server not
answering, another model process, port in use, chat app missing, wrong chat
app version, Check This Mac.

Decided: Open at Login is off by default; the "Which Model Fits This Mac"
guide ships in the first version.

## First launch

A regular app window, not a menu. Six steps, each one moving picture, at most
six words and one number, in the landing site's look:

| Step | Words | What moves |
| --- | --- | --- |
| Welcome | Run the models you couldn't before. | The site's two expert grids: 175 GB against 2.36 GB |
| Your Mac | Your Mac: 24 GB. | Models drop into a memory box; the ones that fit stay |
| Pick a model | Pick one. | Three cards with a live grid and the measured speed |
| Download | Arriving. | The grid fills as expert files arrive, with an exact GB counter |
| First chat | Say hello. | The first answer streams at its measured speed |
| All set | It lives up there. | The window flies into the menu-bar icon; updates and open at login start off |

The app and the site read the same sizes and speeds (`site/data.js`).

## Terminal status

`mference status` prints one block; `--watch` redraws it in place.

```text
● ready   Qwen 3.6 35B-A3B
  speed     26.4 tokens/s   last reply
  memory    1.45 GB         16-slot profile
  waiting   0
  last load 86.4 s          from Gemma 4
installed 3 · skipped 1
  gemma-4-26b-a4b-it     14.29 GB
  qwen3.6-35b-a3b        19.55 GB  loaded
  qwen38flashnext        skipped: install not finished
```

## Automatic updates

Sparkle 2 with Mference's own windows, on the pattern of an existing menu-bar
app:

- No network use until the user turns automatic updates on or presses the
  update button.
- Checks 15 s after launch, then every hour. Updates are signed with an EdDSA
  key; an unsigned update is refused. Copies one release behind get a delta.
- An update restarts the server and unloads the model, so it installs only
  after 30 s with no model answering, reading a prompt, loading or installing.
- A manual update asks once, in a What's New window that also states the model
  reload time.
- Chats and models are never part of an update. If a release needs a model
  reinstalled, What's New names the model and its size first.
- The update list (`appcast.xml`) is published with the site by the existing
  Pages workflow; the disk image goes on the GitHub release.

## Landing site changes

On the existing page, nothing removed:

- The nav button becomes Download for Mac.
- Watch it answer: a recorded run played at its real speed beside the grid.
- Your first minute: the app's first steps as three small frames.
- A Download block at the end, and `appcast.xml` beside the page.

## What the server must add

| Fact | Today |
| --- | --- |
| Loaded model, ready or loading | `/health` |
| Speed of the last reply | Per reply only; the status feed must keep the last one |
| Memory in use | Behind `MFERENCE_DIAGNOSTICS`; must reach the status feed |
| Load time per model | Logged as `swap finished`; must be stored per model |
| Requests waiting | New |
| Prompt progress (tokens read of total) | New |
| Load a model before the first message | New request |
| Install progress (bytes, rate, layer) | New; the installer prints text only |
| Another model process, port in use, chat app version | Launcher text; must become structured results |

## Open decisions

- Publish the site and a downloadable app (a public release).
- Apple notarization, or the "Open Anyway" first-open step.
- Where the site links: this fork or the upstream repository.
