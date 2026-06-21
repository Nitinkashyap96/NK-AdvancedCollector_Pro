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
