<p align="center">
  <img src="screenshots/app-icon.png" alt="JCV Karaoke Pack Editor Logo" width="192"/>
</p>

<h1 align="center">JCV Karaoke Pack Editor</h1>

<p align="center">
  <img src="https://img.shields.io/badge/platform-Windows%20%7C%20Android%20%7C%20Mac-blue.svg" alt="Platforms">
  <img src="https://img.shields.io/badge/Flutter-3-02569B.svg?logo=flutter&logoColor=white" alt="Flutter">
  <img src="https://img.shields.io/badge/packs-KRPK%20%2B%20KRCP-yellow.svg" alt="Pack formats">
</p>

<p align="center">
Create, open, and maintain open karaoke packs — <strong>.krpk</strong> song libraries and sibling <strong>.krcp</strong> album covers — for <strong>JCV Karaoke</strong>. Built for large libraries (tens of thousands of songs): folder import, song-quality checks, lyric fixes, metadata cleanup, online tags and covers, and CSV / song-book export.
</p>

> **Status:** Preview page — screenshots and feature overview first. **Source code will be published after review.** Build scripts below describe the upcoming tree.

## 📊 Features & Screenshots

### 💻 Main window (Windows desktop)

![Pack Editor — main list and toolbar](screenshots/03-main-list.png)
_Dark theme · Compact list · search across ~50k songs · toolbar grouped into File, Add songs, Clean up, Online and Export._

### ✏️ Edit song

![Pack Editor — Edit song dialog](screenshots/02-edit-song.png)
_Code, title, artist, album, plus pack extra JSON: year, language, genre, origin / country._

### 🖼️ Album cover

![Pack Editor — Album cover dialog](screenshots/01-album-cover.png)
_One JPEG per artist + album (shared by every song on that album). Preview, Replace, or Delete._

### 🔍 Key capabilities

**📁 File**
- **New / Open / Save / Save As** `.krpk`, unsaved `*` marker, discard prompt, Ctrl+S / Ctrl+Shift+S
- **Fast, safe saving of big packs:** unchanged songs are copied as stored (no re-compress), new songs are packed and streamed to disk, and the file replaces the pack only when complete — a failed save never damages your library. A ~50,000-song library with 2,000 new songs saves in about 16 s, with a **progress bar**
- **Undo** stack covering clean / merge / bulk / lyric-fix / save-related edits

**📥 Add songs**
- **Add files / drag-drop** (MIDI · KAR · MP3) and **Import folder** (recursive, CRC skip if already in the pack, preview with editable tags / origin)
- **Auto song #:** unused codes from file names or sequential numbering
- **Fix MIDI on import** (on by default): ALL CAPS lyrics → normal case, `~` written for a hyphen → `-`, missing artist found and names spelled like the pack — the original files are never changed
- **Merge pack:** CRC-deduped merge from another `.krpk`

**🧹 Clean up**
- **Check songs folder…** — grades every MIDI / KAR in a folder (Good / Fair / Poor / No lyrics / Broken) by what the player really shows:
  - lyrics present and **on time** with the melody (with a fixable offset when a whole lyric track is shifted)
  - **non-lyric text on screen** (KAR headers, instrument names, credits, web addresses), odd symbols, wrong code page
  - **explicit words** (English + Filipino), ALL CAPS, lines too long, no line breaks
  - duplicates by title + artist; renames to `NNNNNN - Title - Artist` with unused numbers (never clashing with the open pack) and an undo list
  - **Fix lyrics (N)** in place (originals backed up first) and **Clean copy…** (below)
- **Clean copy…** — copies only the good songs of a folder that are **not already in the open pack** (same file checksum, or the same notes with different tags), not duplicates of each other, not explicit, best version per song — with lyrics fixed and names corrected against the pack. Songs with a pack song's title + artist but other music go to a separate review folder. Review list before anything is copied, plus a CSV report
- **Fix lyrics…** — for every MIDI / KAR in the open pack: `~` hyphens → `-` and ALL CAPS → sentence case, one character for one (notes and timing untouched, every file checked byte by byte). Applied on Save, Undo reverts
- **Clean titles:** local Title Case + artist spelling; optional MusicBrainz
- **Genre QA:** religious denylist preview — clear false-positive genre tags
- **Find duplicates:** code / title+artist / CRC
- **Reassign codes…** bulk renumber · **Diff with pack…** CRC index diff (added / removed / changed tags)
- **Compare with .idx…:** match song numbers against another karaoke catalog's `songs.idx` (titles and numbers only)

**🌐 Online** (opt-in)
- **Fetch tags** (title / artist / album / genre / year) and **Download covers** into the `.krcp` (one image per album), in the background with parallel workers
- Sources in your order: **iTunes**, **Deezer**, **MusicBrainz + Cover Art Archive**, **Last.fm** (free API key) and **Spotify** (your own developer app)
- **Pause / Continue downloads** button; downloads also pause automatically while saving (and the ones in flight finish first, so no tag is lost)
- Optional auto-fetch for new songs and **auto-save after downloads**

**📤 Export**
- **Song list CSV** (`song_list.csv` + `song_tags.csv`), **Song book PDF…**
- **Save covers pack / Export covers…** (`.krcp`, baked beside the `.krpk` on Save)
- **Custom meta DB:** sparse `custom_meta.sqlite` keyed by payload CRC32 — **Seed custom DB**, **Custom DB CSV**, per-song revert; Save bakes it into the pack

**🎛️ Settings & more**
- Compact / comfortable density, font size, Dark / Light / OLED themes, online sources + keys, parallel downloads, custom meta DB path
- **🧩 Shared format core:** the same `krpk_core` library as the player — KRPK / KRCP layout and song checks stay in sync

| Toolbar group | Commands |
| :--- | :--- |
| File | New pack · Open pack… · Save · Save as… |
| Add songs | Add files… · Import folder… · Merge pack… |
| Clean up | Clean titles · Genre QA · Find duplicates · Reassign codes… · Diff with pack… · Check songs folder… · Fix lyrics… · Compare with .idx… |
| Online | Fetch tags · Download covers |
| Export | Song list CSV · Song book PDF… · Save covers pack · Export covers… · Seed custom DB · Custom DB CSV |
| Chips | Auto song # · Fix MIDI on import · Custom meta only · Density |

### 🧰 Command-line tools

The same song checks run without the editor (from `packages/krpk_core`, `dart run bin/<tool>.dart`):

| Tool | What it does |
|------|--------------|
| `check_songs.dart <folder>` | Grade a folder, plan clean numbered names, CSV report (rename with `--apply`) |
| `clean_songs.dart <folder> --out <clean> --pack library.krpk [--review <folder>]` | The **Clean copy** above |
| `fix_songs.dart <folder> --pack library.krpk [--apply]` | Fill missing artists, fix ALL CAPS / `~` lyrics in place (dry run by default) |
| `rename_songs.dart <folder> --pack library.krpk [--apply]` | Correct titles / artists against the pack (dry run by default, undo list) |

## 📦 Pack formats

### KRPK — song pack

One randomly accessible file: **catalog index + MIDI/MP3/KAR payloads**. Magic `KRPK`, little-endian header, seekable index by song code, optional zlib payloads, CRC32 of raw payload. Extra UTF-8 JSON per entry for album / year / country (and more).

```
Offset  Field
0       "KRPK"
4       u16 version (=1)
6       u16 flags (bit0 = zlib payloads)
8       u32 songCount
12      u32 indexBytes
16      u64 payloadOffset
32…     index entries (code, title, artist, language, genre, mediaType, sizes, crc32, extra…)
payloadOffset…  concatenated song blobs
```

### KRCP — covers pack

Sibling file keyed by **artist + album** (not song code). Magic `KRCP`, same header skeleton; payloads are center-square JPEG (~512px). Ship next to the song pack:

```
library.krpk
library.krcp
```

Full binary layout docs ship with the source release.

## 🚀 Installation & Setup

> Build scripts and Flutter project land with the **source upload** (after review). Outline below is what the published tree will use.

### Prerequisites

* 🧰 **Flutter** SDK on `PATH` (desktop + Android tooling as needed)
* 🪟 **Windows:** Visual Studio C++ desktop workload for Windows builds
* 🤖 **Android:** Android SDK / Studio for APK builds (`ANDROID_HOME` or default Studio SDK)
* 🍎 **Mac:** Xcode + Flutter macOS desktop enabled

### Windows (recommended)

```bat
REM Rebuild Debug and launch
run-windows.bat

REM Launch existing Debug build only
run-windows.bat run

REM Rebuild Release and launch
run-windows.bat release
```

| Script | Purpose |
|--------|---------|
| `run-windows.bat` | `flutter build windows --debug`, then launch the Debug exe |
| `run-windows.bat run` | Launch existing Debug exe (no rebuild) |
| `run-windows.bat release` | Rebuild Release, then launch |

Settings and the custom meta DB live in `%APPDATA%\JC Villalon\JCV Karaoke Pack Editor\` (moved there automatically from the old "JCV's Karaoke Pack Editor" folder).

### Android APK

```bat
REM Clean + release split-per-abi, then open output folder
build-apk.bat

build-apk.bat debug
build-apk.bat universal
build-apk.bat noclean release
```

| Arg | Effect |
|-----|--------|
| *(none)* | Clean + release **split-per-abi** |
| `debug` / `release` | Choose mode |
| `universal` | Single fat APK |
| `noclean` | Skip `flutter clean` |

### Mac

```bash
flutter pub get
flutter run -d macos
# or
flutter build macos
```

## 👥 Community & Support

### Getting Help

- **Issues:** [GitHub Issues](https://github.com/jcvsite/Karaoke-Pack-Editor/issues) for bug reports and feature requests
- **Discussions:** [GitHub Discussions](https://github.com/jcvsite/Karaoke-Pack-Editor/discussions) for questions

❤️ **Support:**

This project was created and is maintained by **JC Villalon** ([@jcvsite](https://github.com/jcvsite)). It's a personal project provided "as is", and while I aim for it to be useful, I cannot guarantee timely support or fixes for all issues.
If you use any information or code from this project, please consider giving credit by linking back to this repository and starring it on GitHub!

*   **Primary Method:** Please **open an Issue** on the GitHub repository for bug reports, questions, or feature requests.
*   **🐛 Bug Reports:** Include:
    *   Platform (Windows / Android / Mac) and Flutter version (once source is published)
    *   A clear description of the problem and steps to reproduce it
    *   Screenshots if applicable
    *   On Windows, the error log if the editor showed an error: `%APPDATA%\JC Villalon\JCV Karaoke Pack Editor\editor_errors.log`

*   If you find this project useful, consider showing your appreciation:
    *   [Buy Me a Coffee](https://coff.ee/jcvsite)
    *   [PayPal](https://paypal.me/jcvsite)

🤝 **Contributing:**

Contributions will be welcome once the source tree is published.

**By JC Villalon**

---

**GitHub**: [https://github.com/jcvsite/Karaoke-Pack-Editor](https://github.com/jcvsite/Karaoke-Pack-Editor)

Open pack formats (**KRPK** / **KRCP**) and this editor are part of the **JCV Karaoke** project.
