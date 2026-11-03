# Changelog

## 1.2.1
- Destination section is easier to read: the path box wraps and grows, recent/library folders are full-width wrapping rows, the folder browser has
  a clickable breadcrumb, a filter box and natural sorting; larger text; Enter in the destination box runs Preview.

## 1.2.0
- Zip/cbz archives can be moved via the Galleries list (CLI: `--galleries`). Folder-based galleries are reported as not movable.

## 1.1.0
- Images (CLI: `--images`).
- Running-jobs guard (blocks scan/clean/generate-type jobs unless overridden; UI and CLI).
- Stop button, closing the drawer stops the batch after the current file, warning before closing the tab mid-batch, clean Ctrl-C in the CLI.
- Warning when Watchtower is installed. Large selections: chunked re-check, tables capped at 500 rows.

## 1.0.1
- CLI uses only the Python standard library (no `pip install`).
- The post-move check compares paths the same way the planner does (case-insensitive for Windows paths).
- A failing post-move check no longer hides which files moved (CLI exit code `2` instead of `1`).
- UNC paths (`\\server\share`) are kept intact. Scenes without a file get their own "no file" row.
- The plugin no longer depends on `ui.esc`, `ui.fmtBytes` or Stash UI's `kb-hint` / `kb-lab` classes.

## 1.0.0
- First version: Stash UI plugin and CLI for moving scenes with Stash's `moveFiles` mutation.
