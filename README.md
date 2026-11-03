# Bulk Move for Stash

Select many scenes, images or zip/cbz archives in [Stash](https://github.com/stashapp/stash), pick a destination folder, preview, and move them
in one go. It uses **Stash's own `moveFiles` mutation**, so Stash's database always follows the files. Nothing is moved behind Stash's back,
and there is no fallback to the operating system if Stash refuses a move.

It comes in two parts:

| | What | When to use it |
|---|---|---|
| **Stash UI plugin** (`bulkMove/`) | A **Move to folder…** button in the selection bar of Stash UI | Normal use: select, click, preview, confirm |
| **CLI** (`cli/stash_bulk_move.py`) | Give IDs and a folder on the command line | Scripting, or if you prefer a terminal |

> **Requires the [Stash UI](https://discourse.stashapp.cc/t/stash-ui/17496) plugin, version 3.80.0 or newer** (the version with extension API v2 and the
> `bulk.actions` slot). The button does not appear in classic Stash. The CLI needs only Python 3.8+ and no extra packages.

## Install the plugin

1. Download `bulkMove-<version>.zip` from the [Releases](https://github.com/<your-user>/stash-bulk-move/releases) page and unzip it into your Stash
   `plugins` directory (next to Stash's `config.yml`), so you end up with `plugins/bulkMove/bulkMove.yml` and `plugins/bulkMove/web/stashui.js`.
   The `web/` folder must stay: the yml maps it to the plugin's assets. (Or copy the `bulkMove/` folder from this repository.)
2. Stash → Settings → Plugins → **Reload plugins**, then hard-refresh the browser (Ctrl+Shift+R).
3. In Stash UI open **Scenes**, **Images** or **Galleries**, select items: **Move to folder…** appears in the selection bar.

If the button is missing on a page, open the browser console: the plugin logs `[Bulk move] bulk.actions ctx.kind = …` for every list it is offered
on, and Stash UI's Plugins page lists any errors of the module.

## Using it

Type or browse to the destination → **Preview** (every file, its new path, total size, conflicts and warnings) → **Move N file(s)** → confirm.
Afterwards Stash is asked again where each file is, and the result is checked against what was expected. Moved and failed files are listed
separately, with a **Copy report** button.

* **Destination box:** the whole path is shown (it wraps and grows). Recent destinations and your library folders are one click away.
* **Folder browser:** clickable breadcrumb, a **Filter folders…** box, natural sorting.
* **Scenes with several files:** by default all files of a scene move together; you can choose to move only the primary file.

## Safety checks

* **Running jobs.** Before moving, and again right before the first file, the job queue is checked. A job whose description mentions scan, clean,
  generate, auto-tag, identify, import/export, migrate, optimise, backup or anonymise **blocks** the move: *Move* stays disabled until you tick
  *Move anyway* (CLI: `--ignore-jobs`). Other active jobs are only listed. This is a heuristic on free-text job descriptions. It is not repeated
  during a batch, on purpose: Stash itself (or a plugin such as Watchtower) may start a scan because of your own moves.
* **Stopping a batch.** *Close* becomes *Stop after this file* while moving; closing the drawer does the same. The file in progress finishes,
  the rest is reported as "not attempted" and stays where it was. The browser asks before the tab is closed during a batch. CLI: Ctrl-C does the same.
* **No overwriting, ever.** Name clashes in the destination (and between selected files) are detected in the preview and skipped.
* **One call per file.** A single `moveFiles` call is all-or-nothing, so each file gets its own call: "moved" and "failed" are exact, a failure
  leaves that file untouched, and nothing is retried.
* **Watchtower.** If the Watchtower plugin is installed and enabled, the preview warns that its file monitor and automatic renaming may react to
  the moves. Pause its monitoring and auto-rename for the duration of a batch.

## Images

Works like scenes. Images that belong to a **gallery** are flagged: moving the file out of the gallery's folder may change that membership, and
if "create galleries from folders" is on, a later scan may create a gallery for the destination folder. Images *inside* a zip cannot be moved
individually. Large selections are fine: tables show the first 500 rows, everything is still handled and reported.

## Archives (zip / cbz)

Select **galleries** on the Galleries list. A zip/cbz gallery *is* the archive file, so moving the gallery moves the archive and every image inside
it. Notes:

* **Only .zip and .cbz** (Stash's gallery extensions, a setting) are known to Stash. Other archives (.rar, .7z, .tar …) have no entry in Stash's
  database, so `moveFiles` cannot move them. Move those with your file manager.
* **Folder-based galleries** have no file and are listed as "cannot move". Stash moves files, not folders.
* After moving an archive, open one of its galleries to check it still loads. The re-check verifies the archive's own path, not each image in it.
  If images look missing, scan the destination folder.

## CLI

```bash
export STASH_URL=http://localhost:9999
export STASH_API_KEY=...            # Settings > Security > API Key (only if Stash requires login)

python cli/stash_bulk_move.py --scenes 101,102,103 --destination "/data/videos/Sorted/Studio-A"
python cli/stash_bulk_move.py --images 501,502    --destination "/data/images/Trip"
python cli/stash_bulk_move.py --galleries 7,8     --destination "/data/archives"
```

Options: `--dry-run` (preview only) · `--primary-only` · `--ignore-jobs` · `--browse [PATH]` (list the folders Stash sees) · `--report out.json` ·
`--log-file` · `--config file.json` (see `stash_bulk_move.example.json`; **keep it out of git**, `.gitignore` already does) · `--url`, `--timeout`.
Credentials come only from the environment or the config file, never from the command line, and are never printed or logged. Without `--scenes`
etc. it asks. Exit code: `0` ok · `1` error, nothing changed · `2` some moves failed, were not attempted, or could not be verified.

## Paths: Stash's view, not yours

The destination must be the path **as Stash sees it**. If Stash runs in Docker with `-v /mnt/media/videos:/data/videos`, use
`/data/videos/Sorted/Studio-A`, not `/mnt/media/...`. The same goes for encrypted or virtual drives (for example a Cryptomator vault): use the
mounted path Stash is configured with. Stash only moves into its configured library folders; anything else is refused, and both tools check
this first. Missing destination folders are created by Stash. On slow mounts raise `--timeout`: a timed-out call is reported as failed and the
re-check tells you whether the file actually moved.

## Behaviour this plugin relies on

<!-- Author: confirm these against the Stash version you test with, and name it in "Testing status" below. -->

* `moveFiles(input: { ids, destination_folder })` takes **file** IDs. Scenes, images and galleries are resolved to their files first.
* A single `moveFiles` call is all-or-nothing, including folders it created. A name clash fails the call; nothing is overwritten.
* Files keep their basename and file ID. Moving a file into the folder it is already in is a no-op ("already there"). Files inside zip archives
  cannot be moved.
* Stash does **not** move side files (funscripts, caption files) that lie next to a video.

## Limitations

* **Funscripts and caption files next to a video are not moved by Stash.** Scenes that have them are flagged in the preview; move those side
  files yourself or the scene loses them.
* The "already exists" pre-check uses what Stash has indexed. A file on disk that was never scanned is caught by Stash itself at move time and
  reported as a failed move; the file stays where it was.
* Not transactional across files: if a run is stopped or interrupted, files already moved stay moved. Stash and disk stay consistent for each
  file, and the report says which.
* Scenes, images and zip/cbz archives only. English only.

## Testing status

Tested by the maintainer on: Stash `<fill in version>` · Stash UI `<fill in version>` · `<fill in OS / Docker or not>`.

During development the planning, verification and job-guard logic, a simulated drawer, and the CLI (against a mock Stash server) were covered by
automated tests. These do not replace testing against a real library. Points reported as **unconfirmed** so far, and welcome as issue reports:

* that Stash UI offers the selection bar (and reports the kind as `image` / `gallery`) on the Images and Galleries lists;
* that the images inside a moved zip follow it automatically;
* the exact Stash image schema (`visual_files`, with a fallback to `files`), and the `{ raw: true }` option of `stashui.gql`.

Back up your Stash database (Settings → Tasks → Backup) before a large batch, and try two or three files first.

## AI assistance

This project was developed with AI assistance (Claude, by Anthropic). The code was reviewed and tested by the maintainer, who takes full
responsibility for it.

## License

[MIT](LICENSE). Built on the extension API of [Stash UI](https://discourse.stashapp.cc/t/stash-ui/17496) and on Stash's GraphQL API.
