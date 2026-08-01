---
url: https://www.recoll.org/
title: "Recoll"
author: Jean-Francois Dockes
date_fetched: 2026-07-11
---

Recoll is an open-source (GPL) full-text desktop search engine built on top of
the Xapian search library. It indexes document *contents*, not just filenames,
and can reach deep into nested structures — an MS Word attachment inside an
email inside a Thunderbird folder archived in a Zip file, for example.

It runs on Linux (primary), Windows, macOS, and Android (via an F-Droid app
added in late 2024). A Flatpak and an AppImage are also available. The current
release is 1.44.0 (June 2026).

Architecturally, Recoll adds a text-extraction layer above Xapian that handles
dozens of document formats. It provides both a Qt-based desktop GUI and a web
interface (Recoll WEBUI) for remote querying. Notable extensions include a
Gnome Shell Search Provider, a Firefox add-on, and a Mutt-friendly terminal
interface. Indexing configuration lives in `~/.recoll/recoll.conf`; GUI prefs
in `~/.config/Recoll.org/recoll.ini`.

The project has been actively maintained since at least 2013. Its source repo
and issue tracker moved to Framagit in 2020, and the website moved to its own
domain (recoll.org) in 2024.
