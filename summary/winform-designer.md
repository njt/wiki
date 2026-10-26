---
url: https://github.com/macauhome/winform-designer
title: "WinForms Designer"
author: macauhome
date_fetched: 2026-10-26
date_published: 2026
topics:
  - developer-tools
  - software-engineering-craft
---

macauhome's WinForms Designer is a standalone, Visual Studio–style designer for Windows Forms that runs on .NET 10 and needs neither Visual Studio nor any design-time assemblies: the toolbox, property grid, code generator and `.designer.cs` loader are all written from scratch. It opens an existing `.designer.cs` file, lets you drag, resize and wire controls on a custom-painted canvas, and generates a complete compilable project (`Form1.cs`, `Form1.designer.cs`, `.csproj`, `Program.cs`) whose output matches the layout Visual Studio itself writes — containers suspended before filling and resumed in reverse, z-order preserved, only non-default properties emitted.

The architecture is a strict model–view split: `WinFormsDesigner.Core` holds the document model (`DesignNode` tree, a `PropertyBag` that stores only values differing from control defaults), a reflection layer (`ControlTypeRegistry`, `ControlFactory`, `PropertyCatalog`) and the code generator, with no UI; the App project renders the tree into a bitmap and maps pointer gestures back onto the session. The loader parses `.designer.cs` with regexes rather than executing or compiling the source.

The README opens with "This project totally AI generated, I just burn some tokens" — making it as interesting as an artefact of agent-written software (~9,800 lines of C#) as as a tool.
