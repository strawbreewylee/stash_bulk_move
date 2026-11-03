100% vibe coded


# Bulk Move for Stash

Select many scenes, images or zip/cbz archives in [Stash](https://github.com/stashapp/stash), pick a destination folder, preview, and move them
in one go. It uses **Stash's own `moveFiles` mutation**, so Stash's database always follows the files. Nothing is moved behind Stash's back, and there is no fallback to the operating system if Stash refuses a move.

> **Requires the [Stash UI](https://discourse.stashapp.cc/t/stash-ui/17496) plugin, version 3.80.0 or newer** (the version with extension API v2 and the
> `bulk.actions` slot). The button does not appear in classic Stash. The CLI needs only Python 3.8+ and no extra packages.





## Install via Stash's plugins menu

Settings > Plugins > Add source


## Install the plugin manually

Download `bulkMove-<version>.zip` from the [Releases](https://github.com/<your-user>/stash-bulk-move/releases) page and unzip it into your /stash/plugins folder so you end up with 

-stash
--plugins
---stash-bulk-move
----bulkMove
----README.md
----stash_bulk_move.py
----stash_bulk_move.example.json

Now, 
Stash → Settings → Plugins → **Reload plugins**, then hard-refresh the browser (Ctrl+f5)
3. In Stash UI open **Scenes**, **Images** or **Galleries**, select items: **Move to folder…** appears in the selection bar.


## Using it

1. Select multiple images, scenes, or zips you'd like to move
2. On the right, next to "Delete" there will be a move button
3. Confirm the move destination on the right sidebar

## Behaviour this plugin relies on

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

## AI assistance

This project was developed with AI assistance (Claude, by Anthropic). The code was reviewed and tested by the maintainer, who takes full
responsibility for it.

## License

[MIT](LICENSE). Built on the extension API of [Stash UI](https://discourse.stashapp.cc/t/stash-ui/17496) and on Stash's GraphQL API.
