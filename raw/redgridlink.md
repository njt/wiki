---
url: https://github.com/RedGridTactical/RedGridLink
date_fetched: 2026-07-05
backfilled: true
---

**Offline MGRS navigation and proximity team coordination for small teams (2-8 people). No cell service needed.**

Built on the MGRS engine from Red Grid MGRS. Field Link adds zero-config proximity sync over Bluetooth, with Apple Multipeer Connectivity (iOS) and Google Play Services Nearby Connections (Android) running alongside as a parallel higher-bandwidth transport -- your team appears on the map the moment they're in range.


Solo navigator?Red Grid MGRS gives you the same MGRS engine as a standalone tool with DAGR-class precision, 9 tactical tools, and 6 radio-ready report templates. Part of the Red Grid Tactical ecosystem.


Available now on iOS.Android version in closed beta testing.

| Team Map | MGRS Grid | Field Link | Tools | Themes | 
|---|---|---|---|---|

| Peer Detail | Dead Reckoning | Celestial Nav | Search Area | Team Roster | 
|---|---|---|---|---|

Live Military Grid Reference System coordinates with 1-meter precision. GPS Kalman filter for smooth, accurate position tracking. MGRS grid overlay on offline maps from GZD down to 100m resolution. Bearing, distance, dead reckoning, resection, pace count (with accelerometer step detection), declination, and coordinate conversion tools. NATO phonetic voice readout for hands-free grid calls.

Zero-config proximity sync over BLE on all platforms, with Apple Multipeer Connectivity (AWDL) added on iOS for higher-bandwidth peer transport when the app is backgrounded. Devices within range automatically discover each other and share position, marker, and annotation data. No internet required. No pairing codes. No servers. (Android Wi-Fi Direct / Nearby Connections is on the roadmap; the shipping Android build is BLE-only.)

- 2-8 devices per session
- AES-256-GCM encryption with ECDH P-256 ephemeral keys for PIN and QR sessions; Open sessions are unencrypted by design (training / demo use)
- Tiered session security: Open (auto-join, no encryption), PIN (4-digit, encrypted), QR code (host-generated session secret, encrypted)
- Delta payloads under 200 bytes per position update
- Ghost markers with time-decay visualization when teammates disconnect
- Velocity vectors project last-known movement direction
- Expedition Mode: <3% battery/hr (BLE-only, 30s updates)
- Ultra Expedition Mode: <2% battery/hr (BLE-only, 60s updates)
- Auto-reconnect with exponential backoff on disconnect

Download map packs from OpenStreetMap or OpenTopoMap to MBTiles for offline operation, with MGRS grid lines rendered as a dynamic overlay. Region downloads are throttled to respect public-tile-server usage policies; for sustained heavy offline usage we recommend a licensed provider. (Native USGS / Mapbox / MapTiler integrations are on the roadmap.)

One engine, four presentation layers. Terminology, icons, and quick actions adapt to your mission:

- **Search & Rescue**-- sector assignments, clue markers, search patterns
- **Backcountry**-- camp, waypoint, and trail navigation
- **Hunting**-- stand locations, game sightings, property boundaries
- **Training**-- exercise objectives, rally points, phase lines

Dead Reckoning, Resection, Pace Count, Bearing/Back Azimuth, Coordinate Converter (MGRS/Lat-Lon/DMS/UTM), Range Estimation, Slope Calculator, ETA/Speed Calculator, Magnetic Declination, Celestial Navigation, MGRS Precision Reference.

Assign roles (Lead, Scout, Medic, Comms, custom) with callsigns. Lead controls the session like a group admin. Share waypoints with the whole team or save them privately. Draw tap-to-place annotations visible to all peers. Set boundary geofences with automatic alerts when someone crosses. NATO phonetic voice callouts announce teammate positions hands-free. Export and import sessions as versioned JSON for backup and review.

BLE Long Range (Coded PHY) extends range from ~100m to 400m-1km on supported devices. Live RSSI signal bars show connection quality for each teammate with warnings when signal weakens. FixPhrase encodes any location as 4 easy-to-remember words (~11m accuracy, order-independent). Choose between OpenStreetMap or OpenTopoMap when downloading offline regions. Coordinate bar cycles between MGRS and FixPhrase display.

Real ECDH P-256 key exchange with per-peer derived encryption keys. Actual BLE Coded PHY negotiation for extended range on Android. One-tap emergency beacon sends GPS coordinates to all team members with 30-second retransmission. 7 pre-canned tactical messages (HELP, STOP, RALLY ON ME, ALL CLEAR, FOUND SOMETHING, HEADING BACK, NEED SUPPLIES) plus 160-character free text over encrypted CRDT sync.

One-tap PDF export: map snapshot, mission timeline, track data, timestamps, team roster with roles, per-member tracks, boundary events, markers, and session log. Share via AirDrop, file share, or any local transfer.

Red Light (night vision, free), NVG Green (Pro), Day White (Pro), Blue Force (Pro).

Open Red Grid Link and your MGRS position appears on the offline map. Navigate using bearing, distance, and dead reckoning tools -- identical to Red Grid MGRS but with a full map view and 11 tactical tools.

- **Start a session**-- tap one button to begin broadcasting over BLE
- **Set security**-- choose Open, PIN, or QR code authentication
- **Teammates appear**-- any device running Red Grid Link within range (~100-300m standard, up to 1km with BLE Long Range) is automatically discovered
- **Positions sync**-- delta updates flow between all devices at configurable intervals; PIN and QR sessions wrap each delta in an AES-256-GCM envelope, Open sessions send plaintext
- **Ghosting**-- if a teammate moves out of range, their last-known position remains on your map with time-decay opacity (100% to outline over 30 minutes)
- **Reconnect**-- when a ghost comes back in range, their marker snaps to live position

No accounts. No servers. No cell service. No configuration. It just works.

| Feature | Free | Pro | Pro+Link | Team | 
|---|---|---|---|---|
| MGRS Navigation | Yes | Yes | Yes | Yes | 
| All Operational Modes | Yes | Yes | Yes | Yes | 
| 11 Tactical Tools | Yes | Yes | Yes | Yes | 
| Field Link (2 devices) | Yes | Yes | Yes | Yes | 
| All Themes | -- | Yes | Yes | Yes | 
| Unlimited Map Downloads | -- | Yes | Yes | Yes | 
| AAR Export | -- | Yes | Yes | Yes | 
| Full Field Link (8 devices) | -- | -- | Yes | Yes | 
| Team Management | -- | -- | -- | Yes | 

**Pricing:**

- **Free**-- All modes, 2-device Field Link, 1 map region, Red Light theme
- **Pro**-- $3.99/mo or $29.99/yr
- **Pro+Link**-- $5.99/mo or $44.99/yr (Pro + full 8-device Field Link)
- **Team**-- $199.99/yr (8 seats, includes Pro+Link for all members)
- **Lifetime**-- $149.99 one-time (Pro+Link forever)

| Data | Collected | Stored | Transmitted | 
|---|---|---|---|
| GPS location | In use / background (sessions) | Local session DB | Field Link peers only (PIN / QR: AES-256-GCM; Open: plaintext) | 
| Field Link positions | Active session | Local DB until you delete the session | AES-256-GCM in PIN/QR sessions, plaintext in Open sessions; always device-to-device | 
| Map tiles | Downloaded | Local MBTiles | Standard HTTPS to tile servers (OSM / OpenTopoMap) | 
| Waypoints & markers | User-created | Local DB | Field Link peers only (encrypted in PIN/QR sessions) | 
| After-Action Reports | User-generated | Local only | Never | 
| Device identifiers | Never | Never | Never | 

No accounts. No analytics. No ad networks. No third-party data SDKs. Privacy-safe crash reporting (Sentry, release mode only — no location data transmitted). In-app purchases processed by Apple/Google -- Red Grid Link never sees your payment details. Full details in PRIVACY.md.

```
git clone https://github.com/RedGridTactical/RedGridLink.git
cd RedGridLink
flutter pub get
flutter pub run build_runner build --delete-conflicting-outputs
flutter run
```
Requires Flutter SDK. Currently targeting iOS -- Android support is planned. All free features work from source. Pro features require a valid purchase through Apple. Field Link requires Bluetooth and location permissions on physical devices.

Full roadmap with feature checklists: ROADMAP.md

| Version | Target | Theme | Highlights | 
|---|---|---|---|
| V1.0 | Complete | Foundation | MGRS nav, Field Link (BLE+peer-to-peer Wi-Fi), 11 tools, AAR PDF, offline maps, 4 themes, IAP | 
| V1.1 | Complete | Field Hardening | Kalman filter, step detector, Peer HUD, Ultra Expedition, auto-reconnect, map downloads, Sentry, l10n, Help/About | 
| V1.2.1 | Complete | Reliability | Field Link session fix, waypoint persistence, relative bearing arrow, demo mode | 
| V1.3 | Complete | Team Features | Team roles (Lead/Scout/Medic/Comms), waypoint sharing, shared annotations, boundary alerts, NATO voice callouts, session export/import | 
| V1.4 | Q3 2026 | Android + Range | Android launch, BLE Long Range (Android, Coded PHY), FixPhrase (4-word locations), F-Droid, OSM tiles | 
| V2.0 | Q4 2026 | Intelligence + Interop | ATAK/CoT interop, Meshtastic BLE bridge, elevation profiles, terrain analysis, weather overlay | 
| V2.1 | Q1 2027 | Advanced Nav | Route planning, freehand annotations, track recording, GPX import/export, timeline replay | 
| V3.0 | Q2 2027 | Connected Ops | Cloud relay, web dashboard, mesh networking, session scheduling, API | 
| V3.1 | Q3 2027 | Sensors | Garmin inReach, external GPS, drone overlay, heart rate monitoring | 
| V4.0 | Q4 2027 | Training | Scenario builder, instructor mode, scoring, AR compass, certification tracking | 

- Security audits and cryptographic library updates
- Battery performance optimization
- Map tile source expansion
- Test coverage expansion (target 90%+)
- Store listing optimization and A/B testing

Red Grid Link is built in the open. We welcome bug reports, feature requests, and pull requests.

- **Report a bug:**Open an issue
- **Request a feature:**Start a discussion
- **Submit a PR:**Fork, branch, and open a pull request

See the Roadmap for planned features and where help is needed.

| App | Purpose | Platform | Link | 
|---|---|---|---|
| Red Grid MGRS | Solo MGRS navigator (DAGR-class) | iOS | GitHub · App Store | 
| Red Grid Link | Team coordination + encrypted sync | iOS | GitHub · App Store | 

Website: redgridtactical.com

MIT + Commons Clause -- free for personal non-commercial use. Commercial and organizational deployment requires written permission.

Contact: support@redgridtactical.com

*Your team. Your grid. No cell towers required.*

If Red Grid Link helps you stay coordinated in the field, give it a star and share it with your team.
