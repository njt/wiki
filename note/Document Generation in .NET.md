# Document Generation in .NET

A 2026 field guide by Deepika Kathiravan surveying four approaches to programmatic document generation in C#/.NET: code-built, HTML-to-PDF, headless format libraries, and template-based with an integrated editor. The central question isn't technical — it's organizational: when the document needs to change, who can change it?

---

## The Four Approaches

### Code-Built (Programmatic)
Documents are assembled element-by-element — paragraphs, fonts, tables, images — entirely in C#. Maximum control, zero external dependencies, but "the layout lives inside your source." Every wording tweak becomes a developer ticket. Fine for machine-generated reports where the format *is* the code; disastrous for customer-facing documents that marketing wants to revise on a Tuesday afternoon.

### HTML-to-PDF
Generate HTML with a templating engine, render to PDF. Familiar to web developers, good for teams that already have HTML/CSS muscle. The fatal flaw: "HTML and CSS were designed for screens, not paginated print." Page breaks, headers/footers, exact margins, and reliable numbering are fighting the medium. Business users are still locked out — markup isn't Word.

### Headless Format Libraries
Libraries that manipulate DOCX/PDF formats in code without a UI. Mail merge support is common. Broad format coverage, strong server-side performance. But "there is no editing surface at all" — pair with a separate editor and maintain the integration yourself. The template may be a separate file, but without an editor in the toolkit, changing it still falls to developers.

### Template-Based with Document Model and Editor
Templates designed in any word processor, populated at runtime from JSON. The same document model serves both authoring and generation. "It can run fully headless on the server for automated generation, while still offering an editor when users need to author or edit documents." This is the approach the author advocates, and it's TX Text Control's product pitch — but the architectural argument stands independently.

## The Template Ownership Problem

> "when the document needs to change, who can change it?"

This is the article's sharpest insight, and it generalizes well beyond document generation. It's the same question that separates agent workflows built by engineers from those built by domain experts. Code-built and HTML-to-PDF approaches put the template in source code — every adjustment is a developer ticket, a code review, and a deployment. Headless libraries move the template to a separate file but keep the editing burden on developers. Only a template-based model with an editor lets business users revise content "without touching application code."

The mapping is clean:
- **Documents fully defined in code** → code-built or HTML-to-PDF
- **Server-only transformation, no human authoring** → headless format library
- **Humans need to author, edit, or review documents** → template-based with editor
- **Full lifecycle (author → generate → render → sign)** → template-based is "the lowest-maintenance path"

## The TX Text Control Stack

The article is a vendor piece — TX Text Control sells this four-stage pipeline — but the code sample is concrete enough to evaluate:

```csharp
using TXTextControl.DocumentServer;

TXTextControl.LoadSettings ls = new TXTextControl.LoadSettings()
{
    ApplicationFieldFormat = TXTextControl.ApplicationFieldFormat.MSWord
};

string jsonData = File.ReadAllText("data.json");

using (TXTextControl.ServerTextControl tx = new TXTextControl.ServerTextControl())
{
    tx.Create();
    tx.Load("template.docx", TXTextControl.StreamType.WordprocessingML, ls);

    using (MailMerge mailMerge = new MailMerge())
    {
        mailMerge.TextComponent = tx;
        mailMerge.MergeJsonData(jsonData);
    }

    TXTextControl.SaveSettings saveSettings = new TXTextControl.SaveSettings();
    tx.Save("result.pdf", TXTextControl.StreamType.AdobePDFA, saveSettings);
}
```

`ServerTextControl` runs headless — no Office, no UI, pure server-side engine. `MergeJsonData` handles single objects or arrays for mail merge. `StreamType.AdobePDFA` targets archival PDF/A; `AdobePDF` for standard PDF. The API surface is small and coherent: load, merge, save. That's a good sign.

## Critical Analysis

**The vendor shape fits the thesis.** The author's preferred approach happens to be what her employer sells. But the taxonomy itself — code-built, HTML-to-PDF, headless library, template-with-editor — is genuinely useful and vendor-neutral. You can map any .NET document library (iText, IronPDF, Aspose, QuestPDF, Open XML SDK, DocuSign) onto these four categories.

**The template ownership question is the real value.** Most document generation comparisons obsess over performance, format coverage, and API ergonomics. This one asks who maintains the template after launch. That's the question that determines whether your document generation pipeline is a feature or a maintenance albatross.

**What's missing:** No mention of open-source options (QuestPDF, MigraDoc, Open XML SDK), no cost comparison, no discussion of licensing models. For a "developer's guide," the absence of the freely-available baseline is a gap. The article also skips over the editor-integration complexity — embedding a document editor in an ASP.NET Core app with React/Angular is nontrivial, and the "just add the editor" framing understates the engineering effort.

**The sign stage is aspirational.** Positioning e-signatures as "a stage of the process rather than an additional platform to integrate" is elegant, but in practice e-signature compliance (audit trails, identity verification, legal validity across jurisdictions) is a whole product category. Bundling it into the document generation SDK is an ambitious claim.

**The four approaches map cleanly onto modern agent architecture.** Code-built is like programmatic tool calling; HTML-to-PDF is like prompt-based templates that break on edge cases; headless libraries are like headless browser automation; template-with-editor is like an agent harness with a human-in-the-loop editing surface. The same ownership question — who changes it when it needs to change? — applies to agent workflows just as much as to invoice templates.

---

*Sources: [[raw/csharp-document-generation-developer-guide-for-dotnet]]*
*Last updated: 2026-07-18*
