---
url: https://resident.inanimate.tech/
date_fetched: 2026-07-05
backfilled: true
---

# Resident

Code sandbox with hot reload for ESP32 devices.

## Why Resident

Devices need sandboxes for agents. AI is coming into the real world. Humble devices gain superpowers when they host user code.

A Lua sandbox with hardware IO. Drivers expose approved peripherals. The sandbox sees their events.

Claude, meet websockets. Use the Claude skill plus optional websockets to push and iterate apps. Your end users can swap and load apps too.

## Start building

- Bring up your ESP32 device as normal. Arduino and esp-idf frameworks are supported.
- Add the sandbox with `Resident::Sandbox`and your custom drivers. Resident handles Wi-Fi, a WebSocket, JSON message routing and hot-reloading Lua apps over the network.
- Create and push apps using the built-in Claude skills. Replace the back-end when you're ready.

Read the Resident docs on GitHub ↗

```
#include <Resident.h>
#include "MyDisplayDriver.h"
#include "MyButtonsDriver.h"
MyDisplayDriver display;
MyButtonsDriver buttons;
Resident::SandboxConfig makeConfig() {
    Resident::SandboxConfig cfg;
    cfg.deviceType    = "demo";
    cfg.statusDisplay = &display;
    cfg.extensions    = {&display, &buttons};
    Courier::Config courier;
    courier.host = "your-server.example.com";
    cfg.network  = courier;
    return cfg;
}
Resident::Sandbox sandbox{makeConfig()};
void setup() { sandbox.setup(); }
void loop()  { sandbox.loop();  }
```
Device. Your ESP32 device with connectivity and custom hardware.

Driver. Expose hardware functionality to the Lua sandbox with C++ extensions.

Sandbox. Lua runtime with hot-reloading capabilities and an isolated execution environment.

App. Lua apps that are quick to load, can be loaded over the network, and shared by your users.

Events. Events from hardware drivers and the network — JSON over websockets or MQTT.

Resident builds on Courier ↗ for batteries-included connectivity including Wi-Fi config and JSON messaging.

## Try it now

We love M5StickS3 for prototyping. Before developing your ESP32 device, get this pocket-sized prototyping device from M5. It has an ESP32, 135×240 LCD, buttons, buzzer, IMU and a battery, and is perfect for trying Resident. Buy from Amazon ↗

Use our simulator right now. No hardware required — drag and drop an app onto our M5StickS3 simulator below. The simulator has the Resident sandbox pre-flashed and it is running in your browser.

Create and push apps from your terminal. Install the Resident skills to create, validate and push apps. We provide a back-end relay to load apps via websocket.

Try this in the browser. Tap to connect, then push an app to see the simulator update live.

## Design notes

Sandboxes are a new primitive. We use Resident in all our prototyping at Inanimate, and these same capabilities will be in our products. As we build devices and a platform to bring AI agents into the real world, we’ve found that users need expressive control of their environments, and agents need a place to run code to make generative UI. On-device sandboxes are the foundational building block. Read more about the thinking behind Resident ↗

Made to be composable. We’re building Resident to integrate with existing firmware, frameworks, and back-end services. We love ESP32 so that’s where we’re starting: it’s used by makers, hardware startups, and in mass production. Our goal is that, whatever your hardware and whatever manages your run loop, you can add a sandbox.

Built for agents. AI agents are how we code now, and also the emerging interface for end users. So we have agent-facing docs that walk you through integrating Resident, and skills for the whole sandbox app creation lifecycle. Although there’s no edge AI here, we want to make it easy for agents to hermit crab behavior into the world. Add our marketplace to your Claude Code ↗

## Resident

- Code sandbox with hot reload for ESP32 devices.
- Supported frameworks: esp-idf, Arduino.
- MIT Open Source License.
- Quick start, API docs, agent skills and examples on GitHub.
