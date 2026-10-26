# WinForms Designer

macauhome's WinForms Designer is a standalone, Visual Studio–style designer for Windows Forms built on .NET 10 — no Visual Studio, no design-time assemblies, no `IDesignerHost`. The toolbox, property grid, code generator and `.designer.cs` parser are all implemented from scratch across ~9,800 lines of C#. Its first README line is a candid artifact note: "This project totally AI generated, I just burn some tokens." That makes it doubly interesting: a genuinely useful reimplemention of one of the hardest pieces of IDE machinery, written almost entirely by an agent.

---

It fills a real gap. Visual Studio's WinForms designer has never shipped outside Visual Studio; anyone editing forms in VS Code, on CI, or in a minimal environment was stuck. This project parses standard `.designer.cs` files, renders an editable surface, and writes back a project that compiles and — per the README — round-trips through Visual Studio's own designer.

## Architecture

Strict two-project split (`WinFormsDesigner.slnx`):

- **`src/WinFormsDesigner.Core`** — pure model, reflection and codegen, no UI. `Model/DesignNode.cs` is the control tree: each node holds a `Type`, a `PropertyBag` of *non-default* property values, an event-wiring dictionary, and children in WinForms z-order convention (index 0 = front-most). `Model/PropertyBag.cs` mirrors VS behaviour by storing only values that differ from the control default, which is what keeps generated code free of noise. `Codegen/CSharpFormGenerator.cs` (925 lines) emits the designer file, code-behind and whole-project scaffolding; `Reflection/ControlTypeRegistry.cs` is the toolbox catalogue with VS-style camel-case naming (`button1`), and `ControlFactory` instantiates live controls to read their defaults.
- **`src/WinFormsDesigner.App`** — the editor. `Designer/DesignSession.cs` (1,362 lines) owns document, live control tree, selection and undo history; `Designer/DesignerSurface.cs` (1,079 lines) is a **model-painted canvas** — live controls are never child windows; the tree is rendered into a bitmap and pointer input is hit-tested against the model, then applied back through the session.

## Key techniques

- **Regex parsing instead of Roslyn.** `DesignerFileLoader.cs` (719 lines) rebuilds a design document from an existing `.designer.cs` using ~20 compiled regexes: field declarations, `new Type()` instantiations, property assignments, `Controls.Add/AddRange` (including TableLayoutPanel column/row overloads), `ToolStrip` item adds, and `event += handler` wiring. It never compiles or executes the source — deliberate, since executing untrusted form code is a non-starter. The cost is fragility against unusual formatting, and the repo carries a `PARSE_ERRORS.log` admitting it.
- **Round-trip fidelity as the spec.** The generator emits controls instantiated first, suspends every container layout before filling and resumes in reverse order, and writes `Controls.Add` calls in z-order so stacking survives the trip. `AlwaysWrittenProperties = ["Name", "TabIndex", "Location", "Size"]` captures exactly what VS treats as load-bearing; everything else is diffed against live-control defaults.
- **Undo as full snapshots.** `DesignSession` serialises the whole tree into `SessionSnapshot`/`NodeSnapshot` records per committed edit rather than tracking inverse operations — simple and correct, at the cost of memory for very large forms.
- **Auto-scaling correctness.** Generated forms carry `AutoScaleMode = Font` plus the recorded `AutoScaleDimensions` (`ComputeAutoScaleDimensions` measures the actual font), so layouts scale like VS-produced ones.
- **Verification proportional to the risk.** A `tests/ParserSmoke` harness dispatches real mouse messages at the canvas to check parsing, rendering, container-relative insertion, z-order and undo — but the README is honest that interactive dragging and the VS round-trip still need a human at a desktop.

## Design decisions

The trade that defines the project is **compatibility over capability**: it targets the exact file format Visual Studio writes rather than inventing a richer intermediate format, which buys round-tripping with the entire existing WinForms ecosystem at the price of a parser that can never be fully general. Relatedly, it optimises for **zero dependencies** — the published exe is a self-contained 49 MB single file, and the property metadata is a hand-maintained table (`PropertyMetadataTable.cs`) rather than a reflection scan over attributes, keeping behaviour predictable but coverage finite. The model-painted canvas sacrifices native control interactivity during editing for total control over rendering, hit-testing and alignment guides — the same trade high-end canvas editors (Figma, etc.) make.

## Comparison notes

- The "totally AI generated" README is a live specimen for [[TextForge Case Study]]: both show agent-written .NET software shipped with discipline, but this repo is the rawer end — one author, tokens burned, structure still unusually clean (strict Core/App split, doc comments throughout).
- It is the build-side complement to [[Win Dev Skills]]: Microsoft's skills package teaches agents to *write* WinUI/WinForms code against ground-truth metadata tools, while this project is an agent-written *tool* for the same ecosystem — the metadata table it hand-maintains is exactly the kind of ground truth that package indexes automatically.
- The document-as-single-source-of-truth architecture (model painted to bitmap, gestures re-enter through the session) is a classic instance of the patterns [[Software Engineering Craft]] collects — and the regex-over-Roslyn parser is a defensible simplicity-over-generality call in the same spirit.

---
*Sources: [[raw/winform-designer]], [[summary/winform-designer]]*
*Last updated: 2026-10-26*
