<div align="center">
  
# NK Advanced Collector Pro

<sub></sub><img width="1024" height="1024" alt="icon_256" src="https://github.com/Nitinkashyap96/NK-AdvancedCollector_Pro/blob/main/icons/icon_256.png" />



**A multi-threaded asset collection and archiving tool for Foundry Nuke**

[![Version](https://img.shields.io/badge/version-1.0.0-29BCE4?style=flat-square)](#)
[![License: GPL v3](https://img.shields.io/badge/license-GPL--3.0-blue.svg?style=flat-square)](LICENSE)
[![Nuke](https://img.shields.io/badge/Nuke-15%2B-1f6f8b?style=flat-square)](#requirements)
[![Python](https://img.shields.io/badge/python-3.10%2B-yellow?style=flat-square)](#requirements)
[![Qt](https://img.shields.io/badge/UI-PySide6%20%2F%20PySide2-444?style=flat-square)](#requirements)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-success?style=flat-square)](#requirements)

[Overview](#overview) • [Features](#key-features) • [Install](#installation) • [Quick Start](#quick-start) • [UI Guide](#interface-guide) • [Settings](#settings-reference) • [CLI](#headless--render-farm-mode) • [Docs](docs/AdvancedCollector_Pro_Manual.pdf)

</div>

---

## Overview

**NK Advanced Collector Pro** scans the current Nuke script for every node that
carries a source file path, validates that those sources actually exist (and
that there's room to copy them) **before** anything is touched, then copies
all footage into a clean, self-contained archive folder using a multi-threaded
copy engine — with live progress, pause/resume/cancel, an HTML collection
report, and full settings persistence between sessions.

It is built as an enhanced, from-scratch implementation of the "collect /
archive script" workflow popularized by tools like cragl's **smartCollect**,
re-engineered with a pre-flight validation pass, a per-node asset filter
table, token-based output paths, and a render-farm-ready headless/CLI mode.

> 📘 A full, illustrated PDF manual is available at
> [`docs/AdvancedCollector_Pro_Manual.pdf`](docs/AdvancedCollector_Pro_Manual.pdf).

---

## Key Features

| | |
|---|---|
| 🔍 **Pre-Flight Validation** | Checks every source path, missing sequence frames, duplicate paths, non-default colorspaces, and target-disk free space *before* you copy a single byte. |
| ⚡ **Parallel Archiving** | A `ThreadPoolExecutor`-backed copy engine with a configurable thread count and buffer size, plus live Pause / Resume / Cancel. |
| 🗂️ **Node Filter Table** | A sortable, searchable asset table (format, frame range, frame count, file type, size on disk) so you can selectively include/exclude individual sources before archiving. |
| 🧭 **Token-Based Target Paths** | `{project}` `{shot}` `{version}` `{artist}` `{date}` `{time}` `{script}` tokens resolve automatically, with auto-incrementing versioned folders (`_v02`, `_v03`, …) if the target already exists. |
| 🔗 **Automatic Relinking** | Copied nodes are relinked to portable, TCL-anchored relative paths (`[file dirname [value root.name]]/footage/...`) so the archive opens correctly on any machine. |
| 📄 **HTML Collection Report** | A self-contained, dark-themed `collection_report.html` with full copy log, byte counts, throughput, and the pre-flight results — generated on completion. |
| 🧩 **69 Node Classes Supported** | Read, Write, Deep, Geo, Camera, Axis, Cryptomatte, CopyCat/ML, VDB, MaterialX, and the full Nuke 17 USD/Hydra/LOP node family — see [Supported Node Types](#supported-node-types). |
| 🧱 **Gizmo → Group Conversion** | Optional pass that converts third-party Gizmos into native Groups so the archive doesn't depend on plugins the recipient may not have installed. |
| 🖥️ **Headless / Farm Mode** | The same engine runs from the command line for Deadline/Tractor/RoyalRender-style automated collection — see [Headless Mode](#headless--render-farm-mode). |
| 🎨 **Themeable Dark UI** | Built-in dark theme plus a drop-in `.qss` theme picker, sound-on-completion, and desktop notifications. |
| 💾 **Full Settings Persistence** | Every preference (threads, buffer size, theme, sound, logging, target mode, etc.) is saved to `data/settings.json` and restored on the next launch. |

---

## Requirements

| Component | Minimum |
|---|---|
| Foundry Nuke | 12.0+ (built and tested against the Nuke 17 USD/Hydra node set; gracefully degrades on older versions) |
| Python | 3.7+ (the version bundled with your Nuke install) |
| Qt bindings | `PySide6` (Nuke 14+) — automatically falls back to `PySide2` (Nuke 12–13) |
| OS | Windows, macOS, or Linux |

No external `pip` packages are required — everything used (`PySide2`/`PySide6`,
`concurrent.futures`, `hashlib`, `subprocess`, etc.) ships with Nuke's
interpreter.

---

## Installation

### Option A — Self-contained folder (recommended)

This keeps the whole tool — script, icons, sounds, styles, and settings — as
one portable, drop-in folder, which is how this repository is laid out.

1. **Download or clone** this repository.
2. **Copy the entire `AdvancedCollector_Pro/` folder** into your Nuke plugin
   path, e.g. directly into your user `.nuke` directory:

   ```
   ~/.nuke/AdvancedCollector_Pro/AdvancedCollector_Pro.py
   ~/.nuke/AdvancedCollector_Pro/menu.py
   ~/.nuke/AdvancedCollector_Pro/icons/
   ~/.nuke/AdvancedCollector_Pro/sounds/
   ~/.nuke/AdvancedCollector_Pro/styles/
   ~/.nuke/AdvancedCollector_Pro/data/
   ```

3. **Register the path** so Nuke picks up the folder at startup. Open (or
   create) `~/.nuke/init.py` and add:

   ```python
   import nuke
   nuke.pluginAddPath('./AdvancedCollector_Pro')
   ```

   `pluginAddPath` adds the folder to `sys.path` **and** tells Nuke to
   auto-execute any `menu.py` it contains — so the menu entries register
   themselves automatically, and `icons/`, `sounds/`, `styles/`, and
   `data/settings.json` are all found right next to the script (the tool
   auto-detects this and uses the folder itself as its base directory).

4. **Restart Nuke.**

### Option B — Flat / traditional `.nuke` layout

If you prefer not to touch `init.py`, you can install the pieces manually,
exactly as described in the comments at the top of `menu.py`:

1. Copy `AdvancedCollector_Pro.py` directly into `~/.nuke/AdvancedCollector_Pro.py` (flat, **not** in a subfolder).
2. Copy `icons/`, `sounds/`, `styles/`, and `data/` into `~/.nuke/AdvancedCollector_Pro/` (a same-named subfolder).
3. Copy the contents of `menu.py` into your existing `~/.nuke/menu.py` (or create one if you don't have one yet).
4. Restart Nuke.

> The tool resolves its own asset folder automatically: if `icons/` exists
> next to `AdvancedCollector_Pro.py`, that folder is used directly (Option A).
> Otherwise it falls back to `~/.nuke/AdvancedCollector_Pro/` (Option B).

### Launching the tool

Once installed, you can open it three ways:

- **Nuke menu bar:** `NK Asset Collector → Advanced Asset Collector`
- **Nodes panel / Tab menu / Node Graph right-click:** `NK Asset Collector → Advanced Asset Collector`
- **Keyboard shortcut:** `Ctrl+Shift+C`

---

## Quick Start

1. **Open the script** you want to archive in Nuke.
2. **Launch the tool** (`Ctrl+Shift+C` or via the menu).
3. On the **Collection** tab, choose your output path mode:
   - **AUTO PATH** — target folder is auto-detected next to your saved script (`.../collected/{shot}_{version}/`), with auto-version-increment if it already exists.
   - **MANUAL PATH** — type or browse to any folder, using `{project} {shot} {version} {artist} {date} {script}` tokens; click **Resolve Tokens** to preview the resolved path.
4. *(Optional)* On the **Node Filter** tab, click **Scan Nodes** to populate the
   asset table, then check/uncheck individual sources, or use **Ignore free
   floating nodes** / **Ignore disabled nodes** / **Ignore selected** to
   quickly trim the list.
5. Click **Pre-Flight Check** to validate every path, sequence frame, and the
   target disk's free space. Review the results table — fix anything flagged
   `ERROR` (or proceed anyway if you understand the risk).
6. Click **START COLLECTION**. Watch live progress, MB/s, and ETA; use
   **Pause** / **Cancel** as needed.
7. When it finishes:
   - Footage has been copied into `<target>/footage/`.
   - Collected nodes are relinked to portable relative paths.
   - Your script has been **Saved As** into `<target>/<script_name>.nk`.
   - An HTML report (`collection_report.html`) is written into the target folder, if enabled.
   - A completion dialog offers to open the report; sound/desktop notification fire if enabled.

> ⚠️ **Note on in-session changes:** collection relinks nodes (and, if enabled,
> converts Gizmos to Groups) on the script that's **currently open** in Nuke,
> then performs a *Save As* into the target folder. The original `.nk` file on
> disk at its original location is **not** modified — but the version you have
> open in the Nuke session now reflects the archived state (new relative
> paths, the new save location, and any Gizmo→Group conversions). If you want
> to keep working on the original, unarchived version, reopen it from disk
> after collecting.

---

## Interface Guide

The tool window is organized into five tabs, a live log panel, a progress
bar with ETA, and the action row (**Pre-Flight Check / START COLLECTION /
Pause / Cancel**).

### Collection tab
Target directory input (Auto/Manual), folder browser, **Detect** (re-sync to
the script's location) and **Resolve Tokens** buttons, a live "Resolved:"
preview, the auto-version-increment checkbox, and the **Selective Frame
Range** group (use each node's own first/last frame, or force one custom
global range across every collected sequence).

### Node Filter tab
A 10-column sortable/searchable table — `archive` checkbox, node name,
class, format, first/last frame, frame count, file type, size on disk, and
filename — populated by **Scan Nodes**. Bulk-action buttons let you ignore
free-floating nodes, ignore disabled nodes, or enable/ignore the current
selection. A separate checkbox controls whether `Write` node outputs are
included in scanning/archiving at all.

### Validation tab
Shows the results of the last **Run Validation** / **Pre-Flight Check** pass
in a color-coded table (red = `ERROR`, orange = `WARNING`, cyan = `INFO`),
with a summary bar above it.

### Settings tab
Two sub-tabs:
- **General** — always-on-top, recreate source paths, gizmo→group
  conversion, reveal-on-finish, HTML report generation, desktop
  notifications, concurrent thread count, copy buffer size, source path
  mode, completion sound (with a folder-reveal button so you can drop in
  your own `.wav` files), UI theme picker (drop-in `.qss` files, same
  reveal button), output path mode, an "ignore nukescripts" filter field,
  and the relative-relink toggle.
- **Logging** — verbose output, errors-only, log skipped files, per-node
  timing, and a log-level dropdown (`INFO` / `WARNING` / `ERROR` / `DEBUG`),
  plus a **Clear Log** button.

Click **Save and close** to persist every option to `data/settings.json`.

### About tab
Tool name, version badge, author credit, a short description, GitHub/LinkedIn
link cards, and a feature summary list.

---

## Settings Reference

These are the **factory defaults** the tool ships with on first launch (i.e.
before you've ever clicked *Save and close*). The bundled `data/settings.json`
is simply a sample/last-saved configuration and will be overwritten the first
time you save your own preferences.

| Key | Type | Default | Description |
|---|---|---|---|
| `always_on_top` | bool | `true` | Keeps the tool window above Nuke while it's open. |
| `recreate_paths` | bool | `true` | *(Reserved — see [Known Limitations](#known-limitations--reserved-settings))* |
| `gizmo_to_group` | bool | `false` | Converts custom Gizmos to native Groups after collection. |
| `reveal_finished` | bool | `false` | Opens the target folder in your OS file browser when collection finishes. |
| `gen_report` | bool | `true` | Writes `collection_report.html` into the target folder on completion. |
| `notify_desktop` | bool | `false` | Fires an OS desktop notification on completion (macOS/Linux). |
| `threads` | int | `min(cpu_count, 8)` | Concurrent copy worker threads (1–99). |
| `buf_size` | string (MB) | `"4"` | Copy buffer size — one of `1 / 4 / 8 / 16 / 32 / 64` MB. |
| `path_mode` | string | `"relative"` | *(Reserved — see [Known Limitations](#known-limitations--reserved-settings))* — `relative` / `absolute` / `preserve`. |
| `sound` | string | `"no sound"` | `.wav` file (from `sounds/`) played on completion, or `"no sound"`. |
| `style` | string | `"Default"` | `.qss` theme file (from `styles/`), or `"Default"` for the built-in dark theme. |
| `target_mode` | string | `"next_to_script"` | `"next_to_script"` (Auto Path) or `"custom"` (Manual Path). |
| `ignore` | string | `"backup, bckp, annotation"` | *(Reserved — see [Known Limitations](#known-limitations--reserved-settings))* |
| `relink` | bool | `true` | Relink collected nodes to relative, TCL-anchored archive paths. |
| `log_verbose` | bool | `true` | *(Reserved)* |
| `log_errors` | bool | `false` | *(Reserved)* |
| `log_skipped` | bool | `true` | *(Reserved)* |
| `log_timing` | bool | `false` | *(Reserved)* |
| `log_level` | string | `"INFO"` | *(Reserved)* — `INFO` / `WARNING` / `ERROR` / `DEBUG`. |

---

## Supported Node Types

The collection engine recognizes the file-bearing knob on **69 node
classes** spanning Nuke's native readers/writers, 3D/geo/camera nodes, the
full Nuke 17 USD/Hydra/LOP node family, and common third-party plugins —
plus **24 fallback knob names** (`filename`, `proxy`, `chan_file`,
`vdb_file`, `lut_file`, `model_file`, …) so gizmos and node variants with
non-standard knob names are still picked up.

<details>
<summary><strong>Click to expand the full supported-node table</strong></summary>

| Category | Node classes |
|---|---|
| **Image / Deep readers** | `Read`, `DeepRead`, `AudioRead`, `Precomp` |
| **Geometry / scene readers** | `ReadGeo`, `ReadGeo2`, `GeoImport`, `UsdImport`, `UsdCameraImport` |
| **Cameras & Axis** | `Camera`, `Camera2`, `Camera3`, `Axis`, `Axis2`, `Axis3` |
| **Writers** | `Write`, `DeepWrite`, `WriteGeo` |
| **Particle / cache** | `ParticleCache` |
| **Colour / OCIO / LUT** | `OCIOFileTransform`, `LUTTransform` |
| **Special file knobs** | `Vectorfield` (`vfield_file`), `ScannedGrain` (`grain_file`) |
| **Live Groups** | `LiveGroup` |
| **Third-party / plugins** | `Cryptomatte`, `DeepRecolor`, `CopyCat`, `MLClient`, `Card3D`, `ReadOFX`, `WriteOFX`, `BorisFX`, `FlameImport`, `ReadVDB`, `VDBRead`, `Render`, `DeadlineRender`, `proWrite`, `proRead`, `NukeStudio`, `TrackClip`, `ClipRead`, `GridWarp3`, `SplineWarp3` |
| **Nuke 17 USD / Hydra / LOP** | `UsdRead`, `UsdWrite`, `UsdStageRead`, `UsdStageWrite`, `UsdLayerRead`, `UsdLayerWrite`, `UsdPrimRead`, `UsdPrimWrite`, `UsdStageWithLayer`, `LookFileMaterialsIn`, `LookFileBake`, `HydraRender`, `ReadPTX`, `ReadParticles` |
| **Nuke 17 3D scene placement** | `GeoReference` (`file_path` knob) |
| **Nuke 17 geo export / transform** | `GeoExport`, `GeoTransform` (`chan_file`), `GeoCamera` (`chan_file`), `GeoEditCamera` (`chan_file`), `GeoCacheTransform`, `GeoCache` |
| **USD Material / Look** | `UsdMaterialBind`, `UsdShadeMaterial`, `MaterialX`, `ReadMaterialX` |

**Fallback knob names** checked when the primary knob is empty (and rewritten
alongside it on every matching node): `filename`, `proxy`, `vfield_file`,
`grain_file`, `fbx_file`, `abc_file`, `chan_file`, `usd_file`, `lop_path`,
`usd_path`, `file_path`, `geo_file`, `scene_path`, `layer_file`,
`material_file`, `mtlx_file`, `look_file`, `cache_file`, `ptx_file`,
`footage_file`, `input_file`, `output_file`, `lut_file`, `profile_file`,
`model_file`, `vdb_file`, `alembic_file`, `scene_file`, `data_file`.

</details>

---

## Path Tokens

Used in both the target-directory field and the auto-detected path:

| Token | Resolves to |
|---|---|
| `{project}` | The part of the script filename before the first `_` |
| `{shot}` | The part of the script filename after the first `_` |
| `{version}` | The first `vNN` pattern found in the script filename, or `v01` |
| `{artist}` | Current OS username |
| `{date}` | `YYYYMMDD` |
| `{time}` | `HHMMSS` |
| `{script}` | The script filename without extension |

---

## Headless / Render-Farm Mode

The same scan → validate → copy → relink → report engine runs without a UI,
for Deadline / Tractor / RoyalRender or any other automated pipeline trigger.
Pass `--target` (and any other flags) after `--` so Nuke's own argument
parser ignores them:

```bash
# Representative example — exact Nuke executable flags vary by version/OS,
# consult your Nuke command-line / render-farm wrapper documentation.
Nuke15.1 -t AdvancedCollector_Pro.py -- --target "/archive/shot010_v01/" --threads 8 --relink --report
```

| Flag | Description |
|---|---|
| `--target <path>` | **(required)** Destination archive folder. |
| `--threads <int>` | Concurrent copy threads (default `4`). |
| `--buf <int>` | Copy buffer size in MB (default `4`). |
| `--relink` | Relink collected nodes to portable archive paths. |
| `--report` | Write `collection_report.html` into the target folder. |
| `--gizmo` | Convert custom Gizmos to Groups before saving. |
| `--zip` | Archive the entire target folder into a `.zip` after collection. |
| `--incremental` | Accepted for forward-compatibility — see [Known Limitations](#known-limitations--reserved-settings). |

---

## How It Works

```
 1. SCAN         Walk every node with a file-bearing knob (69 classes +
                  24 fallback knob names), resolve sequence vs. single-file.
 2. VALIDATE     Pre-flight: missing frames, duplicate paths, non-default
                  colorspace, target disk free space  →  Pre-Flight dialog.
 3. COPY         ThreadPoolExecutor pool copies into <target>/footage/,
                  skip-if-identical (size + mtime) fast path, live
                  progress / MB/s / ETA, pause & cancel supported.
 4. RELINK       Collected node knobs rewritten to a portable, TCL-anchored
                  relative path:  [file dirname [value root.name]]/footage/...
 5. SAVE         nuke.scriptSaveAs() writes the script into the target
                  folder (original file on disk is untouched).
 6. REPORT       collection_report.html generated with the full copy log,
                  throughput, and pre-flight findings; optional sound /
                  desktop notification / auto-reveal in file browser.
```

### Output structure

```
<target>/
├── <script_name>.nk        # Saved-as copy of your script, relinked
├── collection_report.html  # Self-contained HTML report (if enabled)
└── footage/
    ├── <file>.ext                  # Single-file sources (FBX, ABC, audio, LUTs...)
    └── <node_name>_assets/
        └── <file>.%04d.ext         # Frame sequences, one subfolder per node
```

---

## Known Limitations / Reserved Settings

In the interest of full transparency, the following options exist in the UI
and are saved to `settings.json`, but are **not yet wired into the
collection logic** — they're reserved extension points for a future release:

- **Sources path mode** (`relative` / `absolute` / `preserve`) — the engine
  currently always relinks using the relative TCL anchor.
- **Recreate source paths** — saved, no effect yet.
- **Ignore nukescripts** filter — saved, not yet applied to scanning.
- **Logging sub-tab** (Verbose / Errors only / Skipped files / Timing /
  Log level) — saved, but the live log panel currently shows the same set
  of messages regardless of these toggles.
- **Path remapping** — both `ValidationEngine.run()` and the collection
  routine fully support `(old_prefix, new_prefix)` remap pairs, but the
  GUI's `_get_remaps()` always returns an empty list — there's no remap
  table in the UI yet.
- **CLI `--incremental`** — accepted by the headless argument parser but not
  yet connected to MD5 hash verification; headless mode always uses the
  fast size + mtime comparison. (The `_CollectionWorker` engine used by the
  GUI *does* implement MD5-verified incremental copying internally — it's
  simply not yet exposed as a checkbox.)
- **Standalone execution** — `AdvancedCollector_Pro.py` imports the `nuke`
  module unconditionally, so headless/standalone runs must use Nuke's own
  bundled Python interpreter (e.g. `Nuke -t`), not a system `python3`.

---


# Changelog

All notable changes to NK Advanced Collector Pro are documented here.

## [1.0.0] — Initial public release

### Added
- Multi-threaded collection engine (`ThreadPoolExecutor`) with configurable
  thread count and copy buffer size.
- Pre-Flight Validation engine: missing-frame detection, duplicate-path
  detection, non-default colorspace flagging, and target-disk free-space
  checking, surfaced through a dedicated Pre-Flight dialog and Validation tab.
- Node Filter tab with a sortable/searchable 10-column asset table
  (format, frame range, frame count, file type, size on disk) and bulk
  include/ignore actions.
- Support for 69 file-bearing node classes plus 24 fallback knob names,
  including the full Nuke 17 USD/Hydra/LOP node family.
- Token-based target paths (`{project} {shot} {version} {artist} {date}
  {time} {script}`) with auto-incrementing versioned output folders.
- Automatic relative relinking via a portable TCL anchor
  (`[file dirname [value root.name]]/footage/...`).
- Optional Gizmo → Group conversion pass prior to saving the archive.
- Self-contained HTML collection report with full copy log and throughput
  stats.
- Pause / Resume / Cancel controls with live progress, MB/s, and ETA.
- Completion sound, desktop notification, and auto-reveal-in-explorer
  options.
- Drop-in `.qss` UI theme picker (built-in dark theme by default).
- Full settings persistence to `data/settings.json`.
- Headless / CLI mode (`--target`, `--threads`, `--buf`, `--relink`,
  `--report`, `--gizmo`, `--zip`) for render-farm / pipeline automation.

### Known limitations
See [README → Known Limitations / Reserved Settings](README.md#known-limitations--reserved-settings)
for the full list of UI options that are persisted but not yet wired into
the collection logic (path-mode, recreate-paths, ignore-filter, logging
sub-tab, path remapping, and the CLI `--incremental` flag).



## Contributing

Issues and pull requests are welcome — in particular for the reserved
settings above. Please keep contributions ASCII-safe, cross-platform
(Windows/macOS/Linux), and consistent with the existing dark UI theme.

## License

Licensed under the **GNU General Public License v3.0** — see [LICENSE](LICENSE).

## Author

 # **Nitin Kashyap** — Junior VFX Compositor

  <a href="https://www.linkedin.com/in/nitin-kashyap-8541213a6/">
    <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" 
         alt="LinkedIn" 
         style="width: 250px; height: auto;" />
  </a>


  <a href="https://youtu.be/1U4bhtptPwo">
    <img src="https://img.shields.io/badge/YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white" 
         alt="YouTube" 
         style="width: 270px; height: auto;" />
  </a>
</div>

<sub>NK Advanced Collector Pro v1.0.0<sub></sub><img width="1024" height="1024" alt="icon_256" src="https://github.com/Nitinkashyap96/NK-AdvancedCollector_Pro/blob/main/icons/icon_256.png" />
