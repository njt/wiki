---
url: https://github.com/UPwith-me/Container-Maker
date_fetched: 2026-07-05
backfilled: true
---

**Quick Start** •
    **Features** •
    **Cloud** •
    **Commands** •
    **中文文档**

**Container-Maker (cm)** fuses the speed of Makefiles, the isolation of Docker, and the intelligence of VS Code DevContainers into a single, zero-configuration CLI that transforms any machine into a production-grade development studio.

- What is Container-Maker?
- Quick Start
- Core Features
- Advanced Features
- Cloud Control Plane
- TUI Dashboard
- Template Library
- Command Reference
- Configuration Reference
- Clever Design Details
- Security
- FAQ
- Contributing
- License

Container-Maker solves the **"works on my machine"** problem once and for all while delivering a seamless developer experience.

| Your  | 
 | Leverages Docker BuildKit for aggressive layer caching. Your environment spins up in seconds, not minutes. | 

| Feature | Docker CLI | VS Code DevContainers | Container-Maker | 
|---|---|---|---|
| Zero-config startup | ❌ | ✅ | |
| Standalone CLI | ✅ | ❌ | ✅ | 
| AI config generation | ❌ | ❌ | ✅ | 
| Cloud deployment | ❌ | ❌ | ✅ | 
| TUI dashboard | ❌ | ❌ | ✅ | 
| Template marketplace | ❌ | ✅ | |
| Multi-runtime support | ✅ Docker/Podman | 

**🏭 Industrial Grade Capabilities**

**Plugins & Extensibility**:

```
cm plugin install https://example.com/cm-my-plugin   # Install custom commands
cm scan                                              # Vulnerability scanning (Trivy)
```
**Environment Snapshots**:

```
cm snapshot create "stable-v1" "Before upgrade"      # Save exact state
cm snapshot restore "stable-v1"                      # Rollback instantly
```
**Offline Delivery (Air-Gap)**:

```
cm export project.cm                                 # Package image + config + code
cm load project.cm                                   # Restore completely offline
```
**🌐 Multi-Language Project Support**

Clone any project and get a development environment supporting ALL detected languages:

```
cm clone https://github.com/example/fullstack-app
🔍 Detection Results:
   Primary Language: Python
   All Languages: Python, JavaScript, Go
   Frameworks: FastAPI, React
🌐 Multi-language project detected!
   Generating config with all language support...
```
Generates a single `devcontainer.json` with:

- All language runtimes via devcontainer features
- Language-specific VS Code extensions
- Combined post-create commands
- Framework-aware port forwarding

**Supported Languages**: Go, Python, JavaScript, TypeScript, Rust, Java, C++, .NET/C#, Ruby, PHP

**🤖 AI Enhancements**

**Build Debugging**:

```
cm ai debug                  # Diagnose build failures
cm ai debug build.log        # Analyze specific log
```
**Config Optimization**:

```
cm ai optimize              # Analyze and suggest improvements
cm ai optimize --apply      # Auto-apply security/performance fixes
```
**Local AI (No API Key Required)**:

```
cm ai local generate        # Use Ollama for generation
cm ai local models          # List local models
cm ai local pull codellama  # Pull code-optimized model
```
**🌍 Remote Development Improvements**

**Dynamic Container Discovery**:

```
cm remote shell                # Auto-detect container
cm remote shell -c my-app      # Specify container
```
**Port Forwarding**:

```
cm remote forward 8080         # Forward single port
cm remote forward --auto       # Auto-detect ports
```
**Docker Context Support**:

`cm remote context myserver     # Use native Docker context`| Feature | Status | Description | 
|---|---|---|
| `cm clone` | ✅ Stable | Clone repos with auto-detection | 
| `cm shell/run` | ✅ Stable | Container execution | 
| `cm ai generate` | ✅ Stable | AI config generation | 
| `cm ai debug` | ✅ Stable | AI-powered debugging | 
| `cm ai optimize` | ✅ Stable | Config optimization | 
| `cm remote` | ✅ Stable | SSH remote development | 
| `cm env` | ✅ Stable | Multi-environment management | 
| `cm workspace` | ✅ Stable | Multi-service orchestration | 
| `cm monitor` | ✅ Stable | TUI dashboard | 
| `cm snapshot` | ✅ Stable | Environment version control | 
| `cm export` | ✅ Stable | Offline bundle export | 
| `cm profile` | ✅ Stable | Resource usage profiling | 
| `cm plugin` | Beta | Extension system | 
| `cm scan` | Beta | Security vulnerability scanning | 
| `cm cloud` | Beta | Cloud deployment (providers in development) | 
| `cm marketplace` | Beta | Community templates | 


Note: Features marked asBetaare functional but may have limited provider support or require additional configuration.

```
# Windows (PowerShell)
irm https://github.com/UPwith-me/Container-Maker/releases/latest/download/cm-windows-amd64.exe -OutFile cm.exe
# Linux / macOS
curl -Lo cm https://github.com/UPwith-me/Container-Maker/releases/latest/download/cm-linux-amd64
chmod +x cm && sudo mv cm /usr/local/bin/
```
`go install github.com/UPwith-me/Container-Maker/cmd/cm@latest````
git clone https://github.com/UPwith-me/Container-Maker.git
cd Container-Maker
go build -o cm ./cmd/cm
```
```
# Step 1: Auto-detect and install Docker/Podman
cm setup
# Step 2: Initialize a new project with specific template
cm init --template python
# Step 3: Enter the container
cm shell
# Step 4: Run commands
cm run python main.py
# Step 5: Open in VS Code
cm code
```
Automatically detects your OS and installs the optimal container runtime.

`cm setup`- **Windows**: Installs Docker Desktop or WSL2 + Docker
- **Linux**: Installs Docker CE or Podman
- **macOS**: Installs Docker Desktop or Colima

Deep health checks for your development environment.

`cm doctor`Checks include:

- ✅ Container runtime (Docker/Podman)
- ✅ GPU support (NVIDIA/AMD)
- ✅ Network connectivity
- ✅ Disk space
- ✅ Docker Compose availability

Create new projects from curated templates or let AI generate configurations.

```
# Interactive mode
cm init
# Use a specific template
cm init --template pytorch
# AI-powered generation
cm ai generate
```
Multiple ways to interact with your container:

| Command | Description | Use Case | 
|---|---|---|
| `cm shell` | Start persistent container and attach | Interactive development | 
| `cm run <cmd>` | Run command in ephemeral container | One-off builds | 
| `cm exec <cmd>` | Execute in running container | Hot-reload scenarios | 

```
# Start a shell session
cm shell
# Run tests
cm run pytest tests/
# Execute in background container
cm exec npm run build
```
Let AI analyze your project and generate optimized configurations.

`cm ai generate`- Analyzes `package.json`,`requirements.txt`,`go.mod`, etc.
- Suggests optimal base images
- Configures caching strategies
- Adds appropriate VS Code extensions

Browse and install community templates.

```
# Search templates
cm marketplace search pytorch
# List GPU-accelerated templates
cm marketplace search --gpu
# Install a template
cm marketplace install ml-pytorch
```
Generate "One-Click Onboarding" links for your team.

```
# Generate a universal clone command
cm share
# Markdown badge for your README
cm share --format markdown
```
Outputs a command like: `cm clone https://github.com/org/repo`. This single command will clone the repo, detect the configuration, and enter the development environment instantly.

Open your project in VS Code with full DevContainer support.

`cm code`- Automatically detects `devcontainer.json`
- Launches VS Code with Remote-Containers
- Works with local and remote containers

Orchestrate complex microservices architectures. `cm` manages the entire lifecycle, including dependency resolution, shared networks, and volume persistence.

**Commands:**

- `cm workspace init`: Initialize a new workspace or add services.
- `cm workspace graph`: Visualize the dependency tree (Topological Sort).
- `cm workspace services`: List all configured services.
- `cm workspace validate`: Check configuration integrity.

**Lifecycle Control:**

- `cm up`: Start all services in dependency order.
- `cm down`: Stop and remove services.
- `cm restart <service>`: Hot-reload a specific service.
- `cm logs <service>`: Stream unified or service-specific logs.
- `cm ps`: View running processes across the workspace.

```
# cm-workspace.yaml example
workspace:
  name: my-app
  network: bridge
services:
  backend:
    path: ./backend
    exposed_ports: ["8080:8080"]
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
```
Migrate existing projects seamlessly. The import engine parses `docker-compose.yml`, performs compatibility analysis, and generates a native CM configuration.

**Workflow:**

- **Analyze**:- `cm import analyze docker-compose.yml`- Generates a report of supported/unsupported features.
- Checks port conflicts and volume bindings.
 
- **Import**:- `cm import docker-compose.yml --dry-run`- Preview the conversion result.
 
- **Execute**:- `cm import docker-compose.yml`- create `cm-workspace.yaml`and service definitions.
 
- create 

Supported conversions: Services, Ports, Volumes, Networks, Environment variables.

Install additional tools from OCI registries with smart local caching.

```
# Add Go to your container
cm feature add ghcr.io/devcontainers/features/go
# Download and cache a feature locally
cm feature download node
# Manage local cache
cm feature cache
cm feature cache clear
```
Seamlessly works with `docker-compose.yml`:

```
{
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace"
}
```
Automatic persistent caching for major languages:

| Language | Cache Path | Speed Improvement | 
|---|---|---|
| Go | `/go/pkg/mod` | Up to 10x | 
| Node.js | `node_modules` | Up to 5x | 
| Rust | `/usr/local/cargo` | Up to 8x | 
| Python | `~/.cache/pip` | Up to 3x | 
| Java | `~/.m2` | Up to 4x | 

Automatic detection and forwarding:

```
{
  "forwardPorts": [3000, 8080, "5432:5432"]
}
```
Supports:

- Single ports: `3000`
- Port ranges: `8000-8010`
- Mappings: `"host:container"`

Auto-run commands on file changes:

```
# Watch and run tests
cm watch --run "pytest tests/"
# Watch with custom patterns
cm watch --pattern "*.py" --run "python main.py"
```
Proactive security warnings:

`cm doctor --security`Detects:

- ⚠️ Docker socket mounts
- ⚠️ Privileged mode
- ⚠️ Sensitive environment variables
- ✅ Suggests Rootless Docker alternatives

First-class support for AI/ML development. Container-Maker includes a native scheduler to managing GPU resources.

**Features:**

- **Auto-Detection**: Zero-config detection of NVIDIA GPUs via- `nvidia-smi`.
- **Hardware Monitoring**: Real-time tracking of VRAM, Temperature, Power Draw, and Utilization.
- **Intelligent Scheduling**:- *Exclusive Mode*: Lock a GPU for a single high-priority task.
- *Shared Mode*: Allow multiple services to share VRAM.
- *Strategy*: First-fit allocation based on VRAM requirements.
 

```
# List physical GPUs
cm gpu list
# Detailed monitoring dashboard
cm gpu status
# Request resources (blocks until available)
cm gpu allocate training-job --count 2 --vram 16G
```
Accelerate frontend and microservice development by mocking upstream dependencies.

**Capabilities:**

- **Dynamic Reponses**: Use Go templates to generate JSON based on requests.
- **Latency Injection**: Simulate network jitter and timeouts to test resilience.
- **Contract Verification**: Validate that your real service matches the API contract (OpenAPI/YAML).
- **Quick Mock**: Spin up an instant endpoint without config.

```
# Instant mock
cm mock quick /api/status '{"status": "ok"}'
# Full server
cm mock serve mocks.yaml
# Verify contract compliance
cm mock verify --contract api-contract.yaml --url http://localhost:8080
```
Enforce organizational standards using a Rego-inspired policy engine.

**Built-in Policies:**

- `SEC-001`🚨- **Critical**: No Privileged Containers.
- `SEC-002`- ⚠️ - **High**: Non-root user enforcement.
- `RES-001`- ⚠️ - **Medium**: Memory/CPU limits must be defined.
- `BP-001`ℹ️- **Info**: Healthchecks required.

```
cm policy check --strict  # Fail CI pipeline on violation
cm policy list            # View active rules
```
Secure your supply chain by generating Software Bills of Materials (CycloneDX JSON).

**Supported Parsers:**

- 🟢 Go (`go.mod`)
- 🟡 Node.js (`package.json`)
- 🔵 Python (`requirements.txt`)

`cm sbom -o sbom.json`Extend Container-Maker with custom behavior.

- **Lifecycle Hooks**: Inject logic- `PreStart`/- `PostStart`(e.g., for specialized auditing or setup).
- **Custom Auditing**: Built-in audit plugin tracks workspace events.

Capture the exact state of your container (files + memory + processes) and roll back anytime.

```
cm snapshot create "feature-wip"
cm snapshot list
cm snapshot restore "feature-wip"
```
AI-driven resource optimization. Analyzes container usage and suggests P95-based limits.

```
cm profile start
# ... run your workload ...
cm profile stop --report
# Output: "Recommended: CPU: 2.0, Memory: 512MB"
```
Scan your dev container for CVEs using Trivy integration.

```
cm scan
cm scan --severity CRITICAL
```
Bundle your entire environment (Image + Config + Code) into a single file for air-gapped delivery.

```
cm export my-env.cm
# Transfer my-env.cm to offline machine
cm load my-env.cm
```
Manage global settings for behavior, updates, and plugins.

```
cm config set update.check true
cm config get ai.provider
```
Connect to remote machines and sync files seamlessly:

```
# Add a remote host
cm remote add myserver user@192.168.1.100
# List configured remotes
cm remote list
# Test connection
cm remote test myserver
# Set active remote
cm remote use myserver
# Open shell on remote container
cm remote shell
```
**File Synchronization:**

```
# Start continuous sync (local → remote)
cm remote sync start myserver
# One-time push to remote
cm remote sync push
# Pull from remote
cm remote sync pull
```
Container-Maker Cloud extends your local development to the cloud with on-demand GPU instances.

- **One-Click GPU Access**: Provision NVIDIA T4, A10, A100 instances
- **14+ Cloud Providers**: AWS, GCP, Azure, DigitalOcean, and more
- **Pay-as-you-go**: No upfront costs, billed by the second
- **Seamless CLI Integration**:- `cm cloud`commands

| Provider | GPU Support | Regions | 
|---|---|---|
| AWS EC2 | ✅ | 25+ | 
| Google Cloud | ✅ | 35+ | 
| Azure | ✅ | 60+ | 
| DigitalOcean | ❌ | 14 | 
| Hetzner | ❌ | 5 | 
| Linode | ✅ | 11 | 
| Vultr | ✅ | 25 | 
| OCI (Oracle) | ✅ | 41 | 
| Lambda Labs | ✅ | 5 | 
| RunPod | ✅ | 10+ | 
| Vast.ai | ✅ | Community | 
| Paperspace | ✅ | 6 | 
| CoreWeave | ✅ | 3 | 
| Docker (Local) | ✅ | - | 

```
# Login to cloud
cm cloud login
# List available instances
cm cloud instances
# Create a GPU instance
cm cloud create --provider aws --type gpu-t4 --name ml-training
# Connect via SSH
cm cloud connect <instance-id>
# Stop instance
cm cloud stop <instance-id>
# Delete instance
cm cloud delete <instance-id>
```
Access the full-featured web dashboard:

```
# Start the local dashboard
cm cloud dashboard
# Or access the hosted version
# https://cloud.container-maker.dev
```
Features:

- Real-time instance monitoring
- WebSocket-based log streaming
- Interactive terminal
- Usage analytics and billing

```
# Billing & Usage
cm cloud billing
# List supported providers
cm cloud providers
```
Beautiful terminal UI for monitoring your containers.

`cm status`Or simply run `cm` without arguments to launch the home screen.

Features:

- Container list with status
- Resource usage (CPU/Memory)
- Log streaming
- Quick actions (Start/Stop/Delete)

30+ curated templates for various use cases:

| Template | Description | 
|---|---|
| `pytorch` | PyTorch with CUDA support | 
| `tensorflow` | TensorFlow 2.x with GPU | 
| `huggingface` | Transformers + Datasets | 
| `jupyter` | JupyterLab with scientific stack | 

| Template | Description | 
|---|---|
| `miniconda` | Conda/Anaconda data science | 
| `python-poetry` | Poetry modern package manager | 
| `python-pipenv` | Pipenv virtual environments | 
| `cpp-conan` | C++ with Conan package manager | 
| `cpp-vcpkg` | C++ with Vcpkg libraries | 
| `cpp-cmake` | C++ CMake projects | 
| `java-maven` | Java Maven projects | 
| `java-gradle` | Java Gradle projects | 
| `dotnet` | .NET 8.0 development | 
| `php-composer` | PHP with Composer | 

| Template | Description | 
|---|---|
| `node` | Node.js 20 LTS | 
| `react` | React + Vite | 
| `nextjs` | Next.js 14 | 
| `python-web` | FastAPI / Django | 

| Template | Description | 
|---|---|
| `go` | Go 1.21+ | 
| `rust` | Rust + Cargo | 
| `cpp` | C++ with CMake | 

| Template | Description | 
|---|---|
| `terraform` | Terraform + Cloud CLIs | 
| `kubernetes` | kubectl + Helm | 
| `ansible` | Ansible + Python | 

| Command | Description | Example | 
|---|---|---|
| `cm` | Launch TUI home screen | `cm` | 
| `cm init` | Initialize new project | `cm init --template python` | 
| `cm shell` | Enter persistent container | `cm shell` | 
| `cm run <cmd>` | Run command in container | `cm run make build` | 
| `cm exec <cmd>` | Execute in running container | `cm exec npm test` | 
| `cm prepare` | Build container image | `cm prepare` | 

| Command | Description | Example | 
|---|---|---|
| `cm setup` | Install container runtime | `cm setup` | 
| `cm doctor` | Run diagnostics | `cm doctor` | 
| `cm status` | Show TUI dashboard | `cm status` | 
| `cm code` | Open in VS Code | `cm code` | 

| Command | Description | Example | 
|---|---|---|
| `cm ai generate` | AI-generate config | `cm ai generate` | 
| `cm snapshot` | Manage snapshots | `cm snapshot create` | 
| `cm profile` | Profile resources | `cm profile start` | 
| `cm scan` | Scan vulnerabilities | `cm scan` | 
| `cm plugin` | Manage plugins | `cm plugin list` | 
| `cm export/load` | Offline Support | `cm export` | 
| `cm marketplace search` | Search templates | `cm marketplace search --gpu` | 
| `cm marketplace install` | Install template | `cm marketplace install pytorch` | 
| `cm template list` | List local templates | `cm template list` | 

| Command | Description | Example | 
|---|---|---|
| `cm cloud login` | Authenticate | `cm cloud login` | 
| `cm cloud instances` | List instances | `cm cloud instances` | 
| `cm cloud create` | Create instance | `cm cloud create --type gpu-t4` | 
| `cm cloud connect` | SSH into instance | `cm cloud connect abc123` | 
| `cm cloud stop` | Stop instance | `cm cloud stop abc123` | 
| `cm cloud delete` | Delete instance | `cm cloud delete abc123` | 

| Command | Description | Example | 
|---|---|---|
| `cm feature add` | Add OCI feature | `cm feature add ghcr.io/devcontainers/features/go` | 
| `cm feature list` | List features | `cm feature list` | 
| `cm cache clean` | Clear build cache | `cm cache clean` | 
| `cm watch` | Watch file changes | `cm watch --run "pytest"` | 
| `cm backend` | Manage runtimes | `cm backend list` | 
| `cm clone` | Clone + enter container | `cm clone github.com/user/repo` | 
| `cm share` | Generate shareable link | `cm share --format markdown` | 
| `cm images` | Manage preset images | `cm images list` | 
| `cm make` | Run Makefile targets | `cm make build` | 

| Command | Description | Example | 
|---|---|---|
| `cm workspace` | Manage workspaces | `cm workspace graph` | 
| `cm up/down` | Start/Stop workspace | `cm up -d` | 
| `cm import` | Import Docker Compose | `cm import docker-compose.yml` | 
| `cm gpu` | Manage GPU resources | `cm gpu status` | 
| `cm mock` | Mock services | `cm mock serve` | 
| `cm policy` | Enforce policies | `cm policy check` | 
| `cm sbom` | Generate SBOM | `cm sbom` | 
| `cm plugin` | Manage plugins | `cm plugin list` | 

| Command | Description | Example | 
|---|---|---|
| `cm remote add` | Add remote host | `cm remote add server user@host` | 
| `cm remote list` | List remote hosts | `cm remote list` | 
| `cm remote use` | Set active remote | `cm remote use server` | 
| `cm remote test` | Test connection | `cm remote test server` | 
| `cm remote shell` | Shell on remote | `cm remote shell` | 
| `cm remote sync start` | Start file sync | `cm remote sync start` | 
| `cm remote sync push` | Push to remote | `cm remote sync push` | 
| `cm remote sync pull` | Pull from remote | `cm remote sync pull` | 

Manage enterprise configuration, shared templates, and audit logs.

| Command | Description | Example | 
|---|---|---|
| Repo Management | ||
| `cm team add` | Add template repo | `cm team add https://github.com/org/tmpls` | 
| `cm team remove` | Remove repo | `cm team remove my-repo` | 
| `cm team sync` | Sync/Update repos | `cm team sync` | 
| `cm team list` | List cached templates | `cm team list` | 
| Enterprise Controls | ||
| `cm team auth` | Configure auth | `cm team auth --token <gh_token>` | 
| `cm team pin` | Pin repo version | `cm team pin hq@v1.2.0` | 
| `cm team unpin` | Unpin repo | `cm team unpin hq` | 
| `cm team var` | Set global variable | `cm team var REGISTRY=harbor.internal` | 
| Auditing | ||
| `cm team log` | View audit log | `cm team log` | 
| `cm team set-audit` | Enable/Disable audit | `cm team set-audit on` | 
| Configuration | ||
| `cm team set` | Set Org Name | `cm team set MyCorp` | 
| `cm team info` | Show configuration | `cm team info` | 

Container-Maker includes several thoughtful touches:

On first run, `cm` offers to add itself to your system PATH and **refreshes your shell session instantly**—no restart required.

```
🚀 Container-Maker detected this is your first run.
   Would you like to add cm to your PATH? [Y/n]
   ✅ Added to PATH. Session refreshed!
```
After modifying environment variables, `cm` automatically refreshes PowerShell/Bash sessions without needing to close and reopen terminals.

Running `cm` without arguments launches an interactive home screen with:

- Project detection
- Quick action menu
- Container status at a glance

Features are downloaded once and cached. Subsequent projects reuse cached layers for instant setup.

`cm` automatically finds `devcontainer.json` in:

- `.devcontainer/devcontainer.json`
- `devcontainer.json`
- `.devcontainer.json`

Fully compatible with Rootless Docker and Podman:

`cm backend switch podman-rootless``cm doctor --security`Detects and warns about:

- Docker socket mounts (`/var/run/docker.sock`)
- Privileged containers
- Sensitive environment variables
- Excessive capabilities

- Use official base images
- Enable Rootless mode when possible
- Avoid mounting Docker socket unless necessary
- Review `runArgs`for security implications

**Q: Does Container-Maker require VS Code?**

No! Container-Maker is a standalone CLI. VS Code integration via `cm code` is optional.

**Q: Can I use Podman instead of Docker?**

Yes! Use `cm backend switch podman` to switch runtimes.

**Q: How do I enable GPU support?**

- Install NVIDIA Container Toolkit
- Run `cm doctor`to verify
- Use GPU-enabled templates: `cm init --template pytorch`

**Q: Where are my files stored inside the container?**

By default, your project directory is mounted at `/workspaces/<project-name>`.

**Q: How do I persist data between container restarts?**

Use named volumes in your mounts configuration, or the built-in caching system.

We welcome contributions!

```
# Fork and clone
git clone https://github.com/UPwith-me/Container-Maker.git
cd Container-Maker
# Build
go build -o cm ./cmd/cm
# Test
go test ./...
```
Container-Maker is dual-licensed under the AGPL-3.0 (for open source use) and a Commercial License (for proprietary/commercial use).

See LICENSE and COMMERCIAL-LICENSE.md for details.

Made with ❤️ by the Container-Maker Team

`{ // Base image or Dockerfile "image": "mcr.microsoft.com/devcontainers/go:1.21", // Or use a Dockerfile "build": { "dockerfile": "Dockerfile", "context": ".", "args": { "VARIANT": "1.21" } }, // Container options "runArgs": ["--cap-add=SYS_PTRACE"], "mounts": ["source=go-mod,target=/go/pkg/mod,type=volume"], "containerEnv": { "CGO_ENABLED": "0" }, // Lifecycle commands "postCreateCommand": "go mod download", "postStartCommand": "echo 'Ready!'", // DevContainer Features "features": { "ghcr.io/devcontainers/features/docker-in-docker:2": {} }, // Port forwarding "forwardPorts": [8080, 3000], // VS Code customizations "customizations": { "vscode": { "extensions": ["golang.go"], "settings": { "go.useLanguageServer": true } } } }`
