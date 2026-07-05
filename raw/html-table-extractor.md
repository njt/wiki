---
url: https://simonwillison.net/2026/Jun/29/html-table-extractor/
title: "HTML table extractor"
author: Simon Willison
date_fetched: 2026-07-05
date_published: 2026-06-29
---

# HTML table extractor

Simon Willison introduces a new tool in his collection of paste-conversion utilities. The tool accepts pasted rich text from browsers — specifically content containing embedded HTML tables — and automatically detects each table, displaying a preview before allowing export.

The tool supports exporting tables in **five formats**: HTML, Markdown, CSV, TSV, and JSON. The post includes a screenshot showing the interface with tabs for each format, with TSV selected and a "Copy" button visible.

Willison demonstrates the tool by suggesting users try pasting the entire Wikipedia page for the "List of cities and towns in the San Francisco Bay Area" into it.

## Related Work

He notes he recently rebuilt his [Rich text to markdown](https://tools.simonwillison.net/rich-text-to-markdown) tool, "to add support for tables and generally improve the UI." The commit for that rebuild is referenced on GitHub.

## Update: Wikipedia Integration

Willison discovered that Wikipedia exposes an open CORS API for retrieving rendered HTML of any page. He provided a [demo link](https://tools.simonwillison.net/cors-fetch#url=https%3A%2F%2Fen.wikipedia.org%2Fw%2Fapi.php%3Faction%3Dparse%26page%3DList_of_cities_and_towns_in_the_San_Francisco_Bay_Area%26prop%3Dtext%26format%3Djson%26origin%3D%2A) and notes he "had Codex add the ability to search Wikipedia for a page and then automatically import and display any tables."

## Links & References

- Tool URL: https://tools.simonwillison.net/html-table-extractor
- Wikipedia API demo: https://tools.simonwillison.net/cors-fetch (with params for the Bay Area cities page)
- GitHub commit (rich-text-to-markdown rebuild): https://github.com/simonw/tools/commit/f278e977751dbc1948baedfc2f26b6de870f60e6
- Codex Gist: https://gist.github.com/simonw/f226fe96f464ec7d81d6996cb466436d
- Rich text to markdown tool: https://tools.simonwillison.net/rich-text-to-markdown

## Key Quotes

- "Yet another in my growing collection of paste-conversion tools."
- "I had Codex add the ability to search Wikipedia for a page and then automatically import and display any tables from that page."

Tags: html, tools, wikipedia, cors
