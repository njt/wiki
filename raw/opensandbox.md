---
url: https://github.com/alibaba/OpenSandbox
date_fetched: 2026-07-05
backfilled: true
---

OpenSandbox is a **general-purpose sandbox platform** for AI applications, offering multi-language SDKs, unified sandbox APIs, and Docker/Kubernetes runtimes for scenarios like Coding Agents, GUI Agents, Agent Evaluation, AI Code Execution, and RL Training.

- 🧩 **SDKs, CLI, and MCP**: Provides multi-language SDKs, the osb CLI, and MCP server integration for sandbox creation, command execution, and file operations. See SDKs, CLI, and MCP.
- 📜 **Sandbox Protocol**: Defines sandbox lifecycle management APIs and sandbox execution APIs so you can extend custom sandbox runtimes. See API specs.
- 🚀 **Sandbox Runtime**: Built-in lifecycle management supporting Docker and high-performance Kubernetes runtime, enabling both local runs and large-scale distributed scheduling. See Kubernetes runtime.
- 🖥️ **Sandbox Environments**: Built-in Command, Filesystem, and Code Interpreter implementations. Examples cover Coding Agents (e.g., Claude Code), browser automation (Chrome, Playwright), and desktop environments (VNC, VS Code).
- 🚦 **Network Policy**: Unified ingress gateway with multiple routing strategies plus per-sandbox egress controls. See Ingress Gateway and egress controls.
- 🔑 **Credential Vault**: Secure credential injection for sandbox outbound requests without exposing real secrets to workloads. See Credential Vault.
- 🏰 **Strong Isolation**: Supports secure container runtimes like gVisor, Kata Containers, and Firecracker microVM for enhanced isolation between sandbox workloads and the host. See Secure Container Runtime Guide for details.

Python:

`pip install opensandbox`Java/Kotlin (Gradle Kotlin DSL):

```
dependencies {
    implementation("com.alibaba.opensandbox:sandbox:{latest_version}")
}
```
Java/Kotlin (Maven):

```
<dependency>
    <groupId>com.alibaba.opensandbox</groupId>
    <artifactId>sandbox</artifactId>
    <version>{latest_version}</version>
</dependency>
```
JavaScript/TypeScript:

`npm install @alibaba-group/opensandbox`C#/.NET:

`dotnet add package Alibaba.OpenSandbox`Go:

`go get github.com/alibaba/OpenSandbox/sdks/sandbox/go`OpenSandbox also provides `osb`, a terminal CLI for the common sandbox workflow: create sandboxes, run commands, move files, inspect diagnostics, and manage runtime egress policy.

Install:

```
pip install opensandbox-cli
# or
uv tool install opensandbox-cli
```
Quick start:

```
osb config init
osb config set connection.domain localhost:8080
osb config set connection.protocol http
osb config set connection.api_key <your-api-key>
osb sandbox create --image python:3.12 --timeout 30m -o json
osb command run <sandbox-id> -o raw -- python -c "print(1 + 1)"
```
See the CLI README for the full command reference.

The OpenSandbox MCP server exposes sandbox creation, command execution, and text file operations to MCP-capable clients such as Claude Code and Cursor.

Install and run:

```
pip install opensandbox-mcp
opensandbox-mcp --domain localhost:8080 --protocol http
```
Minimal stdio config:

```
{
  "mcpServers": {
    "opensandbox": {
      "command": "opensandbox-mcp",
      "args": ["--domain", "localhost:8080", "--protocol", "http"]
    }
  }
}
```
See the MCP README for client-specific setup.

Requirements:

- Docker (required for local execution)
- Python 3.10+ (required for examples and local runtime)

```
uvx opensandbox-server init-config ~/.sandbox.toml --example docker
uvx opensandbox-server
# Show help
# uvx opensandbox-server -h
```
Install the Code Interpreter SDK

`uv pip install opensandbox-code-interpreter`Create a sandbox and execute commands and codes.

```
import asyncio
from datetime import timedelta
from code_interpreter import CodeInterpreter, SupportedLanguage
from opensandbox import Sandbox
from opensandbox.models import WriteEntry
async def main() -> None:
    # 1. Create a sandbox
    sandbox = await Sandbox.create(
        "opensandbox/code-interpreter:v1.1.0",
        entrypoint=["/opt/code-interpreter/code-interpreter.sh"],
        env={"PYTHON_VERSION": "3.11"},
        timeout=timedelta(minutes=10),
    )
    async with sandbox:
        # 2. Execute a shell command
        execution = await sandbox.commands.run("echo 'Hello OpenSandbox!'")
        print(execution.logs.stdout[0].text)
        # 3. Write a file
        await sandbox.files.write_files([
            WriteEntry(path="/tmp/hello.txt", data="Hello World", mode=644)
        ])
        # 4. Read a file
        content = await sandbox.files.read_file("/tmp/hello.txt")
        print(f"Content: {content}") # Content: Hello World
        # 5. Create a code interpreter
        interpreter = await CodeInterpreter.create(sandbox)
        # 6. Execute Python code (single-run, pass language directly)
        result = await interpreter.codes.run(
              """
                  import sys
                  print(sys.version)
                  result = 2 + 2
                  result
              """,
              language=SupportedLanguage.PYTHON,
        )
        print(result.result[0].text) # 4
        print(result.logs.stdout[0].text) # 3.11.14
        # 7. Cleanup the sandbox
        await sandbox.kill()
if __name__ == "__main__":
    asyncio.run(main())
```
OpenSandbox provides examples covering SDK usage, agent integrations, browser automation, and training workloads. All example code is located in the `examples/` directory.

- **code-interpreter**- End-to-end Code Interpreter SDK workflow in a sandbox.
- **aio-sandbox**- All-in-One sandbox setup using the OpenSandbox SDK.
- **agent-sandbox**- Example integration for running OpenSandbox workloads on Kubernetes with kubernetes-sigs/agent-sandbox.
- **Volumes**— Docker PVC / named volumes, Docker OSSFS, Kubernetes PVC: persistent and shared storage patterns.

- **Coding CLIs**— Claude Code, Gemini CLI, OpenAI Codex CLI, Qwen Code, Kimi CLI: run each vendor CLI inside OpenSandbox.
- **langgraph**- LangGraph state-machine workflow that creates/runs a sandbox job with fallback retry.
- **google-adk**- Google ADK agent using OpenSandbox tools to write/read files and run commands.
- **openclaw**- Launch an OpenClaw Gateway inside a sandbox.

- **chrome**- Chromium sandbox with VNC and DevTools access for automation and debugging.
- **playwright**- Playwright + Chromium headless scraping and testing example.
- **desktop**- Full desktop environment in a sandbox with VNC access.
- **vscode**- code-server (VS Code Web) running inside a sandbox for remote dev.

- **rl-training**- DQN CartPole training in a sandbox with checkpoints and summary output.
- **harbor-evaluation**- Run a Harbor agent evaluation on OpenSandbox, one sandbox per trial.

For more details, please refer to the examples documentation.

| Directory | Description | 
|---|---|
| `sdks/` | Multi-language SDKs (Python, Java/Kotlin, TypeScript/JavaScript, C#/.NET) | 
| `specs/` | OpenAPI specs and lifecycle specifications | 
| `server/` | Python FastAPI sandbox lifecycle server | 
| `cli/` | OpenSandbox command-line interface | 
| `kubernetes/` | Kubernetes deployment and examples | 
| `components/execd/` | Sandbox execution daemon (commands and file operations) | 
| `components/ingress/` | Sandbox traffic ingress proxy | 
| `components/egress/` | Sandbox network egress control | 
| `sandboxes/` | Runtime sandbox implementations | 
| `examples/` | Runnable example code | 
| `docs/examples/` | Example documentation and use cases | 
| `oseps/` | OpenSandbox Enhancement Proposals | 
| `docs/` | Architecture and design documentation | 
| `tests/` | Cross-component E2E tests | 
| `scripts/` | Development and maintenance scripts | 

For detailed architecture, see Architecture.

- Architecture – Overall architecture & design philosophy
- Credential Vault - Credential Vault credential injection guide
- Release Verification - Release signing and artifact verification
- oseps/README.md – OpenSandbox Enhancement Proposals
- SDK
- Sandbox base SDK (Java/Kotlin SDK, Python SDK, JavaScript/TypeScript SDK, C#/.NET SDK), Go SDK - includes sandbox lifecycle, command execution, file operations
- Code Interpreter SDK (Java/Kotlin SDK, Python SDK, JavaScript/TypeScript SDK, C#/.NET SDK) - code interpreter
 
- cli/README.md - OpenSandbox CLI installation and command reference
- sdks/mcp/sandbox/python/README.md - MCP server installation and client setup
- specs/README.md - OpenAPI definitions for sandbox lifecycle API and sandbox execution API
- server/README.md - Sandbox server startup and configuration; supports Docker and Kubernetes runtimes
- ROADMAP.md - Lightweight project roadmap and planning process

This project is open source under the Apache 2.0 License.

See ROADMAP.md for the current project roadmap, planning scope, and how roadmap items are managed.

- Issues: Submit bugs, feature requests, or design discussions through GitHub Issues
- Discord: Join the OpenSandbox Discord community
- DingTalk: Join the OpenSandbox technical discussion group
