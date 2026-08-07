---
url: https://jamiemaguire.net/index.php/2026/08/01/rag-in-net-the-complete-series/
date_fetched: 2026-08-07
---

# RAG in .NET: The Complete Series (index)

Last updated: 28 July 2026 

 Retrieval-Augmented Generation is a pattern, not a framework, and the gap between a RAG tutorial and a RAG pipeline that stays healthy in production is substantial.

 This series covers what I’ve actually learned building and operating RAG pipelines in .NET, mostly for documentation sites and client engagements: chunking decisions, retrieval quality, observability, and the tooling needed to keep a pipeline healthy once it’s shipped.

 This isn’t a single getting-started guide, it’s a working series that grows as I find new gotchas and build better tooling.

 You can start wherever’s relevant to you.

 The Series, In Order

 Microsoft Agent Framework: Adding RAG to Your AI Agent Using TextSearchProvider and In-Memory Vector Store Grounding an Agent Framework agent’s responses in your own documents using TextSearchProvider . (Also part of the Microsoft Agent Framework series .) 

 RAG in .NET: What the Tutorials Don’t Tell You. Chunking, Embedding, and Production Gotchas The chunking decisions that degrade silently, retrieval quality that looks fine in demos and falls apart on real queries, and the observability gaps that make debugging feel like guessing.

 Your RAG Pipeline Is Slow Because You’re Sending Entire Pages to the LLM Diagnosing an 18–25 second query latency down to what was actually being sent to the LLM for synthesis, and fixing it.

 The RAG Workbench I Actually Needed Building DocIngestion, a local-first RAG ingestion and retrieval inspection tool, after concluding most RAG tooling makes answering basic “why did this happen” questions too slow.

 Building a RAG Administration Tool with .NET, Elasticsearch and OpenAI Keeping a pipeline healthy after the initial vectorisation pass, a localhost admin panel for managing document and vector index drift.

 A Developer’s Guide to Building RAG Systems: Lessons from the Trenches Lessons pulled from building RAG pipelines across a number of engagements, where the real cost of early decisions shows up later, not at the start.

 ~

 Earlier Related Reading

 Before this series, I wrote about implementing 100% local RAG using Phi-3 with local embeddings via Semantic Kernel. The model choice has moved on since, but it’s still a useful reference if you need a RAG pipeline that runs fully offline, with no calls out to a cloud model.

 ~

 Building an AI Agent More broadly?

 If you landed here via the RAG-in-Agent-Framework post and want the full picture, function tools, memory, human-in-the-loop, MCP, and more, see the Microsoft Agent Framework series .

 Questions about any post in this series, or want to see a topic covered? Drop a note in the comments, or schedule a call to discuss consulting and development services. 

 ~


======================================================================
# Part 1: Microsoft Agent Framework: Adding RAG to Your AI Agent Using TextSearchProvider and In-Memory Vector Store
URL: https://jamiemaguire.net/index.php/2026/02/21/microsoft-agent-framework-adding-rag-to-your-ai-agent-using-textsearchprovider-and-in-memory-vector-store/
======================================================================

If you’ve been following this series, you’ll know we’ve been incrementally building out the Iron Mind AI personal trainer agent.

 We’ve added function tools, agents as function tools, human-in-the-loop checkpoints, contextual memory, background responses, MCP tool support, and email marketing capabilities.

 In this post, we take a different direction and look at how to give your AI agent access to your own documents using Retrieval Augmented Generation (RAG).

 Instead of relying solely on the LLM’s training data, the agent searches a vector store before each response and grounds its answers in your content.

 ~

 Why RAG Matters for AI Agents

 LLMs are powerful but they have a fundamental limitation, they only know what they were trained on.

 They can’t access your internal documentation, product guides, company policies, or domain-specific knowledge.

 RAG solves this by retrieving relevant documents at runtime and injecting them into the model’s context. The result is an agent that answers questions using your data, not generic knowledge.

 This is useful when you want your agent to:

 Answer questions grounded in specific documentation

 Cite sources so users can verify information

 Avoid hallucination by constraining responses to known content

 RAG also helps your agent stay current without retraining the underlying model

 ~

 How It Works

 The Microsoft Agent Framework provides a TextSearchProvider that hooks into the agent’s execution pipeline. Before each model invocation, it runs a vector search against your document store and injects the matching results into the conversation as additional context messages.

 The flow works like this:

 User asks a question

 TextSearchProvider converts the question into an embedding

 The embedding is compared against document embeddings in the vector store

 The top matching documents are injected into the model context

 The LLM generates a response grounded in the retrieved documents

 For this example, we use the InMemoryVectorStore from the Microsoft.SemanticKernel.Connectors.InMemory NuGet package.

 We’re using this because at the time of writing, the Microsoft Agent Framework doesn’t ship with its own vector store connector.

 Instead, the Microsoft Agent Framework delegates storage to the Microsoft.Extensions.VectorData abstractions.

 These define the interfaces but contain no implementations.  The Semantic Kernel connector packages ship with standard implementations for this interface.

 Note: We’re not using the Semantic Kernel framework itself, just this one connector package for its in-memory vector store implementation.

 In a production scenario, you could swap this out for any vector store that implements these same abstractions, such as Azure AI Search, Qdrant, or Pinecone.

 ~

 Project Structure

 The project follows the same structure we’ve used throughout this series, with an Agents/ folder to keep agent logic separate from the entry point:

 ├── Agent-Framework-9-Basic-RAG.csproj
├── Program.cs
├── Agents/
│   └── IronMindRagAgent.cs
├── TextSearchStore.cs
└── TextSearchDocument.cs 

 Program.cs  is lean and handles only the OpenAI client setup and the chat loop

 IronMindRagAgent.cs  encapsulates the vector store setup, search logic, and agent configuration

 TextSearchStore.cs  wraps the in-memory vector store with a clean API for upserting documents and running searches

 TextSearchDocument.cs  defines the document model

 With an overview of the project structure, lets look at the project definition.

 ~

 Setting Up the Project

 The project targets .NET 8.0 and uses the following NuGet packages:

 <Project Sdk="Microsoft.NET.Sdk">
 <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <RootNamespace>Agent_Framework_9_Basic_RAG</RootNamespace>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
 </PropertyGroup>

 <ItemGroup>
   <PackageReference Include="Microsoft.Agents.AI" Version="1.0.0-preview.260205.1" />
    <PackageReference Include="Microsoft.Agents.AI.OpenAI" Version="1.0.0-preview.260205.1" />
    <PackageReference Include="Microsoft.Extensions.AI.OpenAI" Version="10.2.0-preview.1.26063.2" />
    <PackageReference Include="Microsoft.Extensions.VectorData.Abstractions" Version="9.7.0" />
    <PackageReference Include="Microsoft.SemanticKernel.Connectors.InMemory" Version="1.66.0-preview" />
    <PackageReference Include="OpenAI" Version="2.8.0" />
    <PackageReference Include="System.ClientModel" Version="1.8.1" />
 </ItemGroup>
</Project>

 Some key packages to note include:

 Microsoft.Agents.AI  provides the TextSearchProvider, AIAgent, and ChatClientAgentOptions

 Microsoft.Agents.AI.OpenAI  provides the .AsAIAgent() extension method for OpenAI’s ChatClient

 Microsoft.Extensions.AI.OpenAI  provides .AsIEmbeddingGenerator() to convert OpenAI’s embedding client into the standard IEmbeddingGenerator interface

 Microsoft.Extensions.VectorData.Abstractions  defines the vector store abstractions (VectorStoreCollection, etc.)

 Microsoft.SemanticKernel.Connectors.InMemory  provides the InMemoryVectorStore implementation

 Now let’s dig into the required models.

 ~

 Defining the Document Model

 First, we need a simple model to represent the documents we want to store and search:

 public class TextSearchDocument
{
    public string SourceId { get; set; } = string.Empty;
    public string SourceName { get; set; } = string.Empty;
    public string SourceLink { get; set; } = string.Empty;
    public string Text { get; set; } = string.Empty;
} 

 Each document has an identifier, a human-readable source name, a link for citation purposes, and the actual text content.

 ~

 Building the TextSearchStore

 The TextSearchStore wraps the in-memory vector store and provides a clean API for upserting documents and running searches:

 using Microsoft.Extensions.VectorData;
using Microsoft.SemanticKernel.Connectors.InMemory;
namespace Agent_Framework_9_Basic_RAG;

public class TextSearchStore
{
    private readonly VectorStoreCollection<string, TextSearchRecord> _collection;

    public TextSearchStore(InMemoryVectorStore vectorStore, string collectionName, int dimensions)
    {
        var definition = new VectorStoreCollectionDefinition
        {
            Properties =
            [
                new VectorStoreKeyProperty("SourceId", typeof(string)),
                new VectorStoreDataProperty("SourceName", typeof(string)),
                new VectorStoreDataProperty("SourceLink", typeof(string)),
                new VectorStoreDataProperty("Text", typeof(string)),
                new VectorStoreVectorProperty("Embedding", typeof(string), dimensions),
            ]
        };
        _collection = vectorStore.GetCollection<string, TextSearchRecord>(collectionName, definition);
    }

    public async Task UpsertDocumentsAsync(IEnumerable<TextSearchDocument> documents)
    {
        await _collection.EnsureCollectionExistsAsync();

        foreach (var doc in documents)
        {
            var record = new TextSearchRecord
            {
                SourceId = doc.SourceId,
                SourceName = doc.SourceName,
                SourceLink = doc.SourceLink,
                Text = doc.Text,
                Embedding = doc.Text
            };
            await _collection.UpsertAsync(record);
        }
    }

    public async Task<IEnumerable<TextSearchDocument>> SearchAsync(
        string query, int topK, CancellationToken cancellationToken = default)
    {
        var results = _collection.SearchAsync(query, topK, cancellationToken: cancellationToken);

        var documents = new List<TextSearchDocument>();
        await foreach (var result in results)
        {
            documents.Add(new TextSearchDocument
            {
                SourceId = result.Record.SourceId,
                SourceName = result.Record.SourceName,
                SourceLink = result.Record.SourceLink,
                Text = result.Record.Text
            });
        }

        return documents;
    }
} 

 A few important things to note here:

 The VectorStoreCollectionDefinition defines the schema for the collection, including the vector property with its dimensions (3072 for text-embedding-3-large)

 The Embedding property is typed as string, not ReadOnlyMemory<float>. This is important. When you set the Embedding to the document text and the vector store has an EmbeddingGenerator configured, it automatically converts the text to a vector embedding during upsert. The same happens during search when you pass a string query. If you use ReadOnlyMemory<float> instead, the embeddings won’t be generated and you’ll get a dimension mismatch error at search time

 The TextSearchRecord is an internal model that includes the Embedding property, while TextSearchDocument is the public-facing model

 The internal record type:

 public class TextSearchRecord
{
    public string SourceId { get; set; } = string.Empty;
    public string SourceName { get; set; } = string.Empty;
    public string SourceLink { get; set; } = string.Empty;
    public string Text { get; set; } = string.Empty;
    public string Embedding { get; set; } = string.Empty;
} 
 This is used to represent vectorised content.

 ~

 Creating the Sample Documents

 For this example, we use three Iron Mind AI personal trainer documents covering beginner training, nutrition, and recovery. These are defined as a static method on TextSearchStore :

 public static IEnumerable<TextSearchDocument> GetSampleDocuments()
{
    yield return new TextSearchDocument
    {
        SourceId = "beginner-strength-001",
        SourceName = "Iron Mind AI - Beginner Strength Training Guide",
        SourceLink = "https://ironmindai.com/tips/beginner-strength",
        Text = "For beginners, focus on compound movements like squats, deadlifts, bench press, " +
               "and overhead press. Train 3-4 days per week with at least one rest day between " +
               "sessions. Start with a weight you can control for 8-12 reps with good form. " +
               "Progressive overload is key - aim to gradually increase weight, reps, or sets " +
               "over time. Consistency beats intensity in the early stages."
    };
    yield return new TextSearchDocument
    {
        SourceId = "nutrition-basics-001",
        SourceName = "Iron Mind AI - Nutrition for Muscle Growth",
        SourceLink = "https://ironmindai.com/tips/nutrition-muscle-growth",
        Text = "To support muscle growth, aim for 1.6 to 2.2 grams of protein per kilogram of " +
               "body weight per day. Spread protein intake across 3-5 meals for optimal muscle " +
               "protein synthesis. Prioritize whole food sources like chicken, fish, eggs, Greek " +
               "yogurt, and legumes. Don't neglect carbohydrates - they fuel your workouts and " +
               "aid recovery. A slight caloric surplus of 200-300 calories above maintenance is " +
               "ideal for lean muscle gain."
    };
    yield return new TextSearchDocument
    {
        SourceId = "recovery-sleep-001",
        SourceName = "Iron Mind AI - Recovery and Sleep Guide",
        SourceLink = "https://ironmindai.com/tips/recovery-sleep",
        Text = "Sleep is when your body repairs and builds muscle tissue. Aim for 7-9 hours of " +
               "quality sleep per night. Poor sleep increases cortisol levels, which can impair " +
               "muscle recovery and promote fat storage. Establish a consistent sleep schedule, " +
               "limit screen time before bed, and keep your room cool and dark. Active recovery " +
               "on rest days - such as walking, stretching, or light yoga - also helps reduce " +
               "soreness and improve circulation."
    };
} 

 In a real-world scenario, these documents would come from a database, CMS, file system, or an external API.

 ~

 The IronMindRagAgent

 This is where the RAG logic lives. The IronMindRagAgent class encapsulates the vector store setup, the search adapter, and the agent configuration.

 This keeps Program.cs clean and follows the same Agents/ folder convention used across the other projects in this series.

 using Microsoft.Agents.AI;
using Microsoft.Extensions.AI;
using Microsoft.SemanticKernel.Connectors.InMemory;
using OpenAI;
using OpenAI.Chat;

namespace Agent_Framework_9_Basic_RAG.Agents;

public class IronMindRagAgent
{
    private const int EmbeddingDimensions = 3072;
    private const string CollectionName = "iron-mind-ai-tips";

    private readonly TextSearchStore _textSearchStore;

    public IronMindRagAgent(OpenAIClient openAIClient, string embeddingModel)
    {
        var vectorStore = new InMemoryVectorStore(new()
        {
            EmbeddingGenerator = openAIClient.GetEmbeddingClient(embeddingModel).AsIEmbeddingGenerator()
        });

        _textSearchStore = new TextSearchStore(vectorStore, CollectionName, EmbeddingDimensions);
    }

    public async Task<AIAgent> CreateAgentAsync(OpenAIClient openAIClient, string model)
    {
        await _textSearchStore.UpsertDocumentsAsync(TextSearchStore.GetSampleDocuments());

        var textSearchOptions = new TextSearchProviderOptions
        {
            SearchTime = TextSearchProviderOptions.TextSearchBehavior.BeforeAIInvoke,
            CitationsPrompt = "Always cite sources at the end of your response using the format: " +
                              "**Source:** [SourceName](SourceLink)",
        };

        return openAIClient
            .GetChatClient(model)
            .AsAIAgent(new ChatClientAgentOptions
            {
                ChatOptions = new()
                {
                    Instructions = "You are Iron Mind AI, a knowledgeable personal trainer. " +
                                   "You MUST base your answers on the provided context documents. " +
                                   "Always cite your sources by name and link at the end of your response. " +
                                   "If the context does not contain relevant information, say so."
                },

                AIContextProviderFactory = (ctx, ct) => new ValueTask<AIContextProvider>(
                    new TextSearchProvider(SearchAsync, ctx.SerializedState,
                                          ctx.JsonSerializerOptions, textSearchOptions)),
                ChatHistoryProviderFactory = (ctx, ct) => new ValueTask<ChatHistoryProvider>(
                    new InMemoryChatHistoryProvider().WithAIContextProviderMessageRemoval()),
            });
    }

    private async Task<IEnumerable<TextSearchProvider.TextSearchResult>> SearchAsync(
        string text, CancellationToken ct)
    {
        var searchResults = await _textSearchStore.SearchAsync(text, 2, ct);
        return searchResults.Select(r => new TextSearchProvider.TextSearchResult
        {
            SourceName = r.SourceName,
            SourceLink = r.SourceLink,
            Text = r.Text,
            RawRepresentation = r
        });

    }
} 

 Let’s walk through the key parts.

 ~

 The Constructor

 The constructor takes the OpenAIClient and the embedding model name. It creates the in-memory vector store with the OpenAI embedding generator and initialises the TextSearchStore :

 public IronMindRagAgent(OpenAIClient openAIClient, string embeddingModel)
{
    var vectorStore = new InMemoryVectorStore(new()
    {
        EmbeddingGenerator = openAIClient.GetEmbeddingClient(embeddingModel).AsIEmbeddingGenerator()
    });

    _textSearchStore = new TextSearchStore(vectorStore, CollectionName, EmbeddingDimensions);
} 

 The .AsIEmbeddingGenerator() extension converts OpenAI’s embedding client into the standard IEmbeddingGenerator interface from Microsoft.Extensions.AI .

 This embedding generator is used automatically during both upsert (to convert document text into vectors) and search (to convert the query into a vector).

 ~

 CreateAgentAsync

 This method loads the sample documents into the vector store, configures the TextSearchProvider , and builds the agent:

 public async Task<AIAgent> CreateAgentAsync(OpenAIClient openAIClient, string model)
{
    await _textSearchStore.UpsertDocumentsAsync(TextSearchStore.GetSampleDocuments());

    var textSearchOptions = new TextSearchProviderOptions
    {
        SearchTime = TextSearchProviderOptions.TextSearchBehavior.BeforeAIInvoke,
        CitationsPrompt = "Always cite sources at the end of your response using the format: " +
                          "**Source:** [SourceName](SourceLink)",
    };

    return openAIClient
        .GetChatClient(model)
        .AsAIAgent(new ChatClientAgentOptions { ... });
} 

 Two options control the RAG behaviour:

 SearchTime = BeforeAIInvoke tells the provider to automatically run a search before every model invocation and inject the results as context messages

 CitationsPrompt  provides explicit instructions to the model on how to format source citations

 An alternative to BeforeAIInvoke is OnDemandFunctionCalling , which exposes the search as a function tool that the model can choose to invoke when it decides it needs more information.

 ~

 The Agent Configuration

 The AIContextProviderFactory creates a new TextSearchProvider instance for each session. The SearchAsync method is passed as a method group reference rather than an inline delegate, keeping the code clean and readable.

 The ctx.SerializedState and ctx.JsonSerializerOptions parameters support session serialisation, allowing the provider’s state to be persisted and restored.

 The ChatHistoryProviderFactory creates an InMemoryChatHistoryProvider with .WithAIContextProviderMessageRemoval() .

 This is important because without it, every search result message would accumulate in the chat history, bloating the context window over a multi-turn conversation.

 Notice the system instructions explicitly tell the model to base answers on the provided context and cite sources.

 Without strong instructions, the model may fall back to its general training data and ignore the injected documents.

 ~

 The Search Adapter

 The TextSearchProvider doesn’t talk to the vector store directly. Instead, it calls the SearchAsync method that you provide.

 This gives you full control over how searches are executed:

 private async Task<IEnumerable<TextSearchProvider.TextSearchResult>> SearchAsync( string text, CancellationToken ct)
{
    var searchResults = await _textSearchStore.SearchAsync(text, 2, ct);
    return searchResults.Select(r => new TextSearchProvider.TextSearchResult
    {
        SourceName = r.SourceName,
        SourceLink = r.SourceLink,
        Text = r.Text,
        RawRepresentation = r
    });
} 

 We return the top 2 results per query. You can adjust this based on how much context you want to provide to the model.

 ~

 Program.cs

 With all the RAG logic encapsulated in the agent class, Program.cs is minimal:

 using Agent_Framework_9_Basic_RAG.Agents;
using OpenAI;

string apiKey = "your-openai-api-key";
string model = "gpt-4o-mini";
string embeddingModel = "text-embedding-3-large";

OpenAIClient openAIClient = new(apiKey);

var ironMind = new IronMindRagAgent(openAIClient, embeddingModel);
var agent = await ironMind.CreateAgentAsync(openAIClient, model);
var session = await agent.CreateSessionAsync();

Console.WriteLine("Iron Mind AI - Personal Trainer");
Console.WriteLine("Ask me anything about training, nutrition, or recovery. Type 'exit' to quit.\n");

while (true)
{
    Console.Write("You: ");
    string? input = Console.ReadLine();
    if (string.IsNullOrWhiteSpace(input) || input.Equals("exit", StringComparison.OrdinalIgnoreCase))
        break;

    Console.WriteLine();
    Console.WriteLine(await agent.RunAsync(input, session));
    Console.WriteLine();
} 

 Three lines to set up the agent. The rest is the chat loop. All vector store configuration, document loading, search logic, and prompt engineering lives inside IronMindRagAgent.

 ~

 Example Output

 Here’s what a conversation with the RAG-enabled agent looks like in action:

 An example from the console output:

 Iron Mind AI - Personal Trainer

Ask me anything about training, nutrition, or recovery. Type 'exit' to quit.

You: as a beginner, what can i do?

As a beginner in strength training, here are some key steps you can follow:

**Focus on Compound Movements**: Start with exercises like squats, deadlifts,

   bench presses, and overhead presses. These movements engage multiple muscle groups,

   which is great for building strength effectively.

**Training Frequency**: Aim to train 3-4 days per week. Ensure you have at least

   one rest day between sessions to allow your muscles to recover.

**Start with Manageable Weights**: Choose a weight that you can control for 8-12

   repetitions while maintaining good form.

**Progressive Overload**: Gradually increase the weight, number of repetitions,

   or sets over time.

**Consistency Over Intensity**: In the beginning, it's better to focus on being

   consistent with your workouts rather than pushing yourself too hard.

**Prioritize Recovery and Sleep**: Aim for 7-9 hours of quality sleep per night,

   as this is when your body repairs and builds muscle tissue.

**Source:** [Iron Mind AI - Beginner Strength Training Guide](https://ironmindai.com/tips/beginner-strength),

[Iron Mind AI - Recovery and Sleep Guide](https://ironmindai.com/tips/recovery-sleep) 

 The agent pulls from the relevant documents, synthesises a coherent answer, and cites the sources at the end. This is the key benefit of RAG: the response is grounded in your content, not hallucinated from general training data.

 ~

 Demo

 In the following demo, we can see the “RAG-fied” Iron Mind AI personal trainer agent in action:

 ~

 Wrapping Up

 In this post we looked at how to add RAG capabilities to a Microsoft Agent Framework agent using TextSearchProvider and an in-memory vector store. The key takeaways:

 TextSearchProvider  integrates directly into the agent pipeline, running searches before each model invocation

 In-Memory Vector Store with an EmbeddingGenerator handles embedding generation automatically during both upsert and search

 The vector property should be typed as  string  (not ReadOnlyMemory<float>) to enable automatic embedding generation

 Strong system instructions  are essential to ensure the model uses the provided context rather than falling back to general knowledge

 CitationsPrompt on TextSearchProviderOptions tells the model how to format source citations

 WithAIContextProviderMessageRemoval()  prevents search result messages from bloating the chat history

 Encapsulating the RAG logic in a dedicated agent class keeps Program.cs lean and follows good separation of concerns

 The in-memory vector store used here is great for prototyping and demos.

 For production workloads, you would swap it for a persistent vector store such as Azure AI Search, Qdrant, Weaviate, or Pinecone.

 The Microsoft.Extensions.VectorData abstractions make this a straightforward change.

 If you have any questions, feel free to reach out.

 ~

 Enjoy what you’ve read, have questions about this content, or would like to see another topic?

 Drop me a note below.

 You can schedule a call using my  Calendly link  to discuss consulting and development services.


======================================================================
# Part 2: RAG in .NET: What the Tutorials Don’t Tell You.  Chunking, Embedding, and Production Gotchas
URL: https://jamiemaguire.net/index.php/2026/04/18/rag-in-dotnet-semantic-kernel-production-guide/
======================================================================

Most RAG tutorials show you the happy path. A clean project, a handful of sample documents, a query that works first time.

 That’s fine for getting oriented but it’s not what building these systems in production actually looks like.

 This post is the one I wished existed when I started working in earlier RAG solutions using Semantic Kernel.

 The chunking decisions that degrade silently, the retrieval quality that looks fine in demos and falls apart on real queries, and observability gaps that make debugging feel like guessing.

 All in .NET.

 This blog is by know means exhaustive and I continue to find optimisations but it’s a good starting point.

 Lets dig in.

 ~

 What RAG Is

 RAG (Retrieval-Augmented Generation) is a pattern, not a framework. Before asking an LLM to answer a question, you first retrieve relevant content from your own data store and include it in the prompt.

 The model answers from that grounded context rather than from training data alone.

 A RAG pipeline has five stages:

 Ingestion — load source documents (HTML, Markdown, PDFs, plain text)

 Chunking — split documents into segments small enough to embed meaningfully

 Embedding — convert each chunk into a vector using an embedding model

 Storage — persist vectors to a vector store (SQLite, Elasticsearch, Azure AI Search, etc.)

 Retrieval + Generation — embed the incoming query, find the closest chunks, build a grounded prompt, generate an answer

 Simple on paper. The devil is in the implementation choices at each stage.

 ~

 Chunking: The Step Most Tutorials Rush

 Chunking quality has a disproportionate effect on retrieval quality. Chunks too large: vector similarity becomes diluted. Too small: you lose the context that makes a chunk meaningful.

 One approach is to use TextChunker.SplitMarkdownParagraphs() from Semantic Kernel. It respects document structure such as  paragraphs, headings, and list items don’t get bisected mid-sentence.

 var chunks = TextChunker.SplitMarkdownParagraphs(
 lines: markdownContent.Split('n').ToList(),
 maxTokensPerParagraph: 512,
 overlapTokens: 50
); 

 The overlapTokens parameter matters. A small overlap (10%-15%) between adjacent chunks ensures that a relevant sentence near a chunk boundary doesn’t disappear from retrieval. Skipping this is a common mistake.

 I implemented my own custom chunking service on one project.

 Gotcha: HTML content

 Convert HTML to Markdown before chunking. Raw HTML bloats chunks with noise such as tags, attributes, and inline styles .  These degrade embedding quality. Use  the HtmlAgilityPack to strip structure first.

 Gotcha: Mixed content types

 A chunk that mixes a code sample with surrounding prose often embeds poorly because the two content types pull the vector in different directions. Chunk code blocks separately and tag them with metadata for filtering at retrieval time.  This was an an important learning for me.

 ~

 The Relevance Threshold Is Not a Magic Number

 Semantic Kernel’s SearchAsync takes a minRelevanceScore parameter. Tutorial defaults (0.75–0.80) are not universally correct.  The right threshold depends on your corpus and embedding model.

 var results = await memory.SearchAsync(
 collection: CollectionName,
 query: userQuery,
 limit: 5,
 minRelevanceScore: 0.70
); 
 Start at 0.70 (or whatever your comfort level is) and run representative queries, and look at what gets returned.

 Build a manual eval set of 20–30 query/expected-answer pairs and iterate. There is no substitute for looking at actual retrieval results on your specific data.

 ~

 Choosing a Vector Store

 Match the tool to the stage:

 VolatileMemoryStore — Demos only. Vectors live in RAM, gone on restart.

 SqliteMemoryStore — Local development and early production. File-based, zero infrastructure overhead.

 Elasticsearch — Already in your stack? Use it. Good for hybrid search.

 Azure AI Search — Production on Azure. Managed, scalable.

 Qdrant / Pinecone — Dedicated vector workloads at scale.

 SQLite is underrated for early production. It’s a one-line swap from VolatileMemoryStore and handles modest query volumes without infrastructure cost. Migrate later when you actually need to.

 ~

 The One-Time Embedding Check

 Once you’re using a persistent store, add a collection existence check before the ingestion loop. Without it, every restart re-embeds the entire corpus — API calls and cost you don’t need.

 var collections = await sqliteStore.GetCollectionsAsync().ToListAsync();
if (!collections.Contains(CollectionName))
{
 await ragService.IngestDocumentsAsync(documents, CollectionName);
}
else
{
 Console.WriteLine("Vectors already stored - skipping ingestion.");
} 

 Small investment. Saves meaningful API cost at scale.

 ~

 Prompt Construction: Ground It Properly

 The difference between a useful RAG system and a hallucinating one often comes down to prompt construction.

 A simple prompt you can use:

 var sb = new StringBuilder();
sb.AppendLine("Answer the question using ONLY the context below.");
sb.AppendLine("If the answer is not in the context, say so explicitly.");
sb.AppendLine();
sb.AppendLine("CONTEXT:");
foreach (var chunk in retrievedChunks)
{
 sb.AppendLine($"[Source: {chunk.Metadata.Id}]");
 sb.AppendLine(chunk.Metadata.Text);
 sb.AppendLine();
}
sb.AppendLine($"QUESTION: {userQuery}"); 
 The key phrases are “ONLY the context below” and “say so explicitly”. Without explicit grounding instructions, models blend retrieved content with training knowledge which looks helpful but introduces unfaithful answers.

 This isn’t optional.

 ~

 Semantic Caching: The Easy Win Most People Skip

 For user-facing or high-volume pipelines, add semantic caching early. Before hitting the vector store and LLM, check whether an incoming query is semantically similar to a recent query already answered.

 If the similarity score is above threshold return the cached answer directly.

 var cachedAnswer = await cacheService.FindSimilarAsync(query, threshold: 0.92f);
if (cachedAnswer != null)
{
 return cachedAnswer.Answer; // No vector search, no LLM call
} 

 At scale this eliminates a large proportion of pipeline calls and cuts latency dramatically for common query patterns.  Add this early.  Retrofitting it later is more work than it needs to be.

 ~

 Observability: Knowing What’s Actually Happening

 A RAG pipeline has multiple failure modes and they all look the same from the outside: a bad answer. Without instrumentation you can’t tell whether the problem is in chunking, retrieval, prompt construction, or the model itself.

 Consider capturing data using a logging record similar to:

 public record RagQueryTrace
{
 public string Query { get; init; }
 public int ChunksRetrieved { get; init; }
 public float TopChunkScore { get; init; }
 public float LowestChunkScore { get; init; }
 public string[] SourceIds { get; init; }
 public string GeneratedAnswer { get; init; }
 public double LatencyMs { get; init; }
 public bool CacheHit { get; init; }
} 
 Signals to watch:

 TopChunkScore consistently below 0.75 : retrieval is struggling.

 ChunksRetrieved always hitting your limit : try widening search and re-ranking.

 CacheHit always false with high latency : cache threshold may be too tight.

 Wire up end-to-end tracing with ILogger :

 public async Task<string> QueryAsync(string query, string collection)
{
 var sw = Stopwatch.StartNew();
 _logger.LogInformation("RAG query started. Query={Query}", query);

 var chunks = await RetrieveChunksAsync(query, collection);
 _logger.LogInformation("Retrieval complete. Chunks={Count}, TopScore={Score:F3}",
 chunks.Count, chunks.FirstOrDefault()?.Relevance ?? 0);

 var answer = await GenerateAnswerAsync(query, chunks);
 _logger.LogInformation("Generation complete. LatencyMs={Ms}", sw.ElapsedMilliseconds);

 return answer;
} 

 Diagnosing bad answers:

 Right chunks not retrieved? – Retrieval problem (threshold, chunking, embedding model)

 Chunks retrieved but answer wrong? – Tighten grounding instructions in the prompt

 Chunks and prompt correct but hallucinated? – Add explicit “do not speculate” to system prompt

 Work backwards through the trace when you experience any of the above.

 ~

 Evaluating RAG Quality and Why CI Matters

 Most RAG prototypes get evaluated informally. This works until the corpus changes, a threshold gets tweaked, or the embedding model is swapped. Quality silently regresses with no way to detect it.

 Build question/answer pairs covering easy queries, hard queries spanning multiple documents, and edge cases where the answer isn’t in the corpus and the system should say “I don’t know”. Three metrics worth tracking include:

 Context Recall : were the right chunks retrieved?

 Faithfulness : does the answer stick to the retrieved context?

 Answer Correctness : does the answer match the expected answer?

 Wire evals into your CI process.  For example:

 [Fact]
public async Task RagEval_ContextRecall_AboveThreshold()
{
 var results = await RunEvalSetAsync(_evalQueries);
 var avgRecall = results.Average(r => r.ContextRecall);
 Assert.True(avgRecall >= 0.80,
 $"Context recall {avgRecall:P0} is below the 80% threshold");
} 
 The edge case eval is the most important, i,e. – queries where the answer genuinely isn’t in the corpus.

 These test whether the system correctly says “I don’t know” rather than hallucinating.

 Hallucination on out-of-scope queries is the thing that erodes user trust fastest and it’s the thing informal testing almost never catches.

 ~

 What to Watch Out For

 Some other things to watch out for:

 Skipping overlap tokens in chunking — sentences near chunk boundaries silently drop out of retrieval. Always set overlapTokens .

 Using tutorial threshold values verbatim — 0.75 or 0.80 is a starting point, not a universal answer. Tune against your actual corpus.

 Re-embedding on every restart — add the collection existence check. It’s five lines and saves real API cost.

 Weak grounding instructions — “use the context below” is not the same as “ONLY the context below”. The difference shows up in production.

 No out-of-scope eval set — hallucination on questions the system can’t answer is the fastest way to lose user trust. Test for it explicitly.

 Another reminded – make sure you chunk prose and code examples differently.  This really caught me out.

 ~

 Tools Used

 Key tools involved in the above included:

 Semantic Kernel (chunking, embedding, retrieval)

 TextChunker.SplitMarkdownParagraphs (structure-aware chunking)

 HtmlAgilityPack (HTML-to-Markdown conversion)

 SqliteMemoryStore / VolatileMemoryStore / Azure AI Search (vector stores)

 ILogger / Stopwatch (observability and tracing)

 xUnit (eval set CI integration)

 .NET 8

 The happy path is easy to build. A RAG system that stays reliable as the corpus grows, thresholds get tuned, and real users ask unexpected questions is a different problem.

 The patterns above are the ones that made the difference in practice.

 ~


======================================================================
# Part 3: Your RAG Pipeline Is Slow Because You’re Sending Entire Pages to the LLM
URL: https://jamiemaguire.net/index.php/2026/05/16/rag-pipeline-slow-sending-entire-pages-llm/
======================================================================

I recently debugged an MCP documentation search implementation running on .NET that was taking 18–25+ seconds per query.

 Far too slow for interactive use.

 The pipeline was straightforward: Elasticsearch KNN vector search, a reranker, then GPT-4.1-mini for synthesis.

 The latency was making it unusable inside GitHub Copilot and VS Code.

 Before touching any config or swapping the model, the right move was to add diagnostic logging and find out what was actually being sent to the LLM. What came back was instructive.

 Let’s dig in.

 ~

 The Setup

 The RAG pipeline looked like this:

 Incoming query gets embedded

 Elasticsearch KNN search retrieves the top n chunks

 A reranker orders them by relevance

 The top chunks are passed to GPT-4.1-mini for synthesis

 The generated answer is returned to the client

 The code responsible for preparing chunks before synthesis lived in a method CleanChunks :

 private IEnumerable<RagChunk> CleanChunks(IEnumerable<RagChunk> chunks)
{
 return chunks
 .Where(c => c.Score >= MinRelevanceScore)
 .OrderByDescending(c => c.Score)
 .Take(MaxChunksForSynthesis);
} 

 Clean and simple.  It turned out this was the source of the problem.

 ~

 What the Logs Actually Showed

 Diagnostic logging was added to capture chunk count, token count, and document URIs before the synthesis call. The output for a straightforward deployment query looked like this:

 Chunks retrieved: 8
Top chunk score: 8.665282
Document URIs: deploying-wasm/, deploying-wasm/, deploying-wasm/,
 deploying-wasm/, deploying-wasm/, api-reference/,
 api-reference/, quickstart/
Token count sent to synthesis LLM: ~19,700 

 Five of the eight chunks came from the same document URI, each with an identical relevance score of 8.665282.

 The KNN search was returning paragraph-level chunks from the same page, so the synthesis step was receiving the equivalent of an entire documentation page rather than a few focused, diverse results.

 Nearly 20,000 tokens for a single query.

 The model wasn’t slow. The chunk size wasn’t the problem. The pipeline was sending too much because there was no deduplication by document source.

 ~

 The Fix: GroupBy DocumentUri

 One LINQ change in CleanChunks :

 private IEnumerable<RagChunk> CleanChunks(IEnumerable<RagChunk> chunks)
{
 return chunks
 .Where(c => c.Score >= MinRelevanceScore)
 .OrderByDescending(c => c.Score)
 .GroupBy(c => c.DocumentUri)
 .SelectMany(g => g.Take(1)) // or Take(2) for multi-section docs
 .Take(MaxChunksForSynthesis);
} 

 After the fix:

 Chunks retrieved: 3 (down from 8)
Token count sent to synthesis LLM: ~12,266 (down from ~19,700)
Latency: reduced significantly 

 GroupBy(c => c.DocumentUri).SelectMany(g => g.Take(1)) ensures you get at most one chunk per source document before applying the overall Take(MaxChunksForSynthesis) cap.

 If documents have meaningful multi-section structure and a single chunk feels too restrictive, Take(2) per group is a reasonable compromise.

 The key is that you’re no longer sending five paragraphs from the same page when you have a limit of eight chunks total.

 ~

 What Else the Logging Revealed

 Two more things surfaced once instrumentation was in place.

 Three LLM calls per request

 There were three separate synthesis calls happening for each query, not one. One of those calls was receiving approximately 62,860 tokens, suggesting a code path that bypassed CleanChunks entirely.

 This hadn’t been visible without per-call logging. Counting your LLM calls per request matters; in a pipeline with multiple steps, it’s easy to accumulate calls that multiply both cost and latency.

 A semantic cache that wasn’t hitting

 The system included a semantic similarity cache using KNN search against previously logged queries at a 0.92 threshold, the intent being to skip the vector store and LLM entirely for queries similar to recent ones.

 Two problems:

 Cache hits were logged at Debug level. At the default Information log level, you’d never see them. There was no way to know if the cache was working without changing the log configuration.

 The KNN query approach was inconsistent. The cache’s FindSimilarAsync method used .Knn() nested inside .Query() , which applies a different score normalisation than the top-level knn raw JSON approach used in the main vector search. The result: the 0.92 threshold was almost certainly never being met, making the cache effectively dead code.

 The fixes here are: raise cache hits to Information level from day one, and align the KNN query construction across both paths so scores are comparable.

 ~

 What to Watch Out For

 KNN vector search returns chunks, not documents. If your corpus has paragraph-level chunking, a single document will produce multiple high-scoring chunks with identical or near-identical scores. Without deduplication, your top-N chunks may all come from the same page.

 Token count is the first thing to log in a slow LLM pipeline. Before questioning the model, the infrastructure, or the chunk size, log what you’re actually sending. The answer is usually there.

 Semantic caching is invisible without the right log level. A cache that only logs at Debug is a cache you can’t monitor in production. If you can’t see hits, you can’t tune the threshold, and you can’t tell if it’s doing anything at all.

 Count your synthesis calls explicitly. One request doesn’t mean one LLM call. Log each call with its token count separately, a single unguarded code path can account for most of your latency.

 ~

 Takeaway

 The fix itself ,  GroupBy(c => c.DocumentUri).SelectMany(g => g.Take(1)) , took seconds to write.

 The work is in knowing to look for it.  Token count logging before the synthesis call is one of the most useful diagnostic metrics you can add to your RAG pipeline. Audit this from the start.

 Once token counts were visible, the bottlenecks stopped being mysterious.

 Everything else followed from that.

 ~


======================================================================
# Part 4: The RAG Workbench I Actually Needed
URL: https://jamiemaguire.net/index.php/2026/05/23/the-rag-workbench-i-actually-needed/
======================================================================

Most RAG tutorials hand you the same shape of code.

 Load some content. Chunk it. Embed it. Query it.

 It works for a demo, but the moment you try to run that workflow repeatedly across different documentation sites, retrieval quality becomes the real problem -not embeddings, not vector databases, not orchestration frameworks.

 Debugging retrieval is where the time goes.

 You change a content selector. Re-ingest. Query again.

 The chunking is wrong.  Fix it. Re-ingest. Query again.

 The crawler pulled navigation instead of the article body.  Fix it. Re-ingest. Query again.

 Every iteration takes longer than it should because the infrastructure sits in the middle of the feedback loop.

 What I wanted was not another production-ready RAG platform.  I wanted a workbench.

 Something that would let me:

 Crawl a documentation site

 Run end-to-end ingest locally

 Inspect chunks and retrieval scores

 Iterate on parser and chunking config quickly

 Debug retrieval failures without fighting infrastructure

 Carry the working configuration into production later

 That became DocIngestion.  Built in .NET using Claude Code, it’s a local-first RAG ingestion and retrieval inspection tool designed around one thing:

 shortening the retrieval debugging loop.

 Lets dig in.

 ~

 The Architectural Decision That Changed Everything

 The most important decision in the entire tool was also the simplest:  a single IVectorStore abstraction with 3 implementations.

 public interface IVectorStore 
{
 Task UpsertAsync (
 string indexName ,
 IReadOnlyList < Chunk > chunks ,
 CancellationToken ct );

 Task < IReadOnlyList < ScoredChunk >> QueryAsync (
 string indexName ,
 float [] queryEmbedding ,
 int topK ,
 CancellationToken ct );

 Task < IReadOnlyList < IndexedDocument >> ListDocumentsAsync (
 string indexName ,
 CancellationToken ct );
} 

 That interface sits underneath the ingest pipeline:

 The pipeline never knows where the chunks are stored.

 There are only two implementations:

 JSON on disk for iteration

 Elasticsearch for production parity

 That’s it.

 For most early-stage retrieval work, JSON is the better tool.  Not because it’s scalable, but because it’s quick, easy to eyeball, and gives you something good enough to iterate on during those crucial early stages.

 You can optimise for scale and production later, once you’ve nailed down retrieval quality and pipeline behaviour.

 ~

 Why JSON Beats a Vector Database Early On

 When most people start a RAG project, they immediately provision infrastructure.

 Pinecone. Qdrant. Elasticsearch. Azure AI Search.  That makes sense once the retrieval behaviour is already understood.

 Before that, it’s mostly friction.  For a few thousand chunks, a JSON file is more than enough.

 The implementation is deliberately primitive:

 public sealed class JsonFileVectorStore : IVectorStore 
{
 public async Task < IReadOnlyList < ScoredChunk >> QueryAsync (
 string indexName ,
 float [] queryEmbedding ,
 int topK ,
 CancellationToken ct )
 {
 var chunks = await LoadAsync ( indexName , ct );

 return chunks 
 . Select ( c => new ScoredChunk (
 c ,
 CosineSimilarity ( queryEmbedding , c . Embedding )))
 . OrderByDescending ( s => s . Score )
 . Take ( topK )
 . ToList ();
 }
} 

 No infrastructure.  No provisioning.  No auth.  No cluster configuration.

 Just:

 crawl

 ingest

 inspect

 query

 rerun

 That changes the speed of iteration completely.  A broken content selector stops being a deployment problem and becomes a two-minute fix.

 ~

 Retrieval Debugging Is the Real Problem

 The most useful part of the workbench is not ingestion.  It’s inspection.

 When retrieval quality is poor, there are usually only three questions:

 Is the correct chunk in the index?

 Is it being retrieved?

 Is it ranking highly enough?

 The workbench surfaces all three immediately.  Query a tenant and the UI shows:

 the synthesized answer

 retrieved chunks

 similarity scores

 source URLs

 indexed page counts

 That turns retrieval debugging into a fast feedback loop instead of an infrastructure exercise.

 If the chunk is missing entirely, the crawler or parser is wrong.

 If the chunk exists but never retrieves, the embedding or query phrasing is wrong.

 If the chunk retrieves but ranks badly, the chunking strategy is usually the issue.

 That sounds obvious written down.  In practice, most RAG tooling often makes answering those questions slow.

 ~

 The Bugs This Catches Quickly

 The inspection workflow caught issues that would have taken hours to diagnose inside a cluster:

 A content selector scoped to the navigation panel instead of the article body, producing near-identical chunks across pages

 A chunker splitting code samples mid-function so signatures and implementations were separated

 A crawl indexing CDN error pages as canonical content across dozens of URLs

 All of those became obvious by looking at:

 chunk counts

 retrieval scores

 duplicate URLs

 chunk text directly on disk

 That’s the part most RAG tutorials skip.  Retrieval problems are usually data problems.

 The faster you can inspect the data, the faster you improve retrieval.

 ~

 Elasticsearch Still Matters

 The point is not that JSON replaces a vector database.  It doesn’t.  The JSON store is intentionally limited:

 everything loads into memory

 concurrent writes are unsafe

 performance drops as chunk counts grow

 The point is that JSON is a better environment for discovering what “good retrieval” actually looks like.  Once retrieval quality is understood, switching to Elasticsearch is a config change:

 "storage" : {
 "type": "elasticsearch" 
} 

 The pipeline code stays the same.

 That’s where the abstraction earns its keep.

 Whatever I learn while iterating locally, parser rules, chunk sizes, overlap strategy,  or retrieval tuning, it all carries directly into the production implementation.  No rewrite required.

 No migration project.  No second ingest pipeline.

 ~

 The Most Useful Part

 The most useful abstractions are the ones that let you defer decisions until you have enough evidence to make them properly.

 IVectorStore is not a complicated abstraction.  It’s three methods.

 What it gave me was a way to:

 debug retrieval quickly

 iterate locally

 inspect data directly

 avoid infrastructure too early

 move into Elasticsearch later without changing the pipeline

 Most RAG problems are not solved by adding more infrastructure.

 They’re solved by shortening the feedback loop between ingest and retrieval quality.

 That’s what this workbench was built for.

 ~

 Enjoy what you’ve read, have questions about this content, or would like to see another topic?

 You can schedule a call using my  Calendly link  to discuss consulting and development services.

 ~

 Courses

 Check my AI courses.  From developers to decision makers, these have you covered:

 Developing an Artificial Intelligence Strategy for Your Organization 

 Aligning Generative AI with Business Cases 

 Vector Databases & Embeddings for Developers 

 ~


======================================================================
# Part 5: Building a RAG Administration Tool with .NET, Elasticsearch and OpenAI
URL: https://jamiemaguire.net/index.php/2026/05/30/building-a-rag-administration-tool-with-net-elasticsearch-and-openai/
======================================================================

When you build and ship a RAG pipeline, you quickly discover that the real work is not the initial vectorisation pass. It is keeping the pipeline healthy after that.

 Documents get deleted, re-crawled content fails to embed, indices drift out of sync, and what was clean on day one slowly becomes unreliable.

 You end up with vector chunks pointing at content that no longer exists, and content records claiming to be vectorised when the vectors are long gone.

 I hit exactly this problem while working on a documentation pipeline for a client and, rather than reach for a generic admin tool or run Elasticsearch queries manually, I built something specific to the job.

 This post walks through what I built, why I built it, and how I use it.

 ~

 Overview

 The DocManagement tool is a localhost administration panel for managing the health of a RAG pipeline.

 It sits on top of two Elasticsearch indices: one holding document metadata (URLs, titles, body content, platform tags and deletion flags) and another holding the vector chunks used for semantic search.

 The tool provides a live view of both indices and lets you take action on individual documents when things go wrong.

 The backend is a .NET Minimal API talking to a plain HTML and vanilla JavaScript frontend.

 No framework overhead, no authentication (it is a local development tool), and no background jobs. Everything is explicit and synchronous so you always know exactly what is happening.

 ~

 Why I Built It

 The short answer is that Postman is fine for querying data but terrible for day-to-day operational work.

 Running _delete_by_query manually, updating individual fields through APIs, and mentally tracking which documents are in which state quickly becomes error-prone.

 The longer answer is that the two-index pattern I am using creates a specific class of problem I started calling zombies.

 A zombie is a soft-deleted document that still has vector chunks sitting in the vector index.

 It is not serving users because the content record is marked as deleted, but it is still polluting the vector space and potentially influencing retrieval.

 These accumulate silently as content gets updated or removed. I needed a way to identify them instantly and clean them up with a single click rather than a sequence of curl commands.

 ~

 The Two-Index Architecture

 Most RAG implementations use a single index where documents contain both metadata and vector representations.

 I took a different approach by splitting them into two indices because the content and the vectors have different lifecycles.

 The content index is written by the crawler. Documents are soft-deleted rather than hard-deleted so they can be recovered if a crawl goes wrong. The index stores body content, platform tags, content types and the flags that drive pipeline state.

 The vector index contains embedded chunks. A single document typically produces between 10 and 40 chunks depending on length, with each chunk stored as an individual document keyed by its parent URL.

 That makes vector cleanup simple. Removing vectors for a document becomes a straightforward _delete_by_query operation rather than a partial update against a nested structure.

 The trade-off is that the two indices can drift out of sync. That is the entire reason this tool exists.

 ~

 How It Works

 The tool connects to Elasticsearch on startup and inspects the index mappings to understand the field structure.

 This matters because the content index contains a field named uRL — a legacy artefact from an earlier C# serialisation model. The tool needs to determine at runtime whether uRL.keyword exists before building queries. It is exactly the sort of detail that quietly breaks hardcoded search logic.

 The backend exposes a small set of Minimal API endpoints:

 Statistics endpoints for document counts and health metrics

 Search endpoints with filtering and pagination

 Action endpoints for un-delete, re-queue, delete vectors and re-vectorise operations

 The frontend is roughly 280 lines of vanilla JavaScript. It renders the results table, handles filtering, and calls the API endpoints. Destructive actions require confirmation before execution.

 The backend also includes an EnvironmentManager that allows switching between staging and production Elasticsearch clusters at runtime.

 That becomes useful when search quality looks suspicious in production and you need to inspect document state without performing write operations.

 ~

 Re-Vectorisation and Chunking 

 The administration tool uses exactly the same chunking logic as the main ingestion pipeline.

 That sounds obvious, but it matters. If the administration tool chunks content differently from the production pipeline, manually regenerated vectors will not match the vectors created during normal ingestion.

 The chunker is paragraph-aware. It splits on paragraph boundaries first rather than hard character counts, which keeps sentences intact and produces chunks that read as coherent units of information.

 Fenced code blocks are preserved in full regardless of size. Splitting a code example halfway through usually produces chunks that are unhelpful both for embeddings and for users viewing retrieved results.

 Each chunk is capped at 1,500 characters with a 225-character overlap between neighbouring chunks. The overlap helps preserve context when relevant information sits near a chunk boundary.

 Splits always occur at word boundaries and never in the middle of a token.

 For a typical documentation page this produces somewhere between 10 and 40 chunks, depending on content type and length.

 One thing I have found over time is that chunking strategies evolve as new content types are onboarded. What works well for API reference documentation is not always what works well for long-form conceptual content.

 ~

 Features

 The main features are:

 Dashboard Statistics 

 The dashboard displays live counts for:

 Total content documents

 Total vector chunks

 Vectorised documents

 Non-vectorised documents

 Soft-deleted documents

 Zombie documents

 This gives a quick snapshot of pipeline health before drilling into individual records.

 Search and Filtering 

 Documents can be searched using URL wildcards and filtered by:

 Platform

 Content type

 Deletion status

 Vectorisation status

 Each result row also displays the associated vector chunk count.

 Un-Delete 

 Clears the isDeleted flag and resets hasVectors so the document is picked up during the next vectorisation cycle.

 Re-Queue 

 Sets hasVectors to false without touching existing vectors.

 Useful when a document needs reprocessing and vector cleanup has already been performed elsewhere.

 Delete Vectors 

 Executes a _delete_by_query operation against the vector index for a specific URL.

 All vector chunks are removed while leaving the content record untouched.

 Re-Vectorise 

 This is the primary maintenance operation.  The tool:

 Deletes any existing vectors

 Re-chunks the document

 Generates fresh embeddings using OpenAI

 Bulk indexes the new vectors

 Marks the document as vectorised

 Embedding requests use exponential backoff (2, 4, 8, 16 and 32 seconds) to handle rate limiting.

 The process is synchronous and typically takes between 20 and 40 seconds per document.

 ~

 How I Use It

 My usual workflow starts with an automated email generated by Claude Dispatch.

 The email provides a summary of content health across the platform.

 I open the tool, check the dashboard statistics and look at the zombie count. If it is non-zero, I filter for deleted documents that still have vectors and remove the remaining chunks.

 It takes about five minutes and keeps the vector space clean.

 When documentation updates are released, I search for the affected URLs and inspect their chunk counts. Unexpectedly high counts often indicate old vectors that were never cleaned up properly.

 If production retrieval quality starts behaving strangely, I switch the tool into production mode, inspect document state and vector counts, and then switch back to staging before making any changes.

 ~

 Lessons Learned

 A few things surprised me during the build.

 The field naming issue caught me off guard. Different services had written the Elasticsearch indices at different times using different serialisation conventions. One index used camelCase, another used UPPER_SNAKE_CASE, and the uRL field was a side effect of that history. The lesson is simple: inspect your mappings before writing query logic.

 Synchronous re-vectorisation turned out to be more practical than expected. I considered implementing server-sent events to stream progress back to the browser, but the documents are small enough that a 30-second request is perfectly acceptable. The simpler implementation won.

 The _msearch batch request for vector counts was necessary. My first version issued one count query per result row. A page showing 25 documents generated 25 Elasticsearch requests. Replacing those with a single _msearch request reduced page load time by roughly 80%.

 Finally, keep chunking behaviour consistent everywhere vectors can be generated. If administration tools and ingestion services chunk content differently, retrieval behaviour becomes inconsistent and difficult to reason about.

 ~

 Why Administration Matters 

 Most discussions around RAG systems focus on embeddings, retrieval strategies and model selection.

 In practice, a surprising amount of operational effort goes into answering much simpler questions:

 Which documents are vectorised?

 Which documents are stale?

 Which documents should no longer exist in the vector index?

 Which documents failed processing?

 Those questions become increasingly important as the amount of indexed content grows.

 This tool gives me visibility into those answers without requiring a Kibana query every time something looks suspicious.

 It is tailored to the schema I have, the operations I perform regularly, and the failure modes I have actually encountered. That specificity is what makes it useful.

 If you are running a similar two-index RAG architecture and finding that Kibana, Postman or curl commands is becoming unwieldy, I would encourage you to build something similar. The implementation took only a few days and has already paid for itself many times over in avoided debugging sessions.

 ~

 Enjoy what you’ve read, have questions about this content, or would like to see another topic?

 You can schedule a call using my  Calendly link  to discuss consulting and development services.

 ~

 Courses

 Check my AI courses.  From developers to decision makers, these have you covered:

 Developing an Artificial Intelligence Strategy for Your Organization 

 Aligning Generative AI with Business Cases 

 Vector Databases & Embeddings for Developers 

 ~


======================================================================
# Part 6: A Developer’s Guide to Building RAG Systems: Lessons from the Trenches
URL: https://jamiemaguire.net/index.php/2026/06/27/a-developers-guide-to-building-rag-systems-lessons-from-the-trenches/
======================================================================

I have spent a good while now building RAG pipelines across a number of engagements, mostly for documentation sites, and the gap between the tutorials and the reality of production is substantial.

 This post is not a getting started guide. There are plenty of those.

 This is a collection of things I got wrong, things I got right eventually, and things I wish someone had told me before I started, drawn from the work rather than from any single project.

 It spans the whole pipeline rather than one corner of it: not just ingestion and chunking, but the retrieval and synthesis side where slow queries and silently duplicated context live, and the operational side where ingestion jobs die quietly and indices drift out of date.

 Some of the hardest lessons were the ones furthest from the embedding model.

 I will also talk about how my build process changed over time, specifically how I went from writing everything by hand to leaning heavily on Claude Code to accelerate development without sacrificing quality or understanding.

 Let’s dig in.

 ~

 How It Started

 The arc tends to be the same on these engagements. The first pipeline I ever built was entirely manual. I wrote the crawler, the chunker, the embedding step, and the query layer from scratch in .NET 9.

 No AI assistance, no scaffolding. I wanted to understand every moving part before I let anything generate code for me.

 That was the right call.

 The problem was that the iteration cycle was slow. Every new documentation site meant going back into the code to adjust selectors, tweak chunking parameters, or add a new filter.

 What I had built was a single-purpose pipeline masquerading as something more general, and that is a trap worth naming because it is the default outcome if you are not deliberate about it.

 So at some point I started using Claude Code more deliberately, not to write code for me, but to accelerate the parts I already understood. I would describe what I wanted, review the output, adjust it, and move on. The key shift was treating it as a collaborator that needed direction rather than a tool that would figure it out on its own.

 I want to be direct about this because there is a version of this post where I undersell it. Claude Code materially accelerated the work, not by replacing my understanding of what I was building, but by removing the friction between understanding something and having it implemented.

 When I knew what I wanted, describing it clearly and getting back something close to correct was significantly faster than writing it from scratch, and that time went back into thinking about the design, testing the behaviour, and figuring out what to build next.

 The parts that went wrong were the parts where I gave it a vague brief and accepted the output without reviewing it carefully. The field naming mismatch I describe later in these lessons is a good example. I described what I needed but did not think through the schema detection logic myself, and the first version made assumptions that did not hold.

 The pattern that works is to understand the domain well enough to know what correct looks like, use Claude Code to get there faster, and review the output with the same care you would apply to your own code.

 It is a collaboration, not a delegation. That combination of me knowing the domain and Claude Code handling the boilerplate is what has let me stand up the moving parts of a pipeline quickly on each new engagement without losing the plot.

 ~

 The Tools You End Up Building

 Across these engagements, the same two tools tend to emerge. I doubt this is specific to me. If you do this work for any length of time you will likely build your own versions of both, so it is worth describing them as patterns rather than products.

 The ingestion pipeline is the obvious one. You give it a URL and a config file and it handles the rest: crawling the documentation site, extracting the meaningful content, chunking it, generating embeddings, and indexing everything into a vector store. You can then send it a natural language question and get back an answer synthesised by an LLM with numbered citations.

 Each project gets its own config, its own index, and its own system prompt, which is what lets the same pipeline serve more than one site.

 The admin tool is the one people underestimate. It is a localhost panel for managing the health of a RAG pipeline. It sits on top of a content index and a vector index and gives you a live view of what is in good shape and what is not. It lets you un-delete documents, re-queue them for embedding, delete orphaned vector chunks, and re-vectorise individual documents when something goes wrong.

 Both patterns come out of the same realisation: building a RAG pipeline is not hard, but keeping it healthy over time is.

 ~

 Part 1: Architecture & Ingestion

 Getting the pipeline stood up is one thing. Getting it stood up in a way that does not fight you for the rest of the engagement is another. The decisions you make here — about storage abstractions, content scoping, chunking, IDs, and configuration — either compound in your favour or accumulate as debt. Most of the mistakes in this section are things I had to retrofit rather than build in from the start.

 ~

 Lesson 1: Start Without Infrastructure

 The first thing I get wrong on every new project is reaching for a proper vector database too early. Spinning up a managed vector store or a dedicated search cluster for an experiment that might not go anywhere is overkill.

 The ingestion pipeline has a dual storage strategy for this reason. The vector store sits behind an interface with two implementations: a file-backed store that runs cosine similarity in memory, and a heavier production store that runs proper vector searches at scale. Switching from one to the other is a single field change in the config file. The point is the abstraction, not the particular backends. Pick whichever lightweight option and whichever production option suit your stack.

 The workflow this enables is simple. Start with the in-memory store, validate that the crawl and chunking are producing sensible results, then switch to the production store when you are ready to scale. No code changes, no migration scripts, just a config update.

 If I had insisted on the production store from day one, I would have spent the first week setting up infrastructure instead of figuring out whether the chunking strategy was actually working.

 ~

 Lesson 2: Content Scoping Is Not Optional

 When you crawl an HTML page and dump the raw text into a chunk, you get the navigation bar, the cookie banner, the footer links, and the social share buttons alongside the actual content. All of that ends up in your embeddings.

 The result is that queries about real content compete with chunks full of “Home About Blog Contact” and “Subscribe to our newsletter.” Your retrieval quality degrades and it is not obvious why.

 The ingestion pipeline uses XPath selectors from the config to scope extraction to the meaningful part of the page:

 "parser": {
 "contentSelector": "//main | //article",
 "excludeSelectors": ["//nav", "//footer", "//aside", "//script", "//style"]
}
 
 This is the single most impactful thing you can do for retrieval quality before you touch chunking or embedding models. Get the right content in, and everything downstream gets easier.

 ~

 Lesson 3: Chunking Strategy Actually Matters

 The naive approach is to split on a fixed character count. It works well enough to demo but poorly enough in production to be worth fixing.

 The problems are twofold. First, fixed-character splits break sentences in the middle, which produces chunks that are confusing to the embedding model and useless to users who see them in search results. Second, if a key sentence falls right at the boundary between two chunks, without overlap it only appears in one of them. Depending on where the query lands semantically, you might miss it entirely.

 One approach that works well is a paragraph-aware chunker: split on paragraph boundaries first, then enforce a size cap, with overlap between adjacent chunks. Fenced code blocks are preserved in full regardless of length, because splitting a code example mid-block produces something that is not useful to anyone.

 A sliding window approach works too, and the pipeline defaults to 800 tokens with 100-token overlap. The overlap is the critical part. It means a sentence near the end of one chunk also appears at the start of the next, which makes retrieval at boundaries much more reliable.

 One practical detail: I use a 4-chars-per-token approximation rather than running a tokeniser on every chunk. It is good enough for chunking purposes and avoids a dependency.

 ~

 Lesson 4: Give Each Chunk a Deterministic ID

 This one sounds obvious in retrospect but I did not do it correctly the first time.

 If your chunk IDs are random UUIDs, re-running the ingestion pipeline creates duplicate entries. You end up with two copies of every chunk, which doubles your vector store size and skews retrieval scores. The fix is to make chunk IDs deterministic.

 The approach that works is to derive each chunk ID from a hash of the source URL combined with the chunk index. That way the same chunk always lands on the same ID, so running the pipeline twice overwrites the first run rather than duplicating it. If you are running multiple projects or tenants out of one store, fold a project identifier into the ID as well so they cannot collide. This makes re-ingestion safe and removes a whole category of data quality problems.

 ~

 Lesson 5: Separate Your Content and Vector Lifecycles

 Most RAG implementations use a single index where each document has both its metadata and its vector representation. I took a different approach and split them across two indices, a content index and a vector index.

 The reason is that content and vectors have different lifecycles. Content gets updated by the crawler on whatever schedule you run it. Vectors depend on the embedding model you are using, the chunking algorithm, and whether the source content has changed. When you update the chunking logic, you need to re-embed everything, but you do not need to re-crawl. When a document is deleted from the source site, you want to soft-delete it in the content index and eventually clean up its vectors, but those are separate operations.

 The tradeoff is that the two indices can get out of sync, which is precisely why a split like this needs tooling to keep them reconciled. But the flexibility it gives you is worth the operational overhead, especially if you are likely to change your embedding model or chunking strategy over time.

 ~

 Lesson 6: Zombies Are a Real Problem

 A zombie is a soft-deleted document that still has vector chunks in the vector index. It is not serving any users because the content is marked as deleted, but it is polluting the vector space and skewing search results. These accumulate silently over time as content gets updated or removed from the source site.

 I did not have visibility into this problem until I built the admin tool. A stats panel shows a live zombie count and you can filter for them directly. Cleaning them up is a one-click operation rather than a sequence of ad hoc queries against the index.

 The broader point is that RAG pipelines are not fire-and-forget. You need visibility into the state of your indices and the ability to intervene at the document level. Building the tooling to do that is not glamorous work, but it is what keeps retrieval quality from degrading over time.

 ~

 Lesson 7: Configuration Belongs in Config, Not Code

 My first pipeline had site-specific logic scattered through the code: hardcoded content selectors, a specific chunking size, a particular system prompt for the LLM. Anything that varied from one documentation site to the next was baked into the source, so adapting it to a second site meant going back into the code.

 The better approach is to make every site-specific detail a property of a config file and keep the pipeline logic generic. A config describes the seed URLs, the content selectors, the chunking parameters, the embedding model, the storage backend, and the system prompt. The pipeline code never changes; only the config does. Whether you are serving one corpus or fifty, the same principle holds, and it is what makes multi-tenancy almost free when you do need it, because a new tenant is just another config rather than another code path.

 The practical benefit is that standing up a new site is a matter of minutes, not hours. You drop a config file in place and point the pipeline at it. The pipeline handles the rest.

 ~

 Part 2: Embeddings & Retrieval

 Getting content into the index correctly is only half the problem. The retrieval side has its own failure modes, and most of them are quieter. Embeddings silently mismatched. Chunker fast paths that only reveal themselves when a 40KB code block hits the API. Token limits that look fine by approximation and aren’t. Duplicate pages flooding the synthesis context. And logging gaps that let all of it hide in plain sight. These are the lessons that cost me the most time to diagnose.

 ~

 Lesson 8: Query Embedding Must Match Ingestion Embedding

 This is a simple one but easy to get wrong if you are iterating quickly. The model you use to embed chunks at ingestion time must be the same model you use to embed queries at retrieval time. If they diverge, cosine similarity scores become meaningless.

 In the ingestion pipeline the embedding model is set in the config file, and the same config drives both ingestion and query, so there is no way to accidentally mismatch them. However you manage it, make sure the model is a property of the index, not a runtime parameter.

 ~

 Lesson 9: Watch Out for the Fast Path in Your Chunker

 This one took me a while to track down because the symptom and the cause were in different places. A common optimisation in chunkers is to pull code blocks out and replace them with short placeholders before measuring the content length. The idea is to reason about the prose without the code getting in the way. The problem is what happens next. If the total length with the placeholders in place looks small enough to skip splitting, the whole thing gets returned as a single chunk once the placeholders are swapped back out for the real code.

 The failure mode is a page with a few hundred characters of prose wrapped around one large code block. With the placeholder in place it looks tiny, so it sails straight through the fast path. Then the real code goes back in and you have a chunk of tens of thousands of characters. The embedding API rejects it and the whole document fails to vectorise. The fix is to measure the restored length, not the placeholder length, before deciding whether to split. Whenever you substitute a stand-in for the real content to make a decision easier, make sure the decision is made against what you are actually going to send.

 ~

 Lesson 10: Atomic Code Block Protection Needs a Size Cap

 Treating code blocks as atomic units is the right instinct. I said as much back in the chunking lesson, and I stand by it. Splitting a code example mid-block produces something useless to a reader and confusing to the embedding model. But “never split” cannot quietly become “no size limit”. A large markup layout or a long fenced code block will be emitted whole regardless of how far it exceeds the embedding model’s token limit, and at that point your good intention has become the thing that breaks ingestion.

 Code blocks still need a maximum size check. The right behaviour is that blocks over the limit get split at a safe boundary, a closing brace or a blank line, rather than emitted whole or dropped entirely. You lose the guarantee that a block is never broken, but you only lose it for the blocks that were too big to keep anyway, and a block split at a sensible boundary is far better than an embedding call that fails outright.

 ~

 Lesson 11: Token Count Approximations Break Down for Code-Heavy Content

 Using a character-based approximation for token counting is the right call, and I said as much back in the chunking lesson. Running a full tokeniser on every chunk adds latency and a dependency you do not need during chunking. The standard approximation of around four characters per token works well for natural language prose. The trouble is that it does not work as well for code, markup, or XML.

 Code tends to run closer to two or three characters per token because of the density of special characters, short identifiers, and whitespace. A chunk that looks comfortably within your size budget by the character approximation can sail past the embedding model’s actual token limit once it is measured properly. This is the same failure as the oversized code blocks from earlier, just arriving through a different door. If your content is documentation-heavy with a lot of code examples, the answer is to be more conservative where it counts. Apply a tighter size cap to chunks you have identified as code, or treat the approximation as a soft limit and give anything near the boundary more careful handling rather than waving it straight through.

 ~

 Lesson 12: Apply the Same Guards to Ingestion That You Apply to Queries

 Both the query path and the ingestion path call the same embedding model, but they are usually written at different times and by someone focused on different problems. Query-side guards get added when you are thinking about search quality and edge-case user input. Ingestion-side guards get added when something breaks during a crawl. The result is that the two paths drift apart even though they are talking to the same model with the same limits.

 The way this bit me was a character limit. I added one before calling the embedding API on the query side, while I was thinking about what a user might paste into a search box, and forgot entirely that the exact same limit applies when generating embeddings for chunks. The chunk side then failed silently on anything long enough to hit the model’s context limit, and I only found out because a document never appeared in search results. The fix is a habit rather than a line of code. Any defensive logic that exists on one path should exist on the other. Review them together, not independently, because the model does not care which path the text arrived on.

 ~

 Lesson 13: Deduplicate Retrieved Chunks by Document URI

 Lesson 4 was about preventing duplicate chunks at ingestion time. This is the other duplication problem, and it lives on the retrieval side where it is much easier to miss.

 A vector search will happily return several chunks from the same source page, often with identical relevance scores, because they are paragraph-level slices of one document that all match the query equally well. I had a query return five chunks from the same page, every one scoring 8.665282, different paragraph offsets, same URL. Nothing in the pipeline flagged it. The synthesis step was being handed the equivalent of an entire documentation page, around 19,700 tokens, when it should have had a few focused chunks.

 The fix is to group by document URI and take one chunk per document before applying your top-N limit:

 return chunks
 .Where(c => c.Score >= MinRelevanceScore)
 .OrderByDescending(c => c.Score)
 .GroupBy(c => c.DocumentUri)
 .SelectMany(g => g.Take(1)) // Take(2) is reasonable for multi-section pages
 .Take(MaxChunksForSynthesis);
 
 That one change dropped the same query from roughly 19,700 tokens to around 12,000 and took a meaningful chunk off the latency. Take one per document if you want maximum diversity, two if a single chunk feels too thin for long pages. The point is that without this step you are silently sending whole pages to the model and paying for it in both latency and answer quality.

 ~

 Lesson 14: Before You Blame the Model, Log What You Are Sending It

 When a RAG query is slow, the instinct is to suspect the model. I spent time convinced the problem was chunk size, then convinced it was the synthesis model being too slow. Both were wrong, and I only found that out by adding diagnostic logging to the synthesis step: number of chunks retrieved, total token count going to the model, and the document URIs of those chunks.

 The logs made the real cause obvious in seconds, and it was the duplicate-page problem from the previous lesson. Token volume was the variable that mattered, and I had been guessing at everything except the one number that would have told me.

 The habit worth forming is that token count is the first thing you log in any slow LLM pipeline, before you touch the model choice or the chunking parameters. The model is rarely the bottleneck. What you are feeding it usually is, and you cannot reason about that without measuring it.

 ~

 Lesson 15: Count Your LLM Calls Per Request

 While I was logging token counts I found something I was not looking for. A single query was making three separate synthesis calls, not one, and one of those calls was receiving roughly 62,860 tokens because it ran through a code path that bypassed the chunk-cleaning logic entirely.

 I had been reasoning about the pipeline as though one query meant one model call. The mental model was wrong, and the cost of that was multiplied latency that no amount of tuning the single call I knew about would have fixed.

 Instrument the number of model calls per request as a first-class metric, the same way you instrument token count. A pipeline that quietly fans out into multiple synthesis calls, or has an older path that skips your guards, will not show up in any single-call profiling. You have to count.

 ~

 Lesson 16: A Cache Is Only Worth Having If You Can Prove It Hits

 I added a semantic cache to the query path: embed the incoming query, check it against recently logged queries with a vector search, and if something scores above a similarity threshold, return the cached answer instead of hitting the model. Sensible idea, and it appeared to work.

 It was dead code. Two problems compounded. The cache hits were logged at debug level, invisible at the default log level, so I had no way to see whether it was ever hitting without reconfiguring logging. And the similarity search used a different score normalisation from the one used elsewhere in the pipeline, which meant the threshold I had set was almost certainly never being met. The cache looked alive and did nothing.

 The lesson is that a cache you cannot observe is a liability, not an optimisation. Log hits and misses at a visible level from the first day, and verify that the scores your threshold compares against are on the scale you think they are on. An optimisation you cannot confirm is firing is just untested code sitting in the hot path.

 ~

 Lesson 17: More Context Is Not a Free Optimisation

 I had an MCP tool returning paraphrased API documentation through a RAG pipeline, and the results were modest. My theory was that paraphrasing was losing exact signatures, so I built a second tool that skipped synthesis and returned raw type definitions straight from the package: precise, machine-readable, no translation loss. It felt obviously better.

 The benchmarks said otherwise. Measured against compile rate, render rate, and hallucination count, the raw type definitions made code generation worse. A single request pulled in tens of kilobytes of type information, and that volume displaced the model’s attention and introduced noise rather than grounding it. Models adapt better from a close working example than they interpret from a specification, and interpretation is exactly where hallucinations creep in. More context is not a free optimisation; it has a cost in attention, and on smaller models and simpler tasks that cost can outweigh the benefit.

 There are two lessons here. The narrow one is to prefer retrieving a working example the model can adapt over injecting raw specifications it has to interpret. The broader one is that without benchmarks I would have shipped a tool that actively hurt results and told people it was an improvement. Feeling useful is not the same as being useful, and the only thing that tells them apart is measurement.

 ~

 Part 3: Operations & Production

 Getting a RAG pipeline working is not the same as keeping it working. The operational problems are slower to surface and harder to diagnose. Status flags that lie. Indices that drift without anyone noticing. Jobs that die silently at 3am. Health checks that can fail without telling you. These lessons are the ones you only learn once the pipeline has been running long enough to accumulate history, and they are the ones I would build for from day one if I were starting over.

 ~

 Lesson 18: Audit Your Index Mappings Before You Write Any Query Logic

 This one cost me time I did not need to spend. The content and vector indices in my setup were written by different services at different times. They ended up with different field naming conventions: camelCase in one, a legacy capitalisation artifact in the other (uRL instead of url).

 The admin tool detects the field structure at runtime by inspecting the index schema on startup rather than assuming it. This is extra work, but it is much better than having queries silently return nothing because a field name does not match what you hardcoded.

 The lesson is to audit your index schema before you write any query logic, and to treat field names as something that can vary rather than something you can assume.

 ~

 Lesson 19: Batch Your Index Queries

 The first version of the admin tool’s results table made an individual vector count query per row. A 25-row results page fired 25 separate requests to the index. Page load time was painful.

 Switching to a single batched request that carried all 25 sub-queries cut the load time by around 80%. If you are building any kind of admin or inspection interface on top of a vector store, batch your queries wherever the store supports it rather than making per-row individual calls. Most engines offer some form of multi-query or bulk request for exactly this reason, and the round-trip savings add up fast.

 ~

 Lesson 20: Synchronous Re-Vectorisation Is Fine for Admin Work

 I considered adding streaming progress updates for the re-vectorisation operation in the admin tool. A document with 40 chunks means 40 embedding API calls, plus bulk indexing, which takes 20 to 40 seconds.

 I did not add streaming. The reason is that for a one-at-a-time admin operation, a 30-second wait is acceptable and the simplicity of keeping the HTTP request open is worth it. I added exponential backoff for when the embedding provider’s rate limits kick in (2, 4, 8, 16, 32 seconds) and left it synchronous.

 If I were processing hundreds of documents at once I would revisit this. For the admin workflow I actually have, synchronous is fine.

 ~

 Lesson 21: Do Not Trust a Single Status Flag as Ground Truth

 Most RAG pipelines end up with some boolean or status field that records whether a document has been vectorised. It is a convenient thing to query against, and that convenience is exactly what gets you into trouble. The flag describes what you believe happened, not what is actually in the vector index, and those two things can diverge.

 The way they diverge is through partial failure. If a bulk write to the vector index succeeds for some chunks and then fails on a later one, the error path tends to mark the whole document as not vectorised. Now the vector index holds some chunks for that document while your status field insists it holds none. The two have quietly disagreed and nothing tells you. Worse, the document shows up in your non-vectorised report, so you retry, hit the same failing chunk, the flag flips back to not vectorised, and you are exactly where you started. It never clears on its own.

 The lesson I took from this is that a status flag is a cache of an answer, not the answer itself. Do not treat it as ground truth. Periodically cross-reference it against the actual contents of the vector index and reconcile any mismatch, and when something looks stuck, go and count the real chunks rather than trusting the field. And of course, fix the underlying failure too, because a flag that keeps flipping is usually pointing at a real problem upstream, not just a bookkeeping error.

 ~

 Lesson 22: Build an Audit for Orphaned Flags

 The inverse problem also exists, and it is sneakier because nothing errors. A document can be marked as vectorised while the vector index holds no chunks for it at all. This happens when content is soft-deleted and the cleanup job does not run, or when an index migration goes partially wrong and leaves the flag set without the data behind it. It is the same divergence as the previous lesson, just pointing the other way.

 These documents are invisible to users, but they skew your health metrics and cause confusing retrieval results when you expect chunks to be there and they are not. A dedicated audit that cross-references your document records against the actual vector index and surfaces the mismatches in both directions is worth building as a first-class part of the pipeline. It is the kind of thing you do not know you need until the numbers stop adding up, and once they do, having the audit already in place is the difference between a five-minute answer and an afternoon of poking at the index by hand.

 ~

 Lesson 23: Automate a Daily Health Email

 An admin dashboard is genuinely useful, but only when you remember to open it, and the whole point of these lessons is that drift happens silently. So I added a daily email that summarises the key stats: total documents, zombie count, non-vectorised count, and the orphaned flags from the previous two lessons. Now I notice drift without having to go looking for it, which is exactly when you want to notice it.

 I kept it simple on purpose. A few headline numbers, a table of anything currently in a bad state, and a red flag at the top if the API was unreachable when the report ran. The one rule I gave myself is that the email should always arrive. If it does not turn up one morning, that absence is itself a signal that something is wrong, either with the pipeline or with the reporting around it. A health check that can fail silently is not really a health check.

 ~

 Lesson 24: Treat Ingestion as a Batch Workload, Not a Web Workload

 My ingestion crawler died overnight with no error. No exception, no stack trace, no shutdown message, the log just stopped mid-run on a routine line. It turned out the process had been killed by the operating system under memory pressure: a long single-process run, full of large transient allocations, sharing a small instance with several web apps. When the instance sat near its memory ceiling, a normal allocation spike from the crawler was enough to get it terminated, and a non-essential background job is exactly what the OS reaches for first.

 The deeper issue is a mismatch. A RAG ingestion pipeline holds full HTML documents, parsed DOM trees several times the source size, chunked text with overlap, embedding vectors, and buffered index batches. Most of those allocations are large enough to land on the large object heap, which is not compacted by default, so a long-running ingestion process grows its working set even when logical usage looks stable. That is a batch workload’s memory profile running on infrastructure shaped for request-response web apps, and the two do not fit.

 Two takeaways. First, “no error” is itself a diagnostic signal: a clean kill with no exception and no crash dump is the platform telling you it terminated the process for you, and the next question is always why. Second, give ingestion infrastructure that matches its shape. Stream responses instead of buffering them, process and discard documents one at a time rather than accumulating them, flush index batches small, and do not co-tenant a multi-hour batch job with customer-facing web apps. Better still, chunk the run into smaller triggered invocations so a failure has a small blast radius and you get restart safety for free.

 ~

 Lesson 25: Monitor Index Freshness, Not Just Job Status

 The silent death in the previous lesson had a second sting. A job-success alert would not have caught it cleanly, and even if it had, job status is the wrong thing to watch. What actually matters is whether the data in the index is current, and that is a different question from whether the last run exited zero.

 This is the classic operational gap in RAG. The job can succeed while the index quietly goes stale, or fail in a way that leaves the index partially updated, and a model retrieving stale or partial data will confidently produce wrong answers. Unlike a 500 error, nobody notices until enough users complain, by which point the trust damage is done.

 Surface a “last successful ingestion” timestamp as a metric and alert when it goes stale, rather than alerting only on job failure. A staleness alert catches both outright job failures and the subtler data-quality drift that a success-or-failure signal misses entirely. It sits alongside the health email from Lesson 23 but measures a different axis: that one tells you the index is internally consistent, this one tells you it is current.

 ~

 Lesson 26: Design for the Long Run from the Start

 A few of these lessons are things I only learned because I did not do them early enough, and if I were starting over they are the ones I would build in from day one rather than retrofit under pressure.

 I would build the admin tooling earlier. It felt like a nice-to-have when I started and became obviously essential once the indices started drifting. The visibility it provides is not optional, it is part of the pipeline, and every engagement has eventually reached the point where I wished it had been there sooner.

 I would design for re-indexing from the start. Chunking strategies change and embedding models improve, so the ability to re-index everything without re-crawling is worth designing for early. Deterministic chunk IDs and the separation of content and vector storage are both things I retrofitted rather than planned for, and retrofitting them is more painful than building them in would have been.

 I would not skip authentication even for internal tools. An ingestion pipeline running on a trusted network with no API authentication is fine right up until it is not, and it is a maintenance debt either way. Adding it later is more work than adding it at the start. The common thread across all three is that the cost of these decisions is invisible at the beginning and obvious by the time you are keeping the thing healthy in production, which is exactly the theme running through everything above.

 ~

 Closing Thoughts

 RAG is not hard to get started with. It is hard to keep working well over time.

 The tutorials give you the happy path. The reality involves content drift, index degradation, chunking regressions, and embedding model updates. The tools and patterns I have described here are all responses to real problems I encountered, not theoretical concerns.

 If you are building RAG systems for production use, invest in the observability and tooling early. The code that keeps the pipeline healthy is as important as the code that runs it.

 The ingestion pipeline and admin tool patterns I keep rebuilding across these engagements are not currently in a public repository, but if there is enough interest I am happy to write them up in more detail and share a reference implementation.

 ~


======================================================================
# Part 7: Semantic Kernel: Implementing 100% Local RAG Using Phi-3 With Local Embeddings
URL: https://jamiemaguire.net/index.php/2024/09/01/semantic-kernel-implementing-100-local-rag-using-phi-3-with-local-embeddings/
======================================================================

In an earlier blog post , we saw how to run the small language model, Phi-3, on your local machine using the ONNX Runtime.

 This let you create a simple agent hosted within a console application.

 The agent had no access to specific data and would use existing generative AI capabilities to answer human prompts.

 To increase the usefulness of your AI agents, you can ground them in your own custom data.

 A pattern often used to help achieve this is Retrieval-Augmented Generation, or RAG for short.

 RAG modifies interactions with a language model, and lets the model responds to human prompts with references to your own data.

 Most of the examples we’ve seen in recent months show how to implement RAG by consuming cloud services such as OpenAI or Azure Search. That might not be suitable for use cases -it wasn’t for me on certain projects.

 In this blog post we see how to implement a 100% local RAG solution using Semantic Kernel.  We also learn about some basic RAG concepts such as vectors and embeddings.

 Other topics include:

 Semantic Kernel Volatile Memory Store

 Embeddings Generation

 Semantic Text Memory

 Adding Embeddings to Semantic Text Memory

 Challenges with the existing Semantic Kernel SDK

 A demo of the agent in action is also included.

 ~

 Volatile Memory Store

 The volatile memory store is a simple embeddings store.  You can use create collections of memories, add, remove, update, and fetch memories.  Memories are stored as Embeddings.

 Learn more about Volatile Memory here .

 ~

 Embeddings

 An embedding is a representation of data.

 When creating agents, this data normally consists of words and sentences.  This data is then converted into numerical vectors that make it easier for the machine to understand and infer meaning.

 When the data is converted to numerical form, it can be used more optimally in other tasks such as finding the similarities between words or sentences, clustering similar content, or retrieving relevant information based on a human or agent prompt.

 A common approach to help measure the similarity between words is cosine similarity.  Learn more about cosine similarity here .

 ~

 Embedding Generator

 Embeddings (numerical vectors) are created by an Embedding Generator.

 Some examples of these include the BERT ONNX Text Embedding Generation Service , or an OpenAI’s TextEmbedding API / service such as text-embedding-ada-002 .

 Either of these will convert text into numerical vectors (embeddings) that can be stored in a memory store.

 You can see an example of how this can look with Semantic Kernel here:

   var memoryWithCustomDb = new MemoryBuilder()
      .WithOpenAITextEmbeddingGeneration("text-embedding-ada-002", apiKey)
      .WithMemoryStore(new VolatileMemoryStore())
      .Build(); 
 ~

 Semantic Text Memory

 The Semantic Text Memory class gives you methods to save, fetch, and search for information in a Semantic Memory Store (such as a Volatile Memory Store).

 Two of the key methods are:

 SaveInformationAsync

 SaveReferenceAsync

 The method SaveInformationAsync saves information into the semantic memory, keeping a copy of the source information.  This method expects a memory record with the following parameters:

 string collection,

 string text,

 string id,

 string? description = null,

 string? additionalMetadata = null,

 e.g.

 var x = 
await memory.SaveInformationAsync(“codingfacts”, id: "info1", text: "C# is a programming language.", kernel: kernel); 

 This method is best if you want to load content and subsequent query the textual contents.

 For example, you can load an entire html file with code examples into memory.  When interacting with the agent, it can formulate a good response to a coding problem, using in-memory vectorised data.

 The method SaveReferenceAsync saves information into the semantic memory, keeping only a reference to the source information.

 This method expects data with the following parameters:

 string collection,

 string text,

 string externalId,

 string externalSourceName,

 string? description = null,

 string? additionalMetadata = null,

 You might use this to load external data in URLs such as guides or other assets.  Querying this type of in-memory data will return the asset alongside a probability score of the asset matching your prompt/query.

 ~

 Bringing It All Together

 Bringing all these pieces together, we need to do the following to implement a 100% local RAG solution:

 Load the Phi-3 model

 Create the Semantic Kernel

 Add the Chat Completions Service and Embeddings Generator Service

 Setup a Volatile Memory Store

 Setup the Memory and add it to the Store

 Add memories to the memory

 Simple, right?

 ~

 A Problem

 I had been reading and following Arafat Tehsin’s fantastic blog whilst working on this and then encountered a problem.

 Specifically, on the final step (adding memories to the memory).

 You can see this here:

 After some digging, it turns out, there is an error Semantic Kernel ONNX Connector when you try to use Local Embeddings.

 Specifically, the line .AddLocalTextEmbeddingGeneration(); when setting up the kernel.

 I shared the screenshot with Arafat who raised in issue GitHub (thankyou!) with Microsoft which is currently being investigated.

 ~

 The Solution

 A temporary solution to the issue above is to manually load the embeddings capability -using a different model.    Shout to Jose Luis Latorre Millas  and David Puplava for this.

 You manually add the local embeddings model by using the following code:

 var textModelPath = @"E:\Source\Models\bge-micro-v2\onnx\model.onnx";

var bgaVocab= @"E:\Source\Models\bge-micro-v2\vocab.txt";

// Load the model and services
var builder = Kernel.CreateBuilder();

builder.AddBertOnnxTextEmbeddingGeneration(textModelPath, bgaVocab);

var kernel = builder.Build();

// Create services such as chatCompletionService and embeddingGeneration
var embeddingGenerator = kernel.GetRequiredService<ITextEmbeddingGenerationService>(); 

 For reference, we have the entire code listing:

 var modelPath = @"Models\\Phi-3-mini-4k-instruct-onnx\\cpu_and_mobile\\cpu-int4-rtn-block-32-acc-level-4";

    var modelId = "localphi3onnx";
    var textModelPath = @"\Models\bge-micro-v2\onnx\model.onnx";
    var bgaVocab= @"E:\Source\Models\bge-micro-v2\vocab.txt";

    var builder = Kernel.CreateBuilder();

    builder.AddOnnxRuntimeGenAIChatCompletion(modelId, modelPath);
    builder.AddBertOnnxTextEmbeddingGeneration(textModelPath, bgaVocab);

    var kernel = builder.Build();

    // Create services such as chatCompletionService and embeddingGeneration
    var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();
    var embeddingGenerator = kernel.GetRequiredService<ITextEmbeddingGenerationService>();

    // Setup a memory store and create a memory out of it
    var memoryStore = new VolatileMemoryStore();
    var memory = new SemanticTextMemory(memoryStore, embeddingGenerator);

    // Loading it for Save, Recall and other methods
    kernel.ImportPluginFromObject(new TextMemoryPlugin(memory));
    string MemoryCollectionName = "MyCustomDataCollection";

    // Start the conversation
    while (true)
    {
        // Get user input
        Console.Write("User > ");
        
 var question = Console.ReadLine()!;

        // Enable auto function calling
        OpenAIPromptExecutionSettings executionSettings = new()
        {
            ToolCallBehavior = ToolCallBehavior.EnableKernelFunctions,
            MaxTokens = 1000
        };

        // Invoke the kernel with the user input
        var response = kernel.InvokePromptStreamingAsync(

            promptTemplate: @"in as few words as possible, answer this Question: {{$input}}
                             Answer the question using the memory content: {{Recall}}",
            arguments: new KernelArguments(executionSettings)
            {
                { "input", question },
                { "collection", MemoryCollectionName }
            }
            );

        Console.Write("\nAssistant > ");

        string combinedResponse = string.Empty;
        await foreach (var message in response)
        {
            //Write the response to the console
            Console.Write(message);
            combinedResponse += message;
        }
        Console.WriteLine();
    } 
 ~

 Demo – Interacting with the Agent and Reading from Memory

 Here, we supply the prompt: User > who developed c#? .  The agent replies with the text: Microsoft developed C# in 2000 : 

 We can check this is accurate by looking at the custom data that was saved to the agent memory:

 public static IEnumerable<CodingFact> GetCustomFacts()
  {
      var facts = new CodingFact[]
      {
      new("C# was developed by Microsoft and released in 2000.", "C# History", "Developer: Microsoft"),

      new(".NET Framework, supporting C#, is an open-source development platform.", "Platform", ".NET: Open-source"),

      new("C# is a statically-typed language.", "Typing", "Type System: Statically-Typed"),

      new("C# supports object-oriented and component-oriented programming.", "Programming Paradigms", "Paradigms: OOP, COP"),

      new("LINQ in C# allows querying data in a declarative manner.", "LINQ", "Feature: Declarative Queries"),

      new("C# has built-in garbage collection for automatic memory management.", "Memory Management", "Feature: Garbage Collection"),

      new("The async and await keywords in C# are used for asynchronous programming.", "Asynchronous Programming", "Keywords: async, await"),

      new("C# supports generics for type-safe data structures.", "Generics", "Feature: Type-Safety"),

      new("Delegates in C# are type-safe and similar to function pointers in C++.", "Delegates", "Comparison: Function Pointers"),

      new("C# uses try, catch, and finally blocks for exception handling.", "Exception Handling", "Keywords: try, catch, finally")

      };

      return facts;
  } 

 To further verify the agent is leveraging in-memory content (embedded as vectors), we can ask it something that will not be in the training set.

 We can ask the agent, does Jamie Maguire have a blog? :

 The agent has no knowledge of this.

 We can add this to the agents memory however:

 new("Jamie Maguire does indeed have a blog. You can find it at www.jamiemaguire.net.","Jamie Maguire Blog", "URL: www.jamiemaguire.net ") 

 After rerunning the agent, we get a response that answers the prompt with accurate information:

 Perfect.

 ~

 Shout Outs

 Shout out to the following people in helping arrive at this solution:

 Bruno Capuano – for his original blog.

 Arafat Tehsin – for his original blog

 Jose Luis Latorre Millas – for identifying the local embeddings fix

 David Puplava – for identifying the local embeddings fix

 ~

 Summary

 In this blog post, we’ve seen how to implement 100% local RAG using Phi-3 and Local Embeddings.

 We’ve saw the agent in action.

 In a future blog post, we will explore working with agent memory in more detail.  This an evolving area with several options.

 ~

 Further Reading and Resources

 You can learn more about Phi-3 and the ONNX Runtime here:

 ONNX Runtime – ONNX Runtime | Home 

 Phi-3 Download from Huggingface – microsoft/Phi-3-mini-4k-instruct-onnx 

 Phi-3 Stats – Enjoy the Power of Phi-3 with ONNX Runtime on your device 

 Enjoy what you’ve read, have questions about this content, or would like to see another topic? Drop me a note below.

 You can schedule a call using my  Calendly link  to discuss consulting and development services.
