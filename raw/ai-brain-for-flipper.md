---
url: https://github.com/elder-plinius/V3SP3R
date_fetched: 2026-07-05
backfilled: true
---

```
 ██╗   ██╗ ██████╗ ███████╗██████╗ ███████╗██████╗
 ██║   ██║╚════██╗██╔════╝██╔══██╗██╔════╝██╔══██╗
 ██║   ██║ █████╔╝███████╗██████╔╝█████╗  ██████╔╝
 ╚██╗ ██╔╝ ╚═══██╗╚════██║██╔═══╝ ██╔══╝  ██╔══██╗
  ╚████╔╝ ██████╔╝███████║██║     ███████╗██║  ██║
   ╚═══╝  ╚═════╝ ╚══════╝╚═╝     ╚══════╝╚═╝  ╚═╝
```

Talk to your Flipper Zero like it's your partner-in-hacking.Vesper turns your pocket hacking tool into an AI-powered command center — controlled entirely through natural language from your Android device or smart glasses.

No menus. No manuals. Just natural language prompting.

The Flipper Zero is one of the most versatile hardware hacking tools ever made — but navigating its menus, managing files, and crafting signals by hand is tedious. **Vesper eliminates the friction.** Plug in an AI brain via OpenRouter, connect over Bluetooth, and you have a voice-commanded hardware lab in your pocket.

- **Instant expertise**— Don't memorize SubGHz protocols or IR formats. Just say what you want.
- **Real-time control**— The AI reads your Flipper's state, executes commands, and reports back in seconds.
- **Multimodal input**— Voice commands, photo analysis, and text chat. Use your phone camera or smart glasses to show the AI what you're looking at.
- **Signal alchemy**— Build, layer, and export custom RF waveforms with a visual editor.
- **Smart glasses integration**— Pair with Mentra smart glasses for hands-free, heads-up Flipper control.
- **Safety-first architecture**— Every AI action is risk-classified. Destructive operations require explicit confirmation. System paths are locked by default.

Whether you're a security researcher, a red teamer, a CTF competitor, a hardware tinkerer, or just someone who wants to understand the invisible signals around you — Vesper makes the Flipper Zero *dramatically* more accessible and more powerful.

Talk to your Flipper in plain English (voice or text):

- *"Show me my SubGHz captures"*
- *"What's my battery level?"*
- *"Create a backup of all my IR remotes"*
- *"Generate a BadUSB script that opens a reverse shell"*
- *"Change the frequency in garage.sub to 315MHz"*

Direct control over all Flipper subsystems:

- **SubGHz**— Transmit and analyze RF signals
- **Infrared**— Send and record IR commands
- **NFC / RFID / iButton**— Emulate stored credentials
- **BadUSB**— Execute HID attack scripts
- **GPIO / LED / Vibro**— Control hardware peripherals
- **App Launcher**— Start any Flipper app by name

- **Voice input**— Speak commands using on-device speech recognition
- **Photo analysis**— Snap a picture of a remote, a device label, or anything — the AI sees what you see
- **Text-to-speech**— AI responses read aloud via OpenRouter TTS
- **Smart glasses**— Hands-free voice + camera via Mentra glasses bridge

Built for reliability-obsessed users:

- **Pipeline Health**— BLE/RPC/CLI readiness and diagnostics at a glance
- **Runbooks**— One-tap recovery and smoke-test sequences
- **Live Status**— Transport and command pipeline behavior in one view

Build custom RF signals from scratch:

- Visual waveform editor with real-time preview
- Layer and fuse multiple signal patterns
- Export directly to your Flipper's SD card

AI-powered payload generation:

- BadUSB scripts, SubGHz signals, IR remotes, NFC tags
- Validated before deployment — the AI checks format and safety
- Direct push to Flipper storage

Browse and install Flipper applications:

- Search the Flipper app catalog
- One-tap install to your device

Find and download community resources:

- Search GitHub for Flipper-compatible files
- Browse repositories and download directly to your Flipper

Full Flipper visibility:

- Battery, storage, firmware info, and connection status
- Scan, pair, and manage BLE connections
- Direct file browsing and management

Every AI action is classified before execution:

- **Low risk**— Read-only ops execute automatically
- **Medium risk**— File writes show a diff for review
- **High risk**— Destructive ops require double-tap confirmation
- **Blocked**— System/firmware paths require explicit unlock

Configure **auto-approve** per risk tier in Settings to move faster when you trust the workflow.

Every action the AI takes is logged:

- Full history of commands, results, and approvals
- Filterable by action type and session
- Exportable for compliance and review

Vesper works with any model on OpenRouter. For the best experience:

| Model | Why Use It | Speed | Cost | 
|---|---|---|---|
| `nousresearch/hermes-4` | Outstanding tool-use — purpose-built for agentic workflows. Top pick for power users. | Fast | $$ | 
| `anthropic/claude-opus-4.6` | Most capable reasoning model. Exceptional at complex multi-step operations and signal analysis. | Medium | $$$$ | 
| `anthropic/claude-sonnet-4` | Best balance of speed, intelligence, and cost. Great default. | Fast | $$ | 
| `anthropic/claude-haiku-4` | Blazing fast for simple reads and quick commands. | Fastest | $ | 
| `openai/gpt-4o` | Strong general-purpose alternative. | Fast | $$ | 

**Our recommendation:** Start with **Hermes 4** or **Claude Sonnet 4** for daily use. Reach for **Claude Opus 4.6** when you need deep reasoning.

| Item | Notes | 
|---|---|
| Flipper Zero | shop.flipperzero.one | 
| Android device | Android 8.0+ (API 26), Bluetooth required | 
| OpenRouter account | Free signup, pay-per-use — openrouter.ai | 

- Charge it up (USB-C)
- Update firmware via qFlipper (recommended)
- Enable Bluetooth: Settings > Bluetooth > ON

- Sign up at openrouter.ai
- Go to **Keys**>**Create Key**
- Copy the key (`sk-or-...`) — you'll paste this into Vesper
- Add $5-10 in credits to start (most conversations cost pennies)

```
git clone https://github.com/elder-plinius/V3SP3R.git
cd V3SP3R
```
Open the project in Android Studio, let Gradle sync, then:

- **Build > Build APK(s)**or hit the green play button
- APK output: `app/build/outputs/apk/debug/app-debug.apk`

Install via USB debugging or transfer the APK to your phone.

**Command-line build (no Android Studio)**

```
# Requires Android SDK and JDK 17+
./gradlew assembleDebug
# APK: app/build/outputs/apk/debug/app-debug.apk
```
- **Grant permissions**— Bluetooth, Location (required for BLE scanning), Notifications
- **Add your API key**— Settings > paste your OpenRouter key
- **Connect**— Device tab > Scan > tap your Flipper
- **Go**— Chat tab > start talking to your Flipper

Vesper supports hands-free operation via Mentra smart glasses.

- Deploy the bridge server from `mentra-bridge/`:cd mentra-bridge npm install && npm run build && npm start 
- In Vesper Settings, enable **Smart Glasses**and enter the bridge URL
- Voice transcriptions from your glasses flow directly to Vesper
- The AI can request photos through your glasses camera

```
┌─────────────────────────────────────────┐
│              Vesper App                  │
├─────────────────────────────────────────┤
│  UI Layer (Jetpack Compose + Hilt)      │
│  ├── Chat Screen (voice, images, text)  │
│  ├── Ops Center                         │
│  ├── Alchemy Lab & Payload Lab          │
│  ├── File Browser & FapHub              │
│  ├── Signal Arsenal & Spectral Oracle   │
│  └── Device & Settings Screens          │
├─────────────────────────────────────────┤
│  Domain Layer                           │
│  ├── VesperAgent (AI orchestration)     │
│  ├── CommandExecutor (risk enforcement) │
│  ├── RiskAssessor + PermissionService   │
│  ├── ForgeEngine (payload generation)   │
│  ├── DiffService + AuditService         │
│  └── Signal Processing                  │
├─────────────────────────────────────────┤
│  Data Layer                             │
│  ├── OpenRouterClient (LLM API)         │
│  ├── FlipperBleService (BLE transport)  │
│  ├── GlassesIntegration (Mentra bridge) │
│  ├── Room Database (chat + audit)       │
│  └── Encrypted DataStore (settings)     │
└─────────────────────────────────────────┘
```
```
V3SP3R/
├── app/src/main/java/com/vesper/flipper/
│   ├── ai/                     # AI integration
│   │   ├── OpenRouterClient.kt # LLM API, tool calling, JSON repair
│   │   ├── VesperAgent.kt      # Conversation orchestrator
│   │   ├── VesperPrompts.kt    # System prompts
│   │   ├── PayloadEngine.kt    # Payload generation
│   │   └── FlipperToolExecutor.kt
│   ├── ble/                    # Bluetooth
│   │   ├── FlipperBleService.kt
│   │   ├── FlipperProtocol.kt
│   │   ├── FlipperFileSystem.kt
│   │   └── MarauderBridge.kt
│   ├── glasses/                # Smart glasses
│   │   ├── GlassesIntegration.kt
│   │   └── GlassesBridgeClient.kt
│   ├── voice/                  # Voice I/O
│   │   ├── SpeechRecognitionHelper.kt
│   │   └── ElevenLabsTtsService.kt
│   ├── domain/
│   │   ├── executor/           # Command execution & risk
│   │   ├── model/              # Data models
│   │   ├── service/            # Audit, diff, permissions
│   │   └── protocol/           # SubGHz, Pwnagotchi
│   ├── data/                   # Persistence & settings
│   ├── security/               # Input validation, sanitization
│   ├── ui/                     # Jetpack Compose screens
│   │   ├── screen/
│   │   ├── viewmodel/
│   │   ├── components/
│   │   └── theme/
│   └── widget/                 # Home screen widget
├── mentra-bridge/              # Smart glasses bridge server (Node.js)
├── docs/                       # Architecture docs, schemas
└── gradle/                     # Build configuration
```
| Action | Description | Risk Level | 
|---|---|---|
| `list_directory` | List files and folders | Low | 
| `read_file` | Read file contents | Low | 
| `write_file` | Create or modify files | Medium | 
| `create_directory` | Create folders | Low | 
| `delete` | Delete files or folders | High | 
| `move`/`rename`/`copy` | File operations | Medium-High | 
| `get_device_info` | Battery, firmware, hardware info | Low | 
| `get_storage_info` | SD card and internal storage stats | Low | 
| `execute_cli` | Run Flipper CLI commands | Medium | 
| `push_artifact` | Push generated files to Flipper | Medium | 
| `forge_payload` | AI-generate SubGHz/IR/BadUSB/NFC payloads | Medium | 
| `subghz_transmit` | Transmit SubGHz signal | High | 
| `ir_transmit` | Send infrared command | Medium | 
| `nfc_emulate`/`rfid_emulate`/`ibutton_emulate` | Emulate credentials | Medium | 
| `badusb_execute` | Run HID attack script | High | 
| `launch_app` | Start a Flipper app | Medium | 
| `led_control`/`vibro_control` | Hardware peripherals | Low | 
| `search_faphub`/`install_faphub_app` | Browse & install apps | Medium | 
| `browse_repo`/`download_resource`/`github_search` | Find community resources | Low | 
| `request_photo` | Capture photo via smart glasses | Low | 

**Flipper not found when scanning**

- On Flipper: Settings > Bluetooth > make sure it's ON
- Toggle Bluetooth off/on on your phone
- Make sure Flipper isn't connected to another device (e.g. qFlipper)
- Move within 3 feet / 1 meter
- Check that Location permission is granted (required for BLE scanning on Android)

**Build failed in Android Studio**

- Ensure JDK 17+ is installed
- File > Sync Project with Gradle Files
- Build > Clean Project > Rebuild Project
- If still failing: close Android Studio, delete `.gradle`folder, reopen

**AI not responding**

- Verify your OpenRouter API key in Settings
- Check your OpenRouter credit balance at openrouter.ai
- Check internet connection
- Try a different model — some may be temporarily unavailable

**"Could not parse tool arguments" errors**

This means the AI model returned malformed JSON. Vesper includes automatic JSON repair, but some models are more reliable than others. Try:

- Tap **Retry**on the error message
- Switch to a recommended model (Hermes 4, Claude Sonnet 4)
- Simplify your request

**Permission denied errors**

- Some Flipper paths are protected by default (system files, firmware areas)
- Go to Settings > Permissions to unlock specific paths
- Enable auto-approve per risk tier to move faster
- Blocked paths always require manual unlock

Contributions are welcome! Please read CONTRIBUTING.md before submitting a PR.

Areas that need love:

- iOS version (SwiftUI)
- Signal format parsers (new protocols)
- Additional payload templates
- UI/UX improvements
- Translations / i18n
- Test coverage

Found a vulnerability? Please report it responsibly. See SECURITY.md for details.

- Vesper is a tool for **education and legitimate security research**
- Only use on devices you own or have explicit authorization to test
- All AI actions are logged and auditable
- The AI refuses clearly malicious requests
- Destructive operations require explicit user confirmation
- You are responsible for complying with all applicable laws in your jurisdiction

GPL-3.0 — see LICENSE for the full text.

**V3SP3R** — AI-powered hardware hacking, in your pocket. Your Flipper Zero just got a brain upgrade.
