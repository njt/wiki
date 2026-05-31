# FlaUInspect

A Windows UI Automation inspection tool for browsing the UIA tree of any running application. Think of it as a modern, open-source replacement for Microsoft's Inspect.exe — the tool accessibility and automation developers have been using for decades. Built on the FlaUI library family (UIA2 and UIA3 backends) as a WPF .NET 8 desktop app.

FlaUInspect matters because Windows UI Automation remains a critical capability for accessibility testing, test automation, and robotic process automation — and Microsoft's own tools are aging and harder to find. FlaUInspect bridges the gap with a polished, configurable, actively-maintained alternative.

---

## Architecture

MVVM WPF monolith with clear layering:

- **Views** (`Views/`): XAML windows with minimal code-behind. `ProcessWindow.xaml` (434 lines) is the main inspection view with a split pane: element tree on the left, property details on the right. Uses extensive XAML style dictionaries from `Controls/`.
- **ViewModels** (`ViewModels/`): `StartupViewModel` handles process/window selection with a mouse-hook-based picker. `ProcessViewModel` (401 lines) is the core — manages the element tree, three selection modes, and pattern data loading.
- **Models** (`Models/`): Thin data classes — `Element` (name/automationId/controlType/children) and `ElementPatternItem` (pattern categories with key-value children).
- **Core** (`Core/`): Services and utilities — `HoverManager`, `ElementOverlay`, `GlobalMouseHook`, `PatternItemsFactory`, `FocusTrackingMode`, exporters, converters, `ObservableObject` MVVM base class.

The app flow: `App.xaml.cs` → `StartupWindow` (pick a process) → `ProcessWindow` (inspect its UI tree).

The tree is displayed as a **flat `ObservableCollection`**, not a WPF TreeView. When a node is expanded, its children are inserted inline after the parent; collapsing removes them. `IsDescendantOf()` walks parent references to determine what to remove.

## Key Techniques

**WinForms overlay windows for cross-process highlighting** (`ElementOverlay.cs`). WPF windows can't draw over external processes' windows, so FlaUInspect creates borderless WinForms `Form` instances positioned via Win32 `SetWindowPos` with `HWND_TOPMOST`. Two factory functions: `FillRectangleFactory` (one rectangle covering the element) and `BoundRectangleFactory` (four border strips — top, left, right, bottom). Each overlay type (hover, selection, pick) gets its own configurable color, margin, size, and mode.

**Dictionary-backed MVVM properties** (`ObservableObject.cs`). Instead of declaring private fields for every bindable property, all values are stored in a `Dictionary<string, object?>` keyed by `[CallerMemberName]`:
```csharp
protected T? GetProperty<T>([CallerMemberName] string? propertyName = null) {
    return _backingFieldValues.TryGetValue(propertyName, out var value) ? (T)value! : default;
}
```
This eliminates boilerplate but sacrifices type safety and IDE discoverability.

**Ctrl-hover element detection** (`HoverManager.cs`). A static `DispatcherTimer` fires every 300ms. When Ctrl is held, it calls `AutomationBase.FromPoint(Mouse.Position)` to get the element under cursor. Uses a listener registry pattern — multiple windows can independently register/unregister callbacks and enable/disable interest by window handle. Thread-safe via `lock` on all mutations.

**Path-to-root tree navigation** (`ProcessViewModel.ElementToSelectChanged()`). Given a hovered/focused element, walks up via `ITreeWalker.GetParent()` to build a `Stack<AutomationElement>`, then reverses the stack, expanding each ancestor node's children inline until the target is reached. Handles circular references and null guards at each step.

**UIA2/UIA3 pattern dispatch** (`PatternItemsFactory.cs`). Maintains separate function registries for UIA2 and UIA3 patterns (14 each), selected at runtime via `automationBase is UIA3Automation`. Text pattern handling has special-cased `MixedAttributeValue` detection because UIA2 and UIA3 represent it differently (managed object vs. COM object).

**Low-level mouse hook for window picking** (`StartupViewModel.PickWindowAsync()`). Installs `WH_MOUSE_LL` to intercept mouse events system-wide, changes cursor to crosshair, highlights windows under cursor, and returns the root window handle on click. Auto-cleans up after 30 seconds via `CancellationToken`.

## Design Decisions

**Resilience over debuggability.** Deep try-catch blocks with `// ignored` comments pervade the codebase. The app prioritizes not crashing over surfacing errors. This is pragmatic for a tool that touches unstable external process state, but makes debugging silent failures difficult.

**Flat list over hierarchical TreeView.** Trading native tree behavior (keyboard nav, collapse animations, lazy loading) for simpler data binding. Children load eagerly on expand — fine for typical apps, could freeze on enormous trees (e.g., browser DOMs).

**Static HoverManager over per-instance.** Simpler to use but means all inspection windows share one hover manager. If you had two ProcessWindows open, enabling hover on one enables it on both (mitigated by per-window enable/disable via `IntPtr` keys).

**UIA3 as default automation backend.** UIA3 (COM-based) is faster and supports more patterns than UIA2 (managed), but requires the target process to run at the same or lower integrity level. UIA2 is more broadly compatible with elevated processes.

**Clone-based settings editing** (`Editable<T>`). Settings are cloned, edited in the UI, and committed back on save. This prevents partial edits from persisting if the user cancels.

**No tests.** Despite `build.cake` referencing `DotNetTest`, there are no test files in the repo. UI automation tools are notoriously hard to test, but combined with the deep catch blocks, this means changes rely entirely on manual verification.

## Comparison Notes

Unlike [[Maestro (UI Testing)]], which is a mobile/web test runner with a YAML DSL and visual inspector, FlaUInspect is purely an inspection tool — it reads and displays UI element data but doesn't execute tests. It's closer to browser DevTools' element inspector, but for native Windows applications.

Unlike Microsoft's Inspect.exe (part of the Windows SDK), FlaUInspect supports both UIA2 (managed) and UIA3 (COM) backends, has a modern WPF UI with dark mode, supports XML export with XPath, and is open source. Microsoft's tool is UIA3-only and hasn't been meaningfully updated in years.

Unlike the [[surf-cli]] browser automation approach, which controls Chrome via a Go native host, FlaUInspect works at the Windows accessibility API level — it can inspect any application with UIA support (Win32, WPF, WinForms, UWP), not just browsers.

## Tags

#tool #project #windows #automation #ui-testing #accessibility

---
*Sources: [[raw/flauinspect]]*
*Last updated: 2026-05-31*
