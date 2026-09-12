---
url: https://www.devleader.ca/2026/07/24/realworld-roslyn-analyzer-examples-in-c-naming-null-checks-and-log-validators
date_fetched: 2026-09-13
---

**Roslyn analyzer examples** are everywhere in the documentation -- but most of them stop at "hello, diagnostic." They show you enough to understand the API and then leave you to figure out how a production-quality analyzer actually looks. That gap is expensive when your team needs something working by Thursday.

This article fills that gap. You'll find three complete, copy-pasteable **roslyn analyzer examples** that your team can drop into a project immediately: a naming convention enforcer for async methods, a null guard detector for public APIs, and a structured logging validator that catches the most expensive logging mistake in production C# codebases. Each roslyn analyzer example includes the full `DiagnosticAnalyzer` plus, where relevant, a `CodeFixProvider`. Each also includes a "How It Works" walkthrough explaining the key decisions, a note on edge cases and false positives, and test patterns you can use directly.

If you're just getting started with the Roslyn API, the Roslyn Analyzers complete guide covers the fundamentals before diving into production patterns. For building your very first custom rule from scratch, see building your first analyzer.

## Example 1: Naming Convention Enforcer -- Async Methods Must End in "Async"

The first of these roslyn analyzer examples enforces the `Async` suffix convention -- one of the most consistently violated rules in .NET codebases, especially in older code that predates widespread async/await adoption. This analyzer detects any method that returns `Task`, `Task<T>`, `ValueTask`, or `ValueTask<T>` but whose name doesn't end in `Async`, and provides a one-click code fix to rename it.

### Why This Matters

Async methods without the `Async` suffix mislead callers at a glance. When a developer sees `DoWork()` in a code review, the missing suffix makes it look synchronous -- reviewers routinely overlook the `Task` return type and approve fire-and-forget bugs that surface in production under load. The built-in Roslyn IDE warnings only flag methods that carry the `async` keyword; methods that return `Task` or `ValueTask` without it slip through without any enforcement. In a team codebase with dozens of contributors making dozens of daily commits, naming drift is steady and invisible -- until it becomes a code review bottleneck where every PR generates the same comment thread. Automating this rule eliminates the entire class of feedback entirely.

### The DiagnosticAnalyzer

The analyzer class registers a syntax node action on method declarations and checks the return type and name suffix:

```
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp;
using Microsoft.CodeAnalysis.CSharp.Syntax;
using Microsoft.CodeAnalysis.Diagnostics;
using System.Collections.Immutable;
namespace TeamAnalyzers.Naming;
[DiagnosticAnalyzer(LanguageNames.CSharp)]
public sealed class AsyncMethodNamingAnalyzer : DiagnosticAnalyzer
{
    public const string DiagnosticId = "TEAM001";
    private static readonly DiagnosticDescriptor Rule = new(
        id: DiagnosticId,
        title: "Async method name should end in 'Async'",
        messageFormat: "Method '{0}' returns a Task type but does not end in 'Async'",
        category: "Naming",
        defaultSeverity: DiagnosticSeverity.Warning,
        isEnabledByDefault: true,
        description: "Methods returning Task, Task<T>, ValueTask, or ValueTask<T> should have names ending in 'Async' per .NET conventions.");
    public override ImmutableArray<DiagnosticDescriptor> SupportedDiagnostics =>
        ImmutableArray.Create(Rule);
    public override void Initialize(AnalysisContext context)
    {
        context.ConfigureGeneratedCodeAnalysis(GeneratedCodeAnalysisFlags.None);
        context.EnableConcurrentExecution();
        context.RegisterSyntaxNodeAction(AnalyzeMethod, SyntaxKind.MethodDeclaration);
    }
    private static void AnalyzeMethod(SyntaxNodeAnalysisContext context)
    {
        var method = (MethodDeclarationSyntax)context.Node;
        var methodName = method.Identifier.Text;
        // Already compliant -- skip
        if (methodName.EndsWith("Async", StringComparison.Ordinal))
        {
            return;
        }
        var returnType = context.SemanticModel.GetTypeInfo(method.ReturnType).Type;
        if (returnType is null)
        {
            return;
        }
        if (!IsTaskLikeType(returnType))
        {
            return;
        }
        var diagnostic = Diagnostic.Create(
            Rule,
            method.Identifier.GetLocation(),
            methodName);
        context.ReportDiagnostic(diagnostic);
    }
    private static bool IsTaskLikeType(ITypeSymbol type)
    {
        // Strip generic wrapper to get the open type name
        var typeName = type is INamedTypeSymbol named && named.IsGenericType
            ? named.ConstructedFrom.ToDisplayString()
            : type.ToDisplayString();
        return typeName is
            "System.Threading.Tasks.Task" or
            "System.Threading.Tasks.Task<TResult>" or
            "System.Threading.Tasks.ValueTask" or
            "System.Threading.Tasks.ValueTask<TResult>";
    }
}
```
### The CodeFixProvider

The fix renames the method by appending `Async`. Because renaming a method must update all call sites, we use `Renamer.RenameSymbolAsync` from the Workspaces layer rather than a simple text replacement.

```
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CodeActions;
using Microsoft.CodeAnalysis.CodeFixes;
using Microsoft.CodeAnalysis.CSharp.Syntax;
using Microsoft.CodeAnalysis.Rename;
using System.Collections.Immutable;
using System.Composition;
namespace TeamAnalyzers.Naming;
[ExportCodeFixProvider(LanguageNames.CSharp, Name = nameof(AsyncMethodNamingCodeFixProvider))]
[Shared]
public sealed class AsyncMethodNamingCodeFixProvider : CodeFixProvider
{
    public override ImmutableArray<string> FixableDiagnosticIds =>
        ImmutableArray.Create(AsyncMethodNamingAnalyzer.DiagnosticId);
    public override FixAllProvider? GetFixAllProvider() =>
        WellKnownFixAllProviders.BatchFixer;
    public override async Task RegisterCodeFixesAsync(CodeFixContext context)
    {
        var root = await context.Document.GetSyntaxRootAsync(context.CancellationToken);
        if (root is null)
        {
            return;
        }
        var diagnostic = context.Diagnostics[0];
        var diagnosticSpan = diagnostic.Location.SourceSpan;
        // Find the method identifier token at the diagnostic location
        var methodDecl = root.FindToken(diagnosticSpan.Start)
            .Parent?
            .AncestorsAndSelf()
            .OfType<MethodDeclarationSyntax>()
            .FirstOrDefault();
        if (methodDecl is null)
        {
            return;
        }
        var currentName = methodDecl.Identifier.Text;
        var newName = currentName + "Async";
        context.RegisterCodeFix(
            CodeAction.Create(
                title: $"Rename to '{newName}'",
                createChangedSolution: ct =>
                    RenameMethodAsync(context.Document, methodDecl, newName, ct),
                equivalenceKey: nameof(AsyncMethodNamingCodeFixProvider)),
            diagnostic);
    }
    private static async Task<Solution> RenameMethodAsync(
        Document document,
        MethodDeclarationSyntax methodDecl,
        string newName,
        CancellationToken cancellationToken)
    {
        var semanticModel = await document.GetSemanticModelAsync(cancellationToken);
        if (semanticModel is null)
        {
            return document.Project.Solution;
        }
        var symbol = semanticModel.GetDeclaredSymbol(methodDecl, cancellationToken);
        if (symbol is null)
        {
            return document.Project.Solution;
        }
        return await Renamer.RenameSymbolAsync(
            document.Project.Solution,
            symbol,
            new SymbolRenameOptions(),
            newName,
            cancellationToken);
    }
}
```
This is a complete, production-quality naming convention enforcer. The key insight here is using `Renamer.RenameSymbolAsync` -- it ensures every reference across the entire solution gets updated atomically, not just the declaration.

### How It Works

The analyzer registers a `RegisterSyntaxNodeAction` for `MethodDeclaration` syntax nodes, which fires once per method declaration in the compilation. For each method, the callback uses the semantic model to resolve the return type symbol and checks whether it matches one of the four task-like types. If the method name doesn't already end with `Async`, the diagnostic is reported at the method identifier's token location specifically -- not the return type or the entire declaration -- so the IDE underlines exactly the name and nothing else. The code fix uses `Renamer.RenameSymbolAsync` from the Workspaces layer rather than a simple text substitution. This distinction matters: the method may be called from dozens of sites across multiple files and projects. `RenameSymbolAsync` traverses the full solution graph and updates every reference in a single atomic operation, ensuring no call site is left referencing the old name.

### Edge Case: Interface Implementations and Overrides

This rule should be suppressed for methods that implement an interface member or override a base class method -- those signatures are defined upstream and cannot be renamed unilaterally. Add this guard at the top of `AnalyzeMethod` before the name check:

```
var symbol = context.SemanticModel.GetDeclaredSymbol(method) as IMethodSymbol;
if (symbol?.IsOverride == true || symbol?.ExplicitInterfaceImplementations.Length > 0)
{
    return;
}
```
This prevents false positives on common patterns like `IHostedService.StartAsync` overrides or explicit interface implementations where the `Async` suffix is already part of the interface contract defined by the framework.

## Example 2: Null Guard Enforcer -- Detect Missing ArgumentNullException.ThrowIfNull

The second roslyn analyzer example in this collection uses IOperation-based analysis to inspect method bodies for a matching `ArgumentNullException.ThrowIfNull` call. Public API methods accepting nullable reference parameters should guard those parameters at entry. For a deeper look at why `IOperation` analysis is more reliable than syntax-tree analysis for this kind of check, see the article on IOperation analysis.

### Why This Matters

Missing null guards on public API parameters are a leading source of `NullReferenceException` bugs in production .NET services. The crash typically surfaces inside a deeply nested call stack -- far from the unguarded entry point -- making it slow to trace and expensive to reproduce. Traditional code review is inconsistent: reviewers catch some missing guards and miss others under time pressure, and no linter fires on the absence of a guard call. This analyzer shifts detection entirely to compile time, making the violation visible at the exact call site where the unguarded parameter is declared. The developer sees the warning before the code ever runs, before it reaches staging, and long before it reaches production.


Note:`ArgumentNullException.ThrowIfNull`is available from .NET 6 onwards. For earlier targets, use`if (param is null) throw new ArgumentNullException(nameof(param))`as the guard idiom.

```
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp;
using Microsoft.CodeAnalysis.CSharp.Syntax;
using Microsoft.CodeAnalysis.Diagnostics;
using Microsoft.CodeAnalysis.Operations;
using System.Collections.Immutable;
namespace TeamAnalyzers.NullGuard;
[DiagnosticAnalyzer(LanguageNames.CSharp)]
public sealed class NullGuardAnalyzer : DiagnosticAnalyzer
{
    public const string DiagnosticId = "TEAM002";
    private static readonly DiagnosticDescriptor Rule = new(
        id: DiagnosticId,
        title: "Public method parameter missing null guard",
        messageFormat: "Parameter '{0}' in public method '{1}' is nullable but lacks ArgumentNullException.ThrowIfNull",
        category: "Reliability",
        defaultSeverity: DiagnosticSeverity.Warning,
        isEnabledByDefault: true,
        description: "Public methods with nullable reference parameters should guard them with ArgumentNullException.ThrowIfNull.");
    public override ImmutableArray<DiagnosticDescriptor> SupportedDiagnostics =>
        ImmutableArray.Create(Rule);
    public override void Initialize(AnalysisContext context)
    {
        context.ConfigureGeneratedCodeAnalysis(GeneratedCodeAnalysisFlags.None);
        context.EnableConcurrentExecution();
        context.RegisterOperationBlockAction(opBlockCtx =>
        {
            if (opBlockCtx.OwningSymbol is not IMethodSymbol method) return;
            if (method.DeclaredAccessibility != Accessibility.Public) return;
            var methodBody = opBlockCtx.OperationBlocks.OfType<IBlockOperation>().FirstOrDefault();
            if (methodBody is null) return;
            var nullableParams = method.Parameters
                .Where(p => p.Type.IsReferenceType &&
                            p.NullableAnnotation == NullableAnnotation.Annotated &&
                            !p.HasExplicitDefaultValue)
                .ToList();
            if (nullableParams.Count == 0) return;
            CheckNullGuards(opBlockCtx, method, nullableParams, methodBody);
        });
    }
    private static void CheckNullGuards(
        OperationBlockAnalysisContext context,
        IMethodSymbol method,
        List<IParameterSymbol> nullableParams,
        IBlockOperation block)
    {
        // Collect all parameter names guarded by ThrowIfNull in this block
        var guardedNames = CollectGuardedParameterNames(block);
        foreach (var param in nullableParams)
        {
            if (guardedNames.Contains(param.Name))
            {
                continue;
            }
            // Report on the method identifier, not the parameter declaration
            var methodSyntax = method.DeclaringSyntaxReferences
                .FirstOrDefault()
                ?.GetSyntax(context.CancellationToken) as MethodDeclarationSyntax;
            var location = methodSyntax?.Identifier.GetLocation()
                           ?? Location.None;
            context.ReportDiagnostic(Diagnostic.Create(
                Rule,
                location,
                param.Name,
                method.Name));
        }
    }
    private static HashSet<string> CollectGuardedParameterNames(IBlockOperation block)
    {
        var guarded = new HashSet<string>(StringComparer.Ordinal);
        foreach (var statement in block.Operations)
        {
            // Pattern: ArgumentNullException.ThrowIfNull(param);
            if (statement is not IExpressionStatementOperation exprStmt)
            {
                continue;
            }
            if (exprStmt.Operation is not IInvocationOperation invocation)
            {
                continue;
            }
            if (!IsThrowIfNull(invocation.TargetMethod))
            {
                continue;
            }
            // The first argument is the value being guarded
            var firstArg = invocation.Arguments.FirstOrDefault();
            if (firstArg?.Value is IParameterReferenceOperation paramRef)
            {
                guarded.Add(paramRef.Parameter.Name);
            }
        }
        return guarded;
    }
    private static bool IsThrowIfNull(IMethodSymbol method)
    {
        return method.Name == "ThrowIfNull" &&
               method.ContainingType.ToDisplayString() == "System.ArgumentNullException";
    }
}
```
This analyzer intentionally checks only the top-level statements in the method body. Null guards should appear at the very start of a method -- not buried inside conditionals. If your team prefers guards inside `if` blocks or a different pattern, adjust `CollectGuardedParameterNames` accordingly.

### How It Works

The analyzer uses `RegisterOperationBlockAction` to receive the completed operation block for each method. This fires once per method -- after the entire block has been analyzed -- rather than once per nested block, which eliminates false-positive duplicate diagnostics on correctly-guarded methods that contain `if` statements or `using` blocks. The `IOperation` API layer is the right choice here because it represents semantic intent rather than textual form -- different syntax patterns that all call `ArgumentNullException.ThrowIfNull` produce the same `IInvocationOperation` tree shape, so the check works regardless of how the caller wrote the call. The implementation walks only top-level statements in the block, collecting parameter names from the first argument of each matching `ThrowIfNull` invocation into a `HashSet<string>`. After the walk, any nullable reference parameter whose name is absent from the set triggers a diagnostic reported at the method identifier's location -- giving the IDE a focused underline on the method name where the guard is missing.

### False Positives to Handle

Two parameter categories should be excluded to prevent noise. First, parameters with a default value of `null` are intentionally nullable -- callers legitimately pass `null` and a null guard would throw on valid input. Guard against this with `p.HasExplicitDefaultValue && p.ExplicitDefaultValue is null`. Second, parameters already guarded by an early `if (param is null) throw` pattern appear in the operation tree as an `IIsNullOperation` inside an `IConditionalOperation`. Recognizing this pattern in `CollectGuardedParameterNames` alongside the `ThrowIfNull` check eliminates false positives on codebases that use both guard styles.

## Example 3: Structured Log Validator -- Ban String Interpolation in Log Messages

The third example targets one of the most common structured logging mistakes -- string interpolation in log calls -- which silently destroys log queryability in production.

### Why This Matters

This is the most impactful of the three **roslyn analyzer examples**. Structured logging with `ILogger` requires message templates like `"Processing order {OrderId}"` -- not `$"Processing order {orderId}"`. String interpolation bypasses the structured data entirely and destroys log queryability. It also inflates log storage: an interpolated string bakes the variable value directly into the message body, defeating the aggregation dashboards that group events by message template. Seq, Datadog, Elastic, and every other modern log aggregator depend on message template consistency to make sense of high-volume log data. A single developer who habitually reaches for string interpolation in log calls can silently corrupt an entire service's observability data -- and the problem is completely undetectable at runtime until observability costs spike or a log query returns zero results for a known event. Many teams discover this pattern only during a post-incident review, after days of interpolated strings have already polluted the index. This roslyn analyzer example fires as an **error** -- not a warning -- because letting it slip into production has real cost.

```
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp;
using Microsoft.CodeAnalysis.CSharp.Syntax;
using Microsoft.CodeAnalysis.Diagnostics;
using Microsoft.CodeAnalysis.Operations;
using System.Collections.Immutable;
namespace TeamAnalyzers.Logging;
[DiagnosticAnalyzer(LanguageNames.CSharp)]
public sealed class StructuredLogInterpolationAnalyzer : DiagnosticAnalyzer
{
    public const string DiagnosticId = "TEAM003";
    private static readonly DiagnosticDescriptor Rule = new(
        id: DiagnosticId,
        title: "ILogger message must not use string interpolation",
        messageFormat: "Log message for '{0}' uses string interpolation -- use a message template with structured parameters instead",
        category: "Logging",
        defaultSeverity: DiagnosticSeverity.Error,
        isEnabledByDefault: true,
        description: "String interpolation in ILogger calls destroys structured logging. Use message templates with named holes like {OrderId} instead of $"{orderId}".");
    public override ImmutableArray<DiagnosticDescriptor> SupportedDiagnostics =>
        ImmutableArray.Create(Rule);
    public override void Initialize(AnalysisContext context)
    {
        context.ConfigureGeneratedCodeAnalysis(GeneratedCodeAnalysisFlags.None);
        context.EnableConcurrentExecution();
        context.RegisterOperationAction(AnalyzeInvocation, OperationKind.Invocation);
    }
    private static void AnalyzeInvocation(OperationAnalysisContext context)
    {
        var invocation = (IInvocationOperation)context.Operation;
        if (!IsLoggerMethod(invocation.TargetMethod))
        {
            return;
        }
        // The message/format argument is typically the first string argument.
        // For LogInformation/LogWarning/etc., it's the first parameter named "message".
        // For Log(LogLevel, ...), it's the parameter named "message" as well.
        var messageArg = invocation.Arguments
            .FirstOrDefault(a =>
                a.Parameter?.Name == "message" ||
                a.Parameter?.Name == "messageFormat");
        if (messageArg is null)
        {
            return;
        }
        // Check if the argument's syntax is an interpolated string expression
        var argSyntax = messageArg.Syntax;
        if (argSyntax is ArgumentSyntax { Expression: InterpolatedStringExpressionSyntax })
        {
            context.ReportDiagnostic(Diagnostic.Create(
                Rule,
                argSyntax.GetLocation(),
                invocation.TargetMethod.Name));
        }
    }
    private static bool IsLoggerMethod(IMethodSymbol method)
    {
        // Target Microsoft.Extensions.Logging.ILogger extension methods
        // and direct ILogger.Log calls
        var containingType = method.ContainingType.ToDisplayString();
        if (containingType == "Microsoft.Extensions.Logging.LoggerExtensions")
        {
            return method.Name is
                "LogTrace" or "LogDebug" or "LogInformation" or
                "LogWarning" or "LogError" or "LogCritical";
        }
        // Also catch direct ILogger.Log(LogLevel, EventId, ...) calls
        if (method.ContainingType.AllInterfaces.Any(i =>
                i.ToDisplayString() == "Microsoft.Extensions.Logging.ILogger"))
        {
            return method.Name == "Log";
        }
        return false;
    }
}
```
The key design choice here is checking the **syntax** of the argument rather than its semantic type. An interpolated string expression (`$"..."`) is always `InterpolatedStringExpressionSyntax` in the syntax tree -- there's no ambiguity, and no semantic model lookup is required for the check itself. This makes the analyzer fast and avoids false negatives from unusual type coercions.

### How It Works

The analyzer registers `RegisterOperationAction` on `OperationKind.Invocation`. For each invocation, `IsLoggerMethod` checks whether the target method belongs to `Microsoft.Extensions.Logging.LoggerExtensions` and is one of the six standard log-level helper methods, or whether it is a direct `ILogger.Log` call via a type implementing `ILogger`. The argument named `message` or `messageFormat` is then extracted from the invocation's argument list using the parameter name -- not a positional index -- so the check is robust against optional parameters and overload variations. Rather than inspecting the semantic type of the argument (which would be `string` for both a literal and an interpolated string, indistinguishable at that level), the analyzer checks the underlying syntax node. If the argument's expression is `InterpolatedStringExpressionSyntax`, the diagnostic fires. This rule fires as an **error** rather than a warning because, in any service that depends on structured log queryability, there is no acceptable use case for string interpolation in production log calls -- warnings get deferred; errors force the fix. Teams where structured logging is optional or partially adopted should downgrade the severity to `Warning` in their `.editorconfig` configuration.

### Extending Coverage

The current `IsLoggerMethod` implementation handles `LoggerExtensions` helper methods and direct `ILogger.Log` calls. To cover `ILogger<T>` typed loggers accessed through DI, extend the interface check: the existing `AllInterfaces.Any(...)` condition already partially handles this, but verify it resolves the open generic `ILogger<T>` correctly. For `ILoggerFactory.CreateLogger` patterns -- where a logger is obtained from a factory and then used immediately -- add a second `OperationKind.Invocation` pass that tracks the declared type of the logger variable and applies the same `IsLoggerMethod` check to calls made on that variable. These two extensions give the analyzer comprehensive coverage across every common ILogger usage pattern in a .NET codebase.

## Choosing Which Example to Start With

Not every team needs all three at once. Use this decision matrix to identify the highest-value starting point for your specific situation.

| Signal | Start With | 
|---|---|
| Team is large (10+ devs), code review is a bottleneck | TEAM001(Naming) -- automatic enforcement beats manual review comments | 
| Public SDK or library with external consumers | TEAM002(Null Guard) -- API contract violations surface at compile time | 
| Microservices with Seq, Datadog, or Elastic | TEAM003(Logging) -- one bad interpolation per service multiplies fast | 
| Codebase is >5 years old, lots of legacy code | TEAM001thenTEAM003-- naming debt is visible, logging debt is invisible | 
| Greenfield .NET 10 project | TEAM002+TEAM003together -- both are cheap to adopt from day one | 
| CI fails frequently on code review nits | TEAM001-- analyzers as errors eliminate the entire category of naming PRs | 

The logging validator (TEAM003) delivers disproportionate value in microservice architectures. A single developer who instinctively reaches for string interpolation in a log call will produce dozens of bad log lines before a reviewer notices. Catching it as a compile error eliminates that entire category of problem.

TEAM002 (null guard) is the right starting point for teams building SDKs or libraries with external consumers. Public API parameter validation is a contract -- when it's missing, the contract is implicit and unverifiable. Detecting the violation at the method declaration rather than at the crash site compresses the feedback loop from hours to seconds, and it signals to consumers that the library takes API hygiene seriously.

For teams enforcing SOLID principles in your codebase, combining TEAM001 and TEAM002 into a single "API hygiene" package creates a consistent forcing function across all public types.

## Combining Examples into a Shared Analyzer Package

Multiple analyzers ship best as a single NuGet package. One package, one version number, one reference in `Directory.Build.props`. Here's how to structure it.

### Project Layout

All three analyzers live in a single project with this structure:

```
TeamAnalyzers/
├── TeamAnalyzers.csproj
├── Logging/
│   └── StructuredLogInterpolationAnalyzer.cs
├── Naming/
│   ├── AsyncMethodNamingAnalyzer.cs
│   └── AsyncMethodNamingCodeFixProvider.cs
└── NullGuard/
    └── NullGuardAnalyzer.cs
```
### Shared Rule ID Registry

Define all rule IDs in one place to prevent collisions as the package grows.

```
namespace TeamAnalyzers;
/// <summary>
/// Central registry of all rule IDs in the TeamAnalyzers package.
/// IDs follow the format TEAM{NNN}.
/// </summary>
internal static class RuleIds
{
    // Naming category
    public const string AsyncMethodNaming = "TEAM001";
    // Reliability category
    public const string NullGuard = "TEAM002";
    // Logging category
    public const string StructuredLogInterpolation = "TEAM003";
}
```
### Diagnostic Category Constants

Alongside `RuleIds.cs`, a companion `DiagnosticCategory.cs` centralizes the category strings used in every `DiagnosticDescriptor`. This prevents typos and ensures IDE filter labels remain consistent across all rules in the package. Without it, one developer writes `"Naming"` and another writes `"naming"`, creating split filter groups in Visual Studio's Error List that are confusing for consumers.

```
namespace TeamAnalyzers;
/// <summary>
/// Category strings used in DiagnosticDescriptor definitions.
/// Centralizing these ensures consistent IDE filter labels and prevents
/// typos when adding new rules to the package.
/// </summary>
internal static class DiagnosticCategory
{
    public const string Naming      = "Naming";
    public const string Reliability = "Reliability";
    public const string Logging     = "Logging";
}
```
Update each `DiagnosticDescriptor` in the three analyzer classes to reference `DiagnosticCategory.Naming`, `DiagnosticCategory.Reliability`, and `DiagnosticCategory.Logging` instead of raw string literals. This is a small change with a meaningful payoff: when a fourth rule is added to the package, the author picks from an explicit list of known categories rather than guessing the exact casing and spelling used by the existing rules. It also makes cross-rule refactoring trivial -- renaming a category is a one-line change in `DiagnosticCategory.cs` rather than a search-and-replace across multiple files.

### .csproj Configuration

The project file needs to output the analyzer DLL, not a regular assembly reference.

```
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <RootNamespace>TeamAnalyzers</RootNamespace>
    <AssemblyName>TeamAnalyzers</AssemblyName>
    <!-- Required for Roslyn analyzer packages -->
    <EnforceExtendedAnalyzerRules>true</EnforceExtendedAnalyzerRules>
    <IncludeBuildOutput>false</IncludeBuildOutput>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
    <!-- NuGet metadata -->
    <PackageId>YourOrg.TeamAnalyzers</PackageId>
    <Version>1.0.0</Version>
    <Description>Team coding standard analyzers for naming, null guards, and structured logging.</Description>
    <DevelopmentDependency>true</DevelopmentDependency>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.14.0" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp.Workspaces" Version="4.14.0" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.3.4" PrivateAssets="all" />
  </ItemGroup>
  <!-- Pack the analyzer DLL into the correct NuGet analyzers folder -->
  <ItemGroup>
    <None Include="$(OutputPath)$(AssemblyName).dll"
          Pack="true"
          PackagePath="analyzers/dotnet/cs"
          Visible="false" />
  </ItemGroup>
</Project>
```
Once you've built and published this package, consumers add it to `Directory.Build.props` at the repo root:

```
<ItemGroup>
  <PackageReference Include="YourOrg.TeamAnalyzers"
                    Version="1.0.0"
                    PrivateAssets="all" />
</ItemGroup>
```
All projects in the repository inherit the analyzers automatically. No individual `.csproj` changes required. The single package reference in `Directory.Build.props` also means you control the version once -- a bump in one file rolls the updated rules out to every project on the next build, with no risk of individual projects running a stale version of the rules.

For the full guide on packaging and distribution, the article on distributing your analyzer covers NuGet feed setup, versioning, and internal distribution strategies.

## Testing All Three Examples

Tests for these roslyn analyzer examples follow the `Microsoft.CodeAnalysis.Testing` pattern. The article on testing your analyzers covers the full test harness setup. Below are the specific test patterns for each roslyn analyzer example.

### Testing TEAM001 -- Async Naming

Testing the async naming rule verifies both the violation case and the passing case:

```
using Microsoft.CodeAnalysis.CSharp.Testing;
using Microsoft.CodeAnalysis.Testing;
using TeamAnalyzers.Naming;
public sealed class AsyncMethodNamingAnalyzerTests
{
    [Fact]
    public async Task MethodReturningTask_WithoutAsyncSuffix_ReportsDiagnostic()
    {
        var source = """
            using System.Threading.Tasks;
            public class MyClass
            {
                public Task [|DoWork|]() => Task.CompletedTask;
            }
            """;
        await new CSharpAnalyzerTest<AsyncMethodNamingAnalyzer, DefaultVerifier>
        {
            TestCode = source,
        }.RunAsync();
    }
    [Fact]
    public async Task MethodReturningTask_WithAsyncSuffix_NoDiagnostic()
    {
        var source = """
            using System.Threading.Tasks;
            public class MyClass
            {
                public Task DoWorkAsync() => Task.CompletedTask;
            }
            """;
        await new CSharpAnalyzerTest<AsyncMethodNamingAnalyzer, DefaultVerifier>
        {
            TestCode = source,
            ExpectedDiagnostics = { /* none */ }
        }.RunAsync();
    }
    [Fact]
    public async Task CodeFix_AppendsAsyncSuffix_ToMethodAndCallSites()
    {
        var before = """
            using System.Threading.Tasks;
            public class MyClass
            {
                public Task [|DoWork|]() => Task.CompletedTask;
                public async Task Caller() => await DoWork();
            }
            """;
        var after = """
            using System.Threading.Tasks;
            public class MyClass
            {
                public Task DoWorkAsync() => Task.CompletedTask;
                public async Task Caller() => await DoWorkAsync();
            }
            """;
        await new CSharpCodeFixTest<AsyncMethodNamingAnalyzer, AsyncMethodNamingCodeFixProvider, DefaultVerifier>
        {
            TestCode = before,
            FixedCode = after,
        }.RunAsync();
    }
}
```
### Testing TEAM002 -- Null Guard

Testing the null guard rule follows the same positive/negative pattern:

```
using Microsoft.CodeAnalysis.CSharp.Testing;
using Microsoft.CodeAnalysis.Testing;
using TeamAnalyzers.NullGuard;
public sealed class NullGuardAnalyzerTests
{
    [Fact]
    public async Task PublicMethod_NullableParam_MissingGuard_ReportsDiagnostic()
    {
        var source = """
            #nullable enable
            public class OrderService
            {
                public void [|Process|](string? orderId)
                {
                    // No guard
                }
            }
            """;
        await new CSharpAnalyzerTest<NullGuardAnalyzer, DefaultVerifier>
        {
            TestCode = source,
        }.RunAsync();
    }
    [Fact]
    public async Task PublicMethod_NullableParam_WithThrowIfNull_NoDiagnostic()
    {
        var source = """
            #nullable enable
            public class OrderService
            {
                public void Process(string? orderId)
                {
                    ArgumentNullException.ThrowIfNull(orderId);
                    // continue
                }
            }
            """;
        await new CSharpAnalyzerTest<NullGuardAnalyzer, DefaultVerifier>
        {
            TestCode = source,
            ExpectedDiagnostics = { /* none */ }
        }.RunAsync();
    }
}
```
### Testing TEAM003 -- Structured Log Validator

```
using Microsoft.CodeAnalysis.CSharp.Testing;
using Microsoft.CodeAnalysis.Testing;
using TeamAnalyzers.Logging;
public sealed class StructuredLogInterpolationAnalyzerTests
{
    [Fact]
    public async Task LogInformation_WithInterpolation_ReportsError()
    {
        var source = """
            using Microsoft.Extensions.Logging;
            public class OrderController
            {
                private readonly ILogger<OrderController> _logger;
                public OrderController(ILogger<OrderController> logger) =>
                    _logger = logger;
                public void Handle(string orderId)
                {
                    _logger.LogInformation([|$"Processing order {orderId}"|]);
                }
            }
            """;
        await new CSharpAnalyzerTest<StructuredLogInterpolationAnalyzer, DefaultVerifier>
        {
            TestCode = source,
            ReferenceAssemblies = ReferenceAssemblies.Net.Net80
                .AddPackages(ImmutableArray.Create(
                    new PackageIdentity("Microsoft.Extensions.Logging.Abstractions", "8.0.0"))),
        }.RunAsync();
    }
    [Fact]
    public async Task LogInformation_WithMessageTemplate_NoDiagnostic()
    {
        var source = """
            using Microsoft.Extensions.Logging;
            public class OrderController
            {
                private readonly ILogger<OrderController> _logger;
                public OrderController(ILogger<OrderController> logger) =>
                    _logger = logger;
                public void Handle(string orderId)
                {
                    _logger.LogInformation("Processing order {OrderId}", orderId);
                }
            }
            """;
        await new CSharpAnalyzerTest<StructuredLogInterpolationAnalyzer, DefaultVerifier>
        {
            TestCode = source,
            ReferenceAssemblies = ReferenceAssemblies.Net.Net80
                .AddPackages(ImmutableArray.Create(
                    new PackageIdentity("Microsoft.Extensions.Logging.Abstractions", "8.0.0"))),
            ExpectedDiagnostics = { /* none */ }
        }.RunAsync();
    }
}
```
These two tests cover the most important correctness properties of the TEAM003 analyzer. The first test verifies the **positive case**: a `LogInformation` call whose message argument is a `$"..."` interpolated string must produce a TEAM003 diagnostic at the exact location of the interpolated string expression. The `[|...|]` markers tell the testing framework the precise source span that should be flagged -- if the analyzer reports on the wrong token (for example the entire method invocation rather than just the argument), the test fails even though a diagnostic was emitted. Location accuracy is not cosmetic: the IDE uses the reported span to draw the underline and to determine which quick-fix actions to offer.

The second test verifies the **negative case**: the same `LogInformation` call using a proper structured message template with a separately passed argument must produce zero diagnostics. Negative cases are as important as positive cases here. A false positive on a correctly-written log call would cause build failures in already-compliant code, which erodes developer trust in the entire analyzer package. Once developers start suppressing or disabling a noisy rule, the entire value of the analyzer is lost.

Both tests supply a real `Microsoft.Extensions.Logging.Abstractions` package reference via `ReferenceAssemblies`, because `IsLoggerMethod` needs the actual `ILogger` type hierarchy to resolve correctly. Without this reference, the analyzer silently skips the invocation, the positive test produces no diagnostic, and the test fails -- which is exactly the right failure mode to catch a misconfigured test harness.

The `[|...|]` marker syntax tells the testing framework exactly which span should carry the diagnostic. If your analyzer reports on the wrong location, the test will fail even though a diagnostic was emitted -- that's intentional. Location accuracy matters for the IDE underline and the quick-fix trigger.

## Putting These Roslyn Analyzer Examples to Work

These three **roslyn analyzer examples** cover different layers of code quality enforcement. TEAM001 enforces naming consistency at the API surface. TEAM002 enforces reliability contracts at public entry points. TEAM003 enforces correct logging behavior in a way that no code review process can realistically catch at scale.

Each follows the same structural pattern: a `DiagnosticAnalyzer` that uses the most appropriate analysis level (syntax nodes for TEAM001, IOperation blocks for TEAM002 and TEAM003), a focused `DiagnosticDescriptor` with a helpful message, and tests that prove the exact diagnostic span and code fix behavior.

The three analyzers also complement each other architecturally. TEAM001 and TEAM002 together enforce the entry-point contract for every public method: it has the right name and it guards its inputs. TEAM003 enforces the observability contract for every method that produces log output. Together, those two contracts cover the most expensive categories of code review feedback and production incidents in typical .NET backend services. Starting with all three in a single internal package means the rules are adopted together as a coherent standard rather than as a patchwork of individual enforcements added at different times with different conventions.

The logical next step is combining these into a versioned internal NuGet package and wiring it into your CI pipeline so violations block merges. This is how custom roslyn rules go from interesting experiments to genuine quality gates -- and it's within reach of any team that invests an afternoon in the setup.

## Frequently Asked Questions

### What's the difference between a roslyn analyzer and a source generator?

Analyzers inspect existing code and report diagnostics (warnings or errors). Source generators add new code to the compilation without modifying existing files. The three roslyn analyzer examples in this article are all analyzers -- they report problems but never modify source. Source generators run earlier in the pipeline and are suited for boilerplate generation, not rule enforcement.

### Can I use these roslyn analyzer examples without a code fix?

Yes. `CodeFixProvider` is entirely optional. Analyzers report diagnostics independently. A code fix is a convenience that the IDE offers when a diagnostic is present -- but it doesn't affect whether the diagnostic fires. TEAM002 and TEAM003 in this article have no code fix, and they work exactly as intended.

### Why does TEAM003 use DiagnosticSeverity.Error instead of Warning?

In services where structured log queryability matters, string interpolation in log calls undermines the entire observability strategy -- there is no meaningful difference between a broken query and an unqueried log. The Error severity forces the fix rather than letting it accumulate. Teams where structured logging is not yet universal can downgrade to `Warning` via `.editorconfig`.

### How do I suppress a custom roslyn rule for a specific line?

Use `#pragma warning disable TEAM001` before the line and `#pragma warning restore TEAM001` after it, exactly like you would for built-in compiler warnings. You can also apply `[SuppressMessage("Naming", "TEAM001")]` at the method or class level. The `DiagnosticId` in the `DiagnosticDescriptor` is the identifier used in both approaches.

### Will these roslyn analyzer examples slow down my build?

Minimally. All three roslyn analyzer examples use `context.EnableConcurrentExecution()` which lets Roslyn parallelize analyzer execution across files. TEAM001 and TEAM003 use syntax-level analysis, which is extremely fast. TEAM002 uses IOperation but only on public methods with nullable parameters -- the filter eliminates the vast majority of methods. In practice, the overhead across a mid-sized solution is measured in milliseconds.

### How do I version custom roslyn rules to avoid breaking existing code?

Start all new rules at `DiagnosticSeverity.Warning`. Publish the package internally. After the team has had a sprint to address the warnings, promote to `Error` in the next package version. This two-phase rollout prevents the analyzer from blocking CI on first adoption. Changing severity is a one-line change in the `DiagnosticDescriptor` and a version bump.

### Where should I look for more roslyn analyzer examples beyond these three?

The Roslyn Analyzers complete guide maps the full API surface. The .NET Roslyn Analyzers GitHub repository (github.com/dotnet/roslyn-analyzers) contains hundreds of production roslyn analyzer examples from the .NET team itself -- reading those implementations alongside the SDK source code is the fastest way to level up past the patterns shown here.

## Wrapping Up

Complete, production-ready **roslyn analyzer examples** like these eliminate entire categories of review comments, enforce logging contracts that no linter can catch, and harden public APIs at compile time rather than runtime. The three roslyn analyzer examples here -- naming convention enforcement, null guard detection, and structured log validation -- are each directly deployable. Pick the one that addresses your team's most expensive recurring problem and ship it.

From there, the path forward is packaging them into a shared internal NuGet (covered in the distributing your analyzer article), writing a full test suite (see testing your analyzers), and iterating as your team's standards evolve.

*This article was written by Nick Cosentino (Dev Leader). Nick is a Principal Engineering Manager at Microsoft who writes about C#, .NET, software architecture, and engineering leadership. All code examples are written for .NET 10.*
