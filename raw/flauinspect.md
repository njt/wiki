---
url: https://github.com/FlaUI/FlaUInspect
title: FlaUInspect
author: Roemer, K. Usenko {kDg}
date_fetched: 2026-05-31
date_published: 2021-04-08
---

# FlaUInspect — Full Repo Analysis

## Project Overview

FlaUInspect is a Windows UI Automation inspection tool — essentially a spiritual successor to Microsoft's old "Inspect.exe" tool from the Windows SDK. It lets you browse the UIAutomation tree of any running Windows application, inspect element properties, patterns, and relationships, and copy element state as XML. It's built as a WPF application on .NET 8, using the FlaUI library family (FlaUI.Core, FlaUI.UIA2, FlaUI.UIA3) as its automation backend.

Version: 3.0.0 (csproj), though Changelog only goes to 1.3.0 (2021-04-08).

## File Tree & Project Size

```
FlaUInspect/
├── src/
│   └── FlaUInspect/
│       ├── FlaUInspect.sln
│       ├── FlaUInspect.csproj          — .NET 8 WPF, WinExe
│       ├── App.xaml / App.xaml.cs      — Entry point, DI, theme, overlay setup
│       ├── appsettings.json            — User settings (theme, overlay configs)
│       ├── Core/
│       │   ├── ObservableObject.cs     — MVVM base (114 lines)
│       │   ├── RelayCommand.cs         — ICommand wrapper (28 lines)
│       │   ├── RelayCommandAsync.cs    — Async command (80 lines)
│       │   ├── Editable.cs             — Clone-based edit tracking
│       │   ├── ElementOverlay.cs       — WinForms overlay rectangles (91 lines)
│       │   ├── GlobalMouseHook.cs      — Low-level mouse hook (67 lines)
│       │   ├── HoverManager.cs         — Ctrl-hover element detection (113 lines)
│       │   ├── FocusTrackingMode.cs    — UIA focus change events (41 lines)
│       │   ├── PatternItemsFactory.cs  — Pattern detail extraction (315 lines)
│       │   ├── Logger/                 — Internal logger
│       │   ├── Exporters/              — XML tree & details export
│       │   ├── Converters/             — WPF value converters (9 files)
│       │   ├── Extensions/             — String, Task, AutomationProperty
│       │   └── Behaviors/              — TreeView bring-into-view
│       ├── ViewModels/
│       │   ├── StartupViewModel.cs     — Process picker, window list (319 lines)
│       │   ├── ProcessViewModel.cs     — Element tree, patterns, modes (401 lines)
│       │   ├── ElementViewModel.cs     — Tree node, child loading (74 lines)
│       │   ├── PatternItem.cs          — Key-value property row (25 lines)
│       │   ├── SettingsViewModel.cs
│       │   └── AboutViewModel.cs
│       ├── Models/
│       │   ├── Element.cs              — Simple data class
│       │   └── ElementPatternItem.cs   — Pattern category (28 lines)
│       ├── Views/                      — XAML + code-behind (7 windows)
│       ├── Settings/                   — JSON settings, overlay config
│       ├── Controls/                   — XAML style dictionaries (15+ files)
│       ├── Themes/                     — Light, Dark theme XAML
│       └── Resources/                  — Icon XAML dictionaries (3 files)
├── build.cake                          — Cake build script (Chocolatey packaging)
├── CHANGELOG.md
└── LICENSE
```

Approximate code size: ~2,000 lines of C# + ~500 lines of XAML across the main sources.

## Architecture Deep Dive

### Application Lifecycle (App.xaml.cs)

The entry point sets up:
1. DI container (Microsoft.Extensions.DependencyInjection) with a single service: `JsonSettingsService<FlaUiAppSettings>`
2. Settings loaded from `appsettings.json`, applied to theme and overlay configs
3. Shows `StartupWindow` as the main window
4. Calls `startupViewModel.Init()` on a background thread to enumerate running windows
5. Has conditional compilation (`#if AUTOMATION_UIA3` / `#elif AUTOMATION_UIA2`) for directly launching into a specific automation mode, but these paths are commented out in the current build — the app always goes through the startup window

### StartupViewModel — Window Picker

Uses `UIA3Automation` (the default automation backend) to:
1. Enumerate all top-level windows via `GetDesktop().FindAllChildren(x => x.ByControlType(ControlType.Window))`
2. Filter out the current process and nameless windows
3. Display in a list with real-time text filtering

Has a "pick window" mode that:
1. Installs a low-level mouse hook (`WH_MOUSE_LL`)
2. Changes cursor to crosshair
3. On mouse move, finds the window under cursor via `WindowFromPoint` + `GetAncestor(GA_ROOT)`, highlights it with an overlay
4. On left-click release, selects that window
5. The hook is automatically cleaned up after 30 seconds via CancellationToken

### ProcessViewModel — Element Inspection Core

This is the main inspection window's ViewModel. Key design decisions:

**Flat Tree Model**: Instead of using WPF's `TreeView` with hierarchical items, elements are stored in a flat `ObservableCollection<ElementViewModel>`. Children are inserted inline after their parent when expanded, and removed when collapsed. This is a deliberate choice — it simplifies data binding at the cost of losing native tree behavior.

**Mode System**: Three mutually-exclusive selection modes, enforced by `SetMode()` which counts enabled modes and ensures only one is active:
1. **HoverMode**: Ctrl+Mouse hover highlights elements via `HoverManager`
2. **HighlightSelectionMode**: Selected item gets a persistent overlay highlight
3. **FocusTrackingMode**: UIA focus change events tracked via `FocusTrackingMode`

**ElementToSelectChanged**: The most complex algorithm in the codebase. Given a hovered/focused `AutomationElement`, it:
1. Walks up the tree via `ITreeWalker.GetParent()` to build a path-to-root stack
2. Reverses the stack, expanding each node along the path in the flat list
3. Inserts children inline via `ExpandElement()` 
4. Sets the final element as `SelectedItem`

**Collapse/Expand**: Children are added/removed from the flat `Elements` collection. `IsDescendantOf()` walks parent references to determine which items to remove on collapse.

### ElementOverlay — Visual Highlighting

Uses WinForms `Form` (not WPF windows) for overlays — this is the key technique for drawing rectangles over external application windows:

```csharp
// Creates borderless, transparent WinForms windows positioned over
// the target element's bounding rectangle using Win32 SetWindowPos
SetWindowPos(handle, new IntPtr(-1), x, y, width, height, 16 /*0x10*/);
ShowWindow(handle, 8);
```

Two rectangle factories:
- `FillRectangleFactory`: One rectangle covering the entire element
- `BoundRectangleFactory`: Four rectangles forming a border (top, left, right, bottom strips)

Configuration is per-overlay-type (hover, selection, pick) via `ElementOverlayConfiguration` with color, size, margin, and mode.

### HoverManager — Ctrl-Hover Detection

Static class with a 300ms `DispatcherTimer`. On each tick:
1. Check if any listeners are enabled
2. If Ctrl is held: get element at mouse position via `AutomationBase.FromPoint()`
3. Skip elements in the current process
4. If element changed: notify all registered listeners, show overlay
5. Deep try-catch blocks everywhere — resilience over debuggability

Uses a listener registry pattern: multiple windows can register `Action<AutomationElement?>` callbacks, independently enabled/disabled by `IntPtr` key (window handle). Thread-safe via `lock` on all mutations.

### PatternItemsFactory — Element Property Extraction

Maps UIA patterns to displayable key-value property rows. Has separate function registries for UIA2 and UIA3:

```csharp
private readonly KeyValuePair<PatternId, Func<AutomationElement, IEnumerable<PatternItem>>>[] _patternsUia3Func = [
    new (GridItemPattern.Pattern, AddGridItemPatternDetails),
    new (GridPattern.Pattern, AddGridPatternPatternDetails),
    // ... 14 patterns total
];
```

Three built-in categories always shown: Identification, Details, Pattern Support. Additional pattern categories shown only when the element supports them.

Text pattern handling is notable — it handles the UIA2/UIA3 difference in `MixedAttributeValue` representation using a runtime automation type check.

### FocusTrackingMode — Focus Change Events

Wraps UIA's `RegisterFocusChangedEvent` which runs on a background thread. `OnFocusChanged` callback dispatches to the UI thread via `Application.Current.Dispatcher.Invoke`. Skips events from the current process (FlaUInspect itself).

### ObservableObject — MVVM Base

Uses a `Dictionary<string, object?>` for backing fields keyed by `[CallerMemberName]` — this means all derived ViewModel properties automatically get change notification without declaring private fields. Clever, but trades type safety and discoverability for convenience.

### GlobalMouseHook — Low-Level Mouse Hook

P/Invoke `SetWindowsHookEx` with `WH_MOUSE_LL` (14) to intercept mouse movements system-wide. Used by `StartupViewModel` for the window picker. Properly unsubscribes via `UnhookWindowsHookEx`.

### Exporters

Two simple XML exporters:
- `XmlTreeExporter`: Recursively exports element tree with Name, AutomationId, ControlType, and optional XPath
- `XmlElementDetailsExporter`: Exports pattern details as XML

### Settings

JSON-based settings for theme (Light/Dark) and three overlay configurations (Hover, Selection, Pick). Each overlay has Size, Margin, OverlayColor, OverlayMode. Settings use clone-based editing via `Editable<T>` — settings are cloned, edited, and committed back.

### Dependencies

- FlaUI.Core 5.0.0 — Core UIAutomation abstractions
- FlaUI.UIA2 5.0.0 — Managed UIA2 (System.Windows.Automation) backend
- FlaUI.UIA3 5.0.0 — COM-based UIA3 backend
- Microsoft.Extensions.Configuration 10.0.1
- Microsoft.Extensions.DependencyInjection 10.0.1
- Microsoft.Extensions.Options 10.0.1

## Key Techniques

1. **WinForms overlay windows for external process highlighting**: Rather than trying to draw over other processes' windows with WPF (which can't), uses borderless WinForms forms positioned via Win32 `SetWindowPos` with `HWND_TOPMOST` (-1).

2. **Flat ObservableCollection tree**: Children inserted/removed inline rather than using hierarchical controls. Makes data binding straightforward but requires `IsDescendantOf()` parent-walking for collapse.

3. **Dictionary-backed MVVM properties**: `ObservableObject` stores all property values in a `Dictionary<string, object?>` keyed by `[CallerMemberName]`. This means derived classes need zero private fields for bindable properties — just `get => GetProperty<T>()` and `set => SetProperty(value)`.

4. **Multi-mode mutual exclusion via counting**: `SetMode()` counts enabled modes and ensures only one is active. Simple logic: `new[] { EnableHoverMode, EnableHighLightSelectionMode, EnableFocusTrackingMode }.Count(x => x) == 1`.

5. **Pattern registry with runtime dispatch**: `PatternItemsFactory` holds separate UIA2/UIA3 function registries and selects the right one based on `automationBase is UIA3Automation`.

6. **Clone-based settings editing**: `Editable<T>` wraps settings with `ICloneable.CopyTo()` for commit-on-save pattern — edit a clone, commit only when saved.

## Design Trade-offs

- **Resilience over debuggability**: Deep try-catch blocks with `// ignored` comments everywhere (HoverManager, ElementViewModel, ProcessViewModel). The app won't crash, but debugging why something silently fails is hard.
- **Flat list over hierarchical tree**: Simpler data binding but loses native tree selection/collapse animations and keyboard navigation.
- **Static HoverManager over instance-based**: Simpler to use but no isolation between windows. All process windows share one hover manager.
- **WinForms overlays over WPF**: Works for cross-process highlighting but creates a dependency on WinForms and Win32 interop.
- **UIA3 as default**: UIA3 (COM-based) is faster and more capable than UIA2 (managed), but requires the target app to be running at the same or lower integrity level. UIA2 is more broadly compatible.
- **No unit tests visible**: The build.cake references `DotNetTest` but no test files exist in the repo. Testing UI automation tools is inherently hard, but the lack of tests means changes rely entirely on manual verification.
