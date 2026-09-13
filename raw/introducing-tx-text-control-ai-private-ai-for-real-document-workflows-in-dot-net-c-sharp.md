---
url: https://www.textcontrol.com/blog/2026/09/09/introducing-tx-text-control-ai-private-ai-for-real-document-workflows-in-dot-net-c-sharp/
date_fetched: 2026-09-13
---

# Introducing TX Text Control AI: Private AI for Real Document Workflows in .NET C#

Today, we are excited to introduce the TX Text Control AI Preview, a suite of six NuGet packages and ten sample applications that integrate local generative AI, private knowledge retrieval, and TX Text Control document processing.

A useful AI assistant can explain a contract. One that can compare it with your approved policies, suggest a revision, apply an authorized change to the actual document, and return a professionally formatted PDF is even better.

Today, we are introducing the **TX Text Control AI Preview**, a suite of six NuGet packages and ten sample applications that integrate local generative AI, private knowledge retrieval, and TX Text Control document processing.

This is a substantial preview. It includes a complete web application with chat, document studio, runtime administration, and knowledge management, a separately deployable AI service, and a document-focused model context protocol server. You can start on a development workstation and use the same building blocks to move inference to a GPU-powered server within your organization.

The guiding idea is simple: Bring AI to your documents without sending them to a public model service.

The packages are available as version 0.1.0-beta.1 on NuGet, and the sample collection is available on GitHub. This is a preview for evaluation and integration feedback, and the APIs and behavior may change before the stable release. Appropriately licensed TX Text Control products are required.

### What Is TX Text Control AI?

TX Text Control AI is a toolkit that integrates document-aware capabilities into .NET applications. It combines three capabilities that are often implemented separately.

- **Language understanding and generation:**Use an application-owned local model for chat, summaries, structured extraction, rewriting, and planning document actions.
- **Document operations:**Use the TX Text Control engine to create, inspect, modify, format, and export real documents.
- **Private reference knowledge:**Retrieve relevant passages from approved documents and use them as context for an answer or an explicitly authorized edit.

The model handles language. The integration layer manages workflow. The document engine performs document operations. Separating these responsibilities makes it possible to choose a model, replace a user interface, or move inference to a different host without turning the entire application into a tightly coupled system.

The developer-facing AI package works with `Microsoft.Extensions.AI.IChatClient`. The ASP.NET Core package provides reusable HTTP endpoints and browser libraries. Your application retains ownership of its pages, navigation, identity, and business rules.

### Why Use Local Models?

For many business applications, model quality is only one factor in the decision-making process. The location of processing is equally important.

For example, a healthcare organization may need to keep patient-related documents within its own systems. A legal team may handle privileged correspondence and confidential agreements. A manufacturer may want to search service manuals alongside unpublished product specifications. An insurer may need to process claims without involving an external model provider in the process.

Local inference provides these organizations with deployment flexibility; the model can run on an employee's workstation, on a private server, or in an approved, organization-controlled environment. In this context, "local" does not have to mean "on the same computer as the browser." Rather, it means that the application owner controls where inference takes place.

There are several practical benefits:

- **Control over data processing:**Prompts, retrieved passages, and document content can remain within your approved network boundary.
- **Control over versions:**You decide when to change a model, quantization, runtime, or configuration. That helps make application testing repeatable.
- **Control over availability:**Once dependencies, models, and runtimes are provisioned, the inference workflow need not depend on a public model API.
- **A different cost model:**You can allocate your own compute capacity instead of paying a hosted provider for each inference request. Hardware, administration, energy, and maintenance still have costs.
- **A choice of deployment scale:**A developer can begin with a compact model, while a shared service can use more powerful hardware.

Just because something is local does not mean it is secure, compliant, or completely offline. Authentication, authorization, encrypted transport, retention policies, logging, backups, and operating system security are still important. Additionally, the sample uses external resources during setup and loads Google Fonts by default. For an isolated deployment, dependencies must be provisioned through approved channels, required web assets must be self-hosted, and outbound traffic must be controlled.

The benefit is architectural control, not the claim that installing a package establishes regulatory compliance.

### Six Packages With Clearly Separated Responsibilities

The following table summarizes the six NuGet packages and their roles in the local AI ecosystem.

| NuGet package | What it provides | When to use it | 
|---|---|---|
| TXTextControl.AI | The developer-facing model, client, and session APIs, including chat, streaming, and structured responses. | Add local generative AI to a .NET application without adopting a particular UI. | 
| TXTextControl.AI.LlamaServer | Integration with a native llama.cpp server process, runtime management, hardware reporting, and an embedding generator. | Manage the inference runtime or build more specialized model and embedding integrations. | 
| TXTextControl.AI.Mcp | MCP client integration, document-tool discovery and invocation, and artifact download handling. | Connect an AI workflow to the independently hosted document server. | 
| TXTextControl.AI.AspNetCore | AI workflow endpoints, streaming, runtime and Knowledge administration APIs, a remote-service proxy, and reusable client.js and editor.js libraries. | Build a browser application or a separately hosted AI integration service. The pages remain yours. | 
| TXTextControl.AI.Knowledge | Private collections, versioned sources, indexing jobs, SQLite full-text search, and optional semantic or hybrid retrieval. | Ground answers in an approved internal document collection. | 
| TXTextControl.AI.McpServer | A hosting library that exposes TX Text Control document operations through MCP, plus rich-document extraction for Knowledge. | Host the document engine separately from inference. The full MCP sample adds its administration pages and authentication. | 

The AI libraries target .NET 8. The MCP hosting package targets .NET 10. The TX Text Control document engine and Document Editor host run on supported Windows or Linux environments. A separate AI service can run on macOS with a compatible native inference runtime.

NuGet packages do not contain model weights or GPU drivers. The native inference engine is provisioned separately. The sample's default configuration requires either an explicit runtime installation or an existing, compatible executable. Restoring packages or starting the application will not silently download a model.

### What Can You Build?

The preview supports workflows that extend beyond text-only chatbots.

- Generate a proposal, invoice, meeting agenda, report, or letter from a request.
- Summarize an uploaded document or ask questions about its content.
- Rewrite selected text inside the Document Editor.
- Change a named style, including its color, emphasis, and paragraph formatting.
- Convert matching paragraph formatting into reusable document styles.
- Work with tables, fields, templates, and document structure through supported MCP tools.
- Export the current document as PDF or another supported format.
- Compare a working document with approved reference material.
- Extract structured information for a downstream application workflow.

For instance, rather than merely returning instructions describing how a person could make a change, an application can handle a document action such as "Change Heading 1 to red and underline it." Successful edits are applied to the document and reloaded into the editor.

Model capability still matters. Tool calling, structured output, and complex instructions require a compatible model and runtime. Generated content and proposed edits should be reviewable, particularly when factual accuracy or professional judgment is important.

### MCP: Connecting Language to Document Operations

The Model Context Protocol defines how applications connect to servers that expose tools and other capabilities. In this preview, the MCP document server exposes operations implemented by TX Text Control.

It is not another language model. It is the component that knows how to work with document formats, native paragraph styles, tables, fields, and exports.

Consider the request:


`Create a professional invoice with three line items, subtotal, tax, total, and payment terms. Save it as PDF.`

The AI integration provides the model with relevant context and tool definitions. The model can then propose a structured document-tool call. The integration dispatches supported calls to MCP, receives the results, and continues the workflow. MCP then uses the document engine to create and render the file. The web application then displays the result and a download option.

Not every operation requires an additional inference. Once a document exists, a follow-up request such as "PDF please" can export the document directly. The sample offers an export option for generated chat explanations and summaries separately. This distinction prevents an answer about a document from being confused with the document itself.

Long document handling also prevents the entire binary file from being sent through the model. Attachments are transferred through the document workflow, and their Base64 representation is not added to the conversation. The integration considers context and can perform a hierarchical, chunked analysis of extracted text. Large inputs still have configurable limits, and exhaustive reviews require coverage checks.

### Explore the Samples

The GitHub repository contains the source code for the application that uses the NuGet packages. The repository does not require private library sources or a package-building step.


TX Text Control AI SamplesBuild document-aware AI applications with the TX Text Control AI and MCP NuGet packages. This repository contains application source only: the reusable libraries and document engine are consumed as packages. No private library repository, signing key, or package-building step is required.


The ten samples provide different starting points:

- LocalChat, LocalStreaming, LocalStructuredOutput, and LocalToolCalling demonstrate focused console scenarios.
- McpChat connects a console assistant to document tools.
- MinimalWeb demonstrates a small custom web UI using the reusable browser client.
- WebDocumentAssistant contains the complete TXTextControl.AI.Web application.
- AiService hosts inference and Knowledge independently of the website.
- MinimalMcpHost shows minimal document-server hosting.
- McpServer adds server administration, settings, style presets, and diagnostics.

The full web sample is the best way to explore the entire workflow. The source code is a standard application that can be customized. Instead of copying them into every project, reusable editor functions and client communication come from the ASP.NET Core package.

### Getting Started With TXTextControl.AI.Web

The original project name is `TXTextControl.AI.Web.csproj`. In the public repository, it lives in `samples/WebDocumentAssistant`.

- 
##### Prepare a Windows or Linux development machineInstall a stable .NET 10 SDK to build the sample collection, and the .NET 8 and ASP.NET Core 8 runtimes to run the AI applications. Installing both the .NET 8 and .NET 10 SDKs is a convenient development setup. You also need the appropriate TX Text Control licensing, access to the required package feeds, and the native prerequisites for your platform. If the Text Control feed returns an authorization error, configure your authorized feed access; a model setting will not resolve a package restore failure. Clone and build the repository: `git clone https://github.com/TextControl/txtextcontrol-ai-samples.git cd txtextcontrol-ai-samples dotnet restore TXTextControl.AI.Samples.sln dotnet build TXTextControl.AI.Samples.sln -c Release --no-restore`
- 
##### Start the MCP document serverThe MCP host has no built-in administration password. Generate one and store it as a development user secret. This example also works with Windows PowerShell: `$bytes = New-Object byte[] 32 $rng = [Security.Cryptography.RandomNumberGenerator]::Create() try { $rng.GetBytes($bytes) } finally { $rng.Dispose() } $adminPassword = [Convert]::ToBase64String($bytes) dotnet user-secrets set 'Admin:Username' 'admin' --project samples/McpServer dotnet user-secrets set 'Admin:Password' "$adminPassword" --project samples/McpServer # Keep the generated password in your password manager before closing this shell. dotnet run --project samples/McpServer`On Bash, generate a password with openssl rand -hex 32 and set the same user-secret keys. User secrets are a development convenience, not an encrypted production secret store. The local MCP endpoint is `http://127.0.0.1:5000/mcp`. Its administration interface is at`http://127.0.0.1:5000/admin`.
- 
##### Supply a chat model and start the websiteObtain an appropriately licensed, compatible chat/instruction GGUF model and place it in: `samples/WebDocumentAssistant/Models`.Alternatively, configure `LocalAI:ModelDirectory`to point to your model store. Model files are not included in the repository.In another terminal, from the repository root, start the full application: `dotnet run --project samples/WebDocumentAssistant`The website will start and be accessible at `http://127.0.0.1:5187`.
- 
##### Configure the Runtime tabFor the first local session: - Leave Integration service → AI service API URL blank. This runs AI integration in the web host.
- Under Runtime installation, explicitly install a supported backend, or configure an existing LocalAI:LlamaServerExecutablePath.
- Enter `http://127.0.0.1:5000/mcp`as the MCP endpoint and select Test connection.
- Enable TX Text Control Document MCP.
- Select your chat model, not an embedding model.
- Choose a context size appropriate for the available memory and select *Apply and load model.*
 The sample configuration has a context setting of 65,536 tokens. This does not mean that every model or graphics card can allocate this amount. For an initial evaluation of a compact model, start with a smaller context, such as 8,192 tokens, and then increase it after measuring memory and workflow coverage. This is a starting point, not a hardware guarantee. The bundled download catalog is pinned to a supported runtime release identifier, not "whatever is latest." The version and download source are configurable, including approved mirrors and custom asset manifests. A startup compatibility probe does not certify every model/backend combination. When all settings are configured correctly, you should be able to connect to the AI service and start using the integrated features within your document workflow. 
- 
##### Try a complete document workflowCreate an invoice or proposal in *Chat*. Then, follow up with a formatting change and ask for the current document as a PDF.In *Document Studio*, open the included nondisclosure agreement (NDA) or one of the financial, healthcare, or transportation examples. Select text to rewrite, request a summary, or ask for a specific edit. The included documents contain illustrative data only.Use the export controls in the answer when you want the generated summary as a separate document. Use a document export request when you want the current working document. The full web sample guide covers configuration, long documents, remote hosting, and deployment details. The localhost development identity is not a production login system. Use the deployment guidance before sharing the application. 

### Private RAG: Give Answers a Source

A chat model is not inherently familiar with your internal policies, negotiated terms, product manuals, or approved wording. Retrieval-augmented generation (RAG) provides a way to incorporate that material into answers without retraining the model.

The process has two parts:

- **Index references:**Upload approved documents. Extract their text, split it into bounded passages, retain source/version information, and optionally generate embeddings.
- **Retrieve for a question:**Search the authorized collection, select relevant evidence within a context budget, and supply it to the chat model along with the user's request.

Uploading documents does not train the model. The model's weights do not change. References are stored and searched so that relevant passages can be included during inference. Updating a source updates retrievable knowledge without requiring the chat model to be retrained.

The **Knowledge package** supports immediate keyword searches. Semantic search is optional and uses a separate embedding model. Hybrid retrieval combines keyword and semantic rankings. The chat model generates the answer, and the embedding model does not replace it.

#### Start With a Collection, Not an Embedding Configuration

Open *Knowledge* and create a collection, such as "Approved Company Policies," to which you can upload your references. Then, watch the indexing activity, inspect failed jobs, and test retrieval before using the collection in a chat.

Text and Markdown extraction occurs locally on the AI host. DOCX, RTF, PDF, TXT, and HTML extraction are delegated to the configured Windows / Linux MCP document server. This is a dedicated extraction request, not an LLM-driven tool loop for every paragraph. Scanned PDFs require OCR before ingestion, and text extraction should not be confused with understanding page images.

The application supports source replacement and version tracking. After successful indexing, uploading a file with the same display filename replaces that source within the collection. A failed replacement does not silently discard the previously active version.

Select "Reference knowledge" in *Chat* or *Document Studio* when you want to use the collection. Restrict the approved source when the task calls for a particular document. The answer may differ with and without reference knowledge because the model receives different context, not because a different model has been trained.

References are read-only by default. To apply approved wording to a working document, allow the requested changes to the document and issue an edit instruction. Text inside a reference document is still data and does not grant permission to execute tools or override the user's request.

#### Use a Separate Embedding Model for Semantic Retrieval

Configure embeddings in the *Global Knowledge Settings* menu. This setting is independent of the selected collection and applies to all collections on that AI host.

The sample configuration preview includes Qwen3-Embedding-0.6B-Q8_0.gguf for retrieval with 1,024 dimensions and last pooling. Follow the model's exact instructions for query and document prefixes. These choices align with Qwen's embedding model documentation. They are not universal settings for every GGUF.

Save the embedding configuration, then restart the AI and Knowledge process and reindex existing sources. In local mode, restart the web application. In remote mode, restart AiService. The sample uses a separate CPU-based embedding process by default. Therefore, changing embeddings does not mean replacing the chat model with an embedding model.

Source references make answers easier to inspect, but retrieval is not proof of completeness. Every agreement in a collection requires a deliberate, exhaustive workflow rather than assuming that the top few search results represent all documents. When relevant evidence is missing or conflicting, the application should indicate that gap.

#### What Should You Upload? Practical Private-Knowledge Scenarios

When deciding which documents to upload for private knowledge scenarios, focus on those that are approved, authoritative, and relevant to the specific workflow. Having a local knowledge base allows for faster retrieval and better control over sensitive information. It also ensures that the AI system references only trusted sources.

| Scenario | Approved reference documents | Example workflow | 
|---|---|---|
| Healthcare administration | Internal procedures, documentation standards, approved patient-information templates, and organizational policies. | "Which documentation is required before this referral can be processed? Cite the relevant procedure." Keep clinical decisions with qualified professionals. | 
| Legal and contract operations | Clause libraries, negotiation playbooks, approved NDA templates, and authorized agreements. | "Compare this confidentiality clause with our approved wording. Explain deviations without changing the document." | 
| Insurance and financial operations | Policy wording, product documents, processing guides, and internal control procedures. | "Draft a response using the applicable policy and identify information that is still missing." | 
| Manufacturing and field service | Product manuals, maintenance instructions, service bulletins, and approved specifications. | "Find the maintenance procedure for this equipment revision and prepare a service-report draft." | 
| Human resources | Current handbooks, onboarding instructions, expense policies, and approved forms. | "Explain the relevant travel policy and link each requirement to its source." | 
| Transportation and logistics | Operating procedures, customer requirements, route instructions, and incident templates. | "Check this transport plan against the approved procedure and list discrepancies." | 

Begin with a small, curated collection. Review the extracted text, confirm that the source is current, test representative questions, and verify that users can only access the material they are authorized to see. Having more uploaded documents does not automatically mean you have better references.

### Centralize GPU Inference Without Sending Data Outside the Company

Not every workstation needs a GPU. The *AiService sample* independently hosts AI integration, runtime management, embeddings, and knowledge, separate from the website. The website retains the same user-facing pages. The server-side proxy forwards requests to an approved AI service URL via an authenticated connection. Service credentials remain on the server and not in browser JavaScript. MCP remains a separate document service.

In this arrangement:

- The web host owns the UI and Document Editor integration.
- The AI host owns chat model files, the inference runtime, the Knowledge database, source data, and embedding configuration.
- The MCP host owns document processing, working document sessions, rendering, conversion, and rich-document extraction.

First, configure an HTTPS API base, such as `https://ai.example.internal/api/`. Then, create its exact allowlist entry and matching server-side credentials. Then, use the Runtime → Integration service to test and save the approved URL. Restart the website when switching integration modes or URLs. Note that saving a connection does not migrate models, collections, or conversations to the new machine.

The MCP connection has its own endpoint-scoped credentials and trust settings. Protect both hops and do not expose the raw native inference port. See the Remote AI Service Guide for complete configuration instructions.

This architecture is useful when several departments need access to one controlled model deployment and a central team manages GPU capacity, model versions, and operational policy. It is neither an automatic multi-GPU cluster nor an unlimited-concurrency inference service. Plan for capacity, access policies, quotas, and request scheduling based on the actual workload. The preview's sample identity and single-host knowledge storage are starting points, not complete enterprise, multi-tenant deployments.

For Apple Silicon, the repository includes an additional deployment path that runs inference natively on the Mac, while the web application and MCP run in supported Linux containers. The document engine does not need to run natively on macOS.

### Models and Hardware: Choose for the Workflow

Hardware is an important consideration when deploying AI models. The choice of CPU, GPU, and memory resources directly impacts the performance and responsiveness of the system. Ensure that the hardware meets the requirements of the models and the expected workload.

The following table summarizes the roles, model files, and purposes in the sample configuration:

| Role | Model file used in the sample configuration | Purpose | 
|---|---|---|
| Chat and document orchestration | Qwen_Qwen3.5-4B-Q4_K_M.gguf | Language generation and tool-enabled workflows with a compatible runtime and chat template. | 
| Retrieval embeddings | Qwen3-Embedding-0.6B-Q8_0.gguf | Semantic indexing and retrieval, independently of the chat model. | 

The Qwen3.5-4B model card describes the underlying model. A GGUF conversion, quantization, and runtime compatibility evaluation are also necessary. A successful load or a filename is not a benchmark, nor is it proof that every document-tool operation behaves correctly.

#### An RTX 5060-Class Development Machine

The desktop we used for testing has a GeForce RTX 5060 with 8 GB of GDDR7 memory, according to NVIDIA's specifications. This makes a compact quantized model a sensible evaluation target. However, the model file size does not reflect its full runtime memory requirement. The context/KV cache, working buffers, GPU offload, other applications, and concurrent requests also require memory.

For development evaluations, a modern multicore CPU, SSD storage, and 16-32 GB of system RAM provide a practical starting range for small models. Use the higher end of this range when running the editor, MCP, a development environment, and embeddings together. These are planning suggestions, not certified minimum requirements or measured capacity claims. A larger model, longer context, or shared service can require substantially more RAM and VRAM.

Where a compatible runtime is supplied, CPU-only inference is available, although latency must be evaluated for the task. The managed catalog on Windows offers CPU, CUDA, and Vulkan choices. The bundled Linux x64 catalog offers CPU and Vulkan options. Linux CUDA and native macOS options require an explicitly provisioned, compatible engine. Use a validated backend and driver combination with the exact GPU and model rather than assuming that the newest graphics card will work with every runtime archive.

For a central GPU server, size the system according to the model, context length, request concurrency, document analysis workload, and latency objective. Measure time to first token, prompt processing time, generation throughput, end-to-end document latency, memory usage, and tool call correctness separately. A short chat benchmark is not a substitute for testing a long document review or an indexing job.

### Start With One Useful Workflow

The most effective initial evaluation is specific: One model, a small reference collection, a representative document, and a task with an observable outcome.

Create a draft. Then, compare it with an approved source. Then, ask for a controlled edit. Review the result in the editor. Then, export the finished document. Then, decide which parts should be on a workstation and which should be moved to a shared AI service.

TX Text Control AI provides the necessary components for this workflow while giving you control over the application, model, infrastructure, and data boundary.

Explore the samples on GitHub, review the package API guides, and try the complete *TXTextControl.AI.Web* sample. We welcome feedback on model compatibility, document workflows, retrieval quality, deployment, and the developer experience.

### Frequently Asked Questions

TX Text Control AI is a toolkit for building document-aware AI applications in .NET. It combines local generative AI, private knowledge retrieval, and TX Text Control document processing so applications can understand, create, modify, and export real business documents.

Yes. TX Text Control AI is designed to work with application-owned local models that can run on a developer workstation, a private server, or infrastructure controlled by your organization. This allows prompts, document content, and retrieved knowledge to remain within your approved environment.

No. TX Text Control AI is specifically designed for private AI scenarios where document content does not need to be sent to a public model provider. The organization controls where inference takes place and where documents, prompts, and retrieved information are processed.

TX Text Control AI is designed to work with compatible local chat and instruction models and integrates with native llama.cpp-based inference. Developers can choose an appropriate model based on their requirements for quality, performance, hardware, tool calling, and structured output.

You can build applications that summarize documents, answer questions about document content, extract structured information, generate new documents, rewrite selected text, modify formatting and styles, work with tables and fields, compare documents with approved reference material, and export finished documents as PDF or other supported formats.

The language model interprets the user request and can select appropriate document tools. TX Text Control performs the deterministic document operations such as changing text, styles, tables, fields, formatting, and document structure. This separates natural-language reasoning from the document processing engine that performs the actual modification.

The Model Context Protocol (MCP) connects the AI workflow with document-processing tools provided by TX Text Control. The model can discover and invoke supported operations for creating, inspecting, modifying, formatting, and exporting documents without having to understand the underlying document formats itself.

Yes. The TXTextControl.AI.Knowledge package provides private document collections, indexing, full-text search, and optional semantic or hybrid retrieval. Relevant passages from approved internal documents can be supplied to the model as context for answers, analysis, or authorized document changes.

Yes. Large documents do not have to be inserted into the AI conversation as complete binary files. TX Text Control can extract document content and the AI integration can analyze it using configurable, hierarchical, and chunked processing strategies while the original document remains part of the document workflow.

Yes. The architecture allows inference, the web application, private knowledge retrieval, and the TX Text Control MCP document server to be deployed as separate components. This makes it possible to move AI inference to dedicated GPU hardware while hosting document processing and application services independently.

No. The NuGet packages do not contain model weights or GPU drivers, and starting the application does not silently download a model. Developers explicitly provision a compatible inference runtime and an appropriately licensed model, giving organizations control over the models and versions used in production.

The core TX Text Control AI libraries target .NET 8, while the MCP hosting package targets .NET 10. The provided samples demonstrate how the components can be combined in modern .NET and ASP.NET Core applications.

Yes. TX Text Control AI is currently available as a preview. The NuGet packages are published as version 0.1.0-beta.1, and a collection of sample applications is available on GitHub. Because this is a preview release, APIs and behavior may change before the stable release.

Yes. The AI integration packages work together with TX Text Control document processing technology, and appropriately licensed TX Text Control products are required for the corresponding document processing functionality.

## ASP.NET

Integrate document processing into your applications to create documents such as PDFs and MS Word documents, including client-side document editing, viewing, and electronic signatures.

- Angular
- Blazor
- React
- JavaScript
- ASP.NET MVC, ASP.NET Core, and WebForms

### Related Posts

#### Running a Local LLM in .NET C# with TX Text Control AI

Build an interactive .NET C# console chat with TX Text Control AI. Download a GGUF model, stream local replies through IChatClient, and retain conversation history without an OpenAI account or API…

#### AI Natural Language Document Generation with MCP and TX Text Control .NET

This article explains how AI agents can use natural language to create documents through an MCP server. Instead of letting a language model generate documents directly, the AI translates prompts…

#### Introducing Text Control Agent Skills

Text Control Agent Skills are structured definitions that teach AI coding assistants how to build applications with the TX Text Control Document Editor. Each skill contains step-by-step…

#### AI-Ready Legal Documents: What to Fix Before Adding AI

Summerization, analysis, and risk detection: AI can help legal professionals process documents faster and more efficiently. However, before integrating AI into your legal document workflows, it's…

#### Explaining Contract Tracked Changes Automatically Using .NET C# and AI

Learn how to use AI and .NET C# to automatically explain changes to contracts, improving the document review and collaboration processes. This comprehensive guide provides practical implementation…
