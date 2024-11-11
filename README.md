**Simplified Chinese | [English](./README.en.md)**

## This is a Simple .NET Implementation Inspired by GraphRag

Based on Microsoft’s approach in the referenced paper, the execution process in GraphRAG mainly achieves the following functions:
- **Source Documents → Text Chunks**: Splits source documents into text chunks.
- **Text Chunks → Element Instances**: Extracts graph nodes and edge instances from each text chunk.
- **Element Instances → Element Summaries**: Generates summaries for each graph element.
- **Element Summaries → Graph Communities**: Uses a community detection algorithm to divide the graph into communities.
- **Graph Communities → Community Summaries**: Generates summaries for each community.
- **Community Summaries → Community Answers → Global Answer**: Generates local answers from community summaries, then aggregates these to create a global answer.

This project serves as a demo for learning GraphRAG concepts.

## You can directly reference the NuGet package in the project or use the API service provided by this project.

For convenience, the LLM interface currently only supports OpenAI’s specification; other large models can consider using one-API integration products.

Configure in `appsettings.json`:

```json
 "GraphOpenAI": {
   "Key": "sk-xxx",
   "EndPoint": "https://api.antsk.cn/",
   "ChatModel": "gpt-4o-mini",
   "EmbeddingModel": "text-embedding-ada-002"
 },
"TextChunker": {
    "LinesToken": 100,
    "ParagraphsToken": 1000
},
"GraphDBConnection": {
    "DbType": "Sqlite", // PostgreSQL
    "DBConnection": "Data Source=graph.db",
    "VectorConnection": "graphmem.db", // Same as DBConnection for PostgreSQL
    "VectorSize": 1536 // Required for PostgreSQL; optional for SQLite
},
"GraphSearch": {
    "SearchMinRelevance": 0.5, // Minimum search relevance
    "SearchLimit": 3, // Limit on vector search nodes
    "NodeDepth": 3, // Node retrieval depth
    "MaxNodes": 100 // Maximum node count
},
"GraphSys": {
    "RetryCount": 2 // Retry count; increase to improve compatibility with domestic models
}
```

## Start the Project

```bash
dotnet run --project GraphRag.Net.Web.csproj
```

After starting the project, access Swagger to view APIs:

```
http://localhost:5000/swagger
```

![Graph](https://github.com/xuzeyu91/GraphRag.Net/blob/main/doc/api.png)

Alternatively, you can access the UI:

```
http://localhost:5000/
```

Open the Blazor UI to import text, upload files, engage in Q&A, and view the knowledge graph.

![Graph](https://github.com/xuzeyu91/GraphRag.Net/blob/main/doc/graph1.png)

## Using the NuGet Package

```bash
dotnet add package GraphRag.Net
```

To facilitate prompt adjustments, copy the `graphPlugins` directory from the `GraphRag.Net.Web` project and set it up as follows:
[graphPlugins](https://github.com/AIDotNet/GraphRag.Net/tree/main/src/GraphRag.Net.Web/graphPlugins)

```xml
<ItemGroup>
    <None Include="graphPlugins\**">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
</ItemGroup>
```

### Default Configuration with OpenAI Standard Interface

After adding the package, configure the settings file and dependency injection:

```csharp
// OpenAI Configuration
builder.Configuration.GetSection("GraphOpenAI").Get<GraphOpenAIOption>();
// Text Chunker Configuration
builder.Configuration.GetSection("TextChunker").Get<TextChunkerOption>();
// Database Connection
builder.Configuration.GetSection("GraphDBConnection").Get<GraphDBConnectionOption>();
// System Settings
builder.Configuration.GetSection("GraphSys").Get<GraphSysOption>();

// Inject AddGraphRagNet, inject the configuration file first, then GraphRagNet
builder.Services.AddGraphRagNet();
```

### For Other Models

```csharp
var kernelBuild = Kernel.CreateBuilder();
kernelBuild.Services.AddKeyedSingleton<ITextGenerationService>("mock-text", new MockTextCompletion());
kernelBuild.Services.AddKeyedSingleton<IChatCompletionService>("mock-chat", new MockChatCompletion());
kernelBuild.Services.AddSingleton((ITextEmbeddingGenerationService)new MockTextEmbeddingGeneratorService());
kernelBuild.Services.AddKeyedSingleton("mock-embedding", new MockTextEmbeddingGeneratorService());

builder.Services.AddGraphRagNet(kernelBuild.Build());
```

### Inject `IGraphService` Service

Example:

```csharp
namespace GraphRag.Net.Api.Controllers
{
    [Route("api/[controller]/[action]")]
    [ApiController]
    public class GraphDemoController(IGraphService _graphService) : ControllerBase
    {
        [HttpGet]
        public async Task<IActionResult> GetAllIndex() => Ok(_graphService.GetAllIndex());

        [HttpGet]
        public async Task<IActionResult> GetAllGraphs(string index)
        {
            if (string.IsNullOrEmpty(index)) return Ok(new GraphViewModel());
            return Ok(_graphService.GetAllGraphs(index));
        }

        [HttpPost]
        public async Task<IActionResult> InsertGraphData(InputModel model)
        {
            await _graphService.InsertGraphDataAsync(model.Index, model.Input);
            return Ok();
        }

        [HttpPost]
        public async Task<IActionResult> SearchGraph(InputModel model)
        {
            var result = await _graphService.SearchGraphAsync(model.Index, model.Input);
            return Ok(result);
        }

        [HttpPost]
        public async Task<IActionResult> SearchGraphCommunity(InputModel model)
        {
            var result = await _graphService.SearchGraphCommunityAsync(model.Index, model.Input);
            return Ok(result);
        }

        [HttpPost]
        public async Task<IActionResult> ImportTxt(string index, IFormFile file)
        {
            var forms = await Request.ReadFormAsync();
            using var stream = new StreamReader(file.OpenReadStream());
            var txt = await stream.ReadToEndAsync();
            await _graphService.InsertTextChunkAsync(index, txt);
            return Ok();
        }

        [HttpGet]
        public async Task<IActionResult> GraphCommunities(string index)
        {
            await _graphService.GraphCommunitiesAsync(index);
            return Ok();
        }

        [HttpGet]
        public async Task<IActionResult> GraphGlobal(string index)
        {
            await _graphService.GraphGlobalAsync(index);
            return Ok();
        }

        [HttpGet]
        public async Task<IActionResult> DeleteGraph(string index)
        {
            await _graphService.DeleteGraph(index);
            return Ok();
        }
    }

    public class InputModel
    {
        public string Index { get; set; }
        public string Input { get; set; }
    }
}
```

## Test Database

Pretrained data can be downloaded and placed in the project directory for testing:

```
https://pan.quark.cn/s/bf2d21f29f85
```

For more RAG scenarios, visit **AntSK**:
[Project Link](https://github.com/AIDotNet/AntSK)

Demo Environment:

[Demo URL](https://demo.antsk.cn)

Account: test  
Password: test

Feel free to join our WeChat group by adding **xuzeyu91** on WeChat and sending a request to join.