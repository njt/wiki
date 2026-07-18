---
url: https://developer.okta.com/blog/2026/07/16/csharp-mcp-cross-app-access
title: Build a Secure C# MCP App with Cross App Access (XAA)
author: Aasawari Sahasrabuddhe
date_fetched: 2026-07-18
date_published: 2026-07-16
---

# Build a Secure C# MCP App with Cross App Access (XAA)

**Author:** Aasawari Sahasrabuddhe — Senior Builder Advocate at Okta
**Publication Date:** July 16, 2026
**Reading Time:** 5 MINUTES
**Source:** Okta Developer Blog

---

## What is Cross App Access (XAA)?

The article frames XAA as an "open standard that securely enables AI agents to act on behalf of a user" when communicating with downstream services, eliminating repeated manual consent. It addresses a gap where a user's identity from an OIDC login isn't trusted by downstream services like MCP servers or APIs.

Two core standard interactions power XAA:

1. **RFC 8693 (Token Exchange)** — The requesting app swaps an OIDC ID token for a JWT Authorization Grant (JAG) at the enterprise IdP.
2. **RFC 7523 (JWT Bearer Grant)** — The app presents that JAG to the MCP authorization server to receive a scoped access token.

The flow diagram shows the complete token exchange sequence across the user, app, IdP, and MCP server.

---

## Implementing XAA with the C# MCP SDK

The SDK's `IdentityAssertionGrantProvider` handles "all the heavy lifting, abstracting these RFCs into a clean, developer-friendly interface."

### Building the OIDC Flow

ASP.NET Core middleware handles authentication via PKCE. Key configuration:

```csharp
builder.Services
    .AddAuthentication()
    .AddCookie(o => { o.Cookie.Name = "xaa.auth"; })
    .AddOpenIdConnect(o =>
    {
        o.Authority    = config["Xaa:IdpBaseUrl"];
        o.ClientId     = config["Xaa:ClientId"];
        o.ResponseType = "code";
        o.UsePkce      = true;
        o.SaveTokens   = true;
        o.MapInboundClaims = false;
        o.Scope.Add("openid");
        o.Scope.Add("email");
    });
```

Setting `MapInboundClaims = false` prevents ASP.NET Core from remapping standard OIDC claim names, keeping "the token payload clean and predictable downstream."

### Automating XAA Token Exchange

The app performs a "two-hop token upgrade on the user's behalf" via the SDK:

```csharp
var provider = new IdentityAssertionGrantProvider(
    new IdentityAssertionGrantProviderOptions
    {
        ClientId         = config["Xaa:McpClientId"]!,
        ClientSecret     = config["Xaa:McpClientSecret"],
        IdpTokenEndpoint = $"{config["Xaa:IdpBaseUrl"]}/token",
        IdpClientId      = config["Xaa:ClientId"]!,
        IdpClientSecret  = config["Xaa:ClientSecret"],
        Scope            = "todos.read mcp.access",
        IdTokenCallback = (_, _) => Task.FromResult(idToken)
    },
    httpClient);

var result = await provider.GetAccessTokenAsync(
    resourceUrl:           new Uri(config["Xaa:McpServerUrl"]!),
    authorizationServerUrl: new Uri(config["Xaa:AuthServerUrl"]!));

var accessToken = result.AccessToken;
```

The IdP returns an ID-JAG — "a short-lived token that grants access to the resource server" — which is then exchanged for an access token.

### Connecting the MCP Client to the Server

```csharp
var transport = new HttpClientTransport(new HttpClientTransportOptions
{
    Endpoint          = new Uri(config["Xaa:McpServerUrl"]!),
    TransportMode     = HttpTransportMode.StreamableHttp,
    AdditionalHeaders = new Dictionary<string, string>
    {
        ["Authorization"] = $"Bearer {accessToken}"
    }
});

await using var client = await McpClient.CreateAsync(transport);
```

`McpClient.CreateAsync` performs the MCP initialization handshake. From there, resource interaction is straightforward:

```csharp
var resources = await client.ListResourcesAsync();
var result    = await client.ReadResourceAsync("todo0://todos");

var raw = string.Join("",
    result.Contents
          .OfType<TextResourceContents>()
          .Select(c => c.Text ?? ""));
```

---

## Testing with xaa.dev

[xaa.dev](https://xaa.dev) serves as "a testing playground" providing "a standardized, functional environment" for verifying the end-to-end flow. Registration steps:

1. Select the **Register, test, and manage your requesting app** tab, then **Continue with your app**.
2. Enter your email, continue, and **Register New App**.
3. Provide Redirect URI and Post-logout URI.
4. Under **Add Resource**, select **ToDo MCP Server**.
5. Click **Register App**.

A screenshot shows the registration screen with redirect URIs, resource connections, and the `todos.read` / `mcp.access` scopes.

---

## Running the App

Clone the repository and populate `appsettings.json`:

```
git clone https://github.com/oktadev/csharp-mcp-sdk-example.git
cd xaa-csharp-mcp-sdk-example
```

Configuration includes `ClientId`, `ClientSecret`, `IdpBaseUrl`, `McpClientId`, `McpClientSecret`, `AuthServerUrl`, `McpServerUrl`, and `RedirectUri`.

Run with `dotnet run`, navigate to `http://localhost:5000/`, sign in with a test email and any 6-digit verification code (since xaa.dev handles the login in demo mode). The dashboard then displays the completed XAA flow steps, access token, claims, and analyzed to-do tasks.

---

## Learn More

The article emphasizes that the entire flow — from SSO login to an AI agent fetching MCP server data — runs "end-to-end in under 50 lines of C#."

Resources provided:

- **Full source code**: [GitHub repository](https://github.com/oktadev/okta-csharp-mcp-sdk-example)
- **XAA playground**: [xaa.dev](https://xaa.dev)
- **Deep dive on enterprise trust gap**: [Integrate Your Enterprise AI Tools with Cross App Access](/blog/2025/06/23/enterprise-ai)

The closing argument: as "AI agents take on more complex, multi-step tasks across organizational boundaries," XAA enables secure cross-app access without sacrificing user experience or security.
