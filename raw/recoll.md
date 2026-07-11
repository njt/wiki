---
url: https://www.recoll.org/
title: Recoll
author: Jean-Francois Dockes
date_fetched: 2026-07-11
date_published: unknown (active since at least 2013, latest release 2026-06-30)
---

# Recoll: Full-Text Desktop Search Engine

Recoll is a full-text search tool for desktop environments that finds documents based on their contents as well as file names. It's built on the Xapian search engine library.

## Key Features

- **Deep indexing**: Recoll can index "an MS-Word document stored as an attachment to an e-mail message inside a Thunderbird folder archived in a Zip file"
- **Broad format support**: Handles most document formats, potentially requiring external tools for text extraction
- **Universal storage access**: Reaches "any storage place: files, archive members, email attachments, transparently handling decompression"
- **One-click opening**: Opens documents in their native editor or provides a text preview
- **Web front-end**: A WEB UI with preview/download features "can replace or supplement the GUI for remote use"
- **Powerful GUI**: "A complete, yet easy to use, Qt graphical interface"
- **Text preview**: Displays quicker text previews without opening full documents
- **PDF page-level linking**: Can "open a copy of a PDF at the right page with two clicks"

## Supported Platforms

- Linux (primary platform)
- MS Windows (via a dedicated port)
- macOS (install method updated 2021)
- Android (via an F-Droid app and companion server, added November 2024)
- Flatpak (contributed by a third party, available on Flathub)
- AppImage (built on Debian Buster, works with most Linux distros from ~2019 onward)

## Architecture & How It Works

- **Backend**: Xapian search engine library handles the indexing and retrieval engine
- **Text extraction layer**: Recoll provides a powerful layer above Xapian for extracting text from diverse formats
- **Indexing modes**: Supports a newer method "using multiple temporary indexes" (not default)
- **Configuration files**: GUI preferences stored at `~/.config/Recoll.org/recoll.ini`; indexing config at `~/.recoll/recoll.conf`
- **Language processing**: Correctly processes Tibetan using n-grams (v1.41+); improved Korean indexing; allows configuring non-alphanumeric characters as separate words (v1.39+)
- **OCR support**: "Vastly improved OCR support, with caching" (v1.26.5+)
- **Python3**: Full switch to Python3 occurred with v1.25.4
- **Debian packaging**: Split into `recollcmd` (command-line), `recollgui` (GUI), and a top-level `recoll` package depending on both

## Notable Components & Extensions

- **Recoll GSSP**: Gnome Shell Search Provider (v1.1.4 as of Feb 2026)
- **Recoll WEBUI**: Web interface for remote querying, hosted on Framagit
- **recolldroid**: Android companion server on GitLab
- **Browser extension**: Firefox add-on for Recoll WE
- **Recoll-Mutt interface**: Terminal-based search via the Links browser working with the WebUI
- **Contributed scripts**: Anki flashcard indexing, Newsboat RSS reader indexing, AC power-based indexing control for laptops

## License

Free on Linux, open source, and licensed under the GPL.

## Current Version

1.44.0 (released June 30, 2026)

## Recent News & History

| Date | Event |
|------|-------|
| 2026-07-01 | Qt/Wayland display issues: workaround is setting `QT_QPA_PLATFORM=xcb` |
| 2026-06-30 | Recoll 1.44.0 released |
| 2025-05-06 | 1.43.2 fixes a static initializer bug causing crashes with gcc 15 |
| 2024-11-28 | Android app published on F-Droid |
| 2024-11-21 | Flatpak made available on Flathub |
| 2024-03-28 | AppImage introduced |
| 2024-03-08 | Website moved to its own host at recoll.org |
| 2020-03-08 | Source repo and issue tracker moved to Framagit |
| 2013-04-30 | Lotus Notes filter and Web browser interface contributed by users |

## Documentation & Support

The project provides extensive documentation, a mailing list, and a problem tracker. The site also hosts FAQ/HowTo sections, contributed result list formats, and user-contributed articles (e.g., using Linux cgroups to limit indexing CPU usage).

## Credits

"Recoll could not exist without a rich free software environment" — the site credits the broader free software ecosystem it depends on.
