# OTS Transporter Architecture

How the Ontempo Store Transporter works -- the messaging layer that syncs data between OTR (head office), the store controller, and individual POS tills.

## Architecture: Hub-and-Spoke, Not Peer-to-Peer

The Transporter is not a direct conversation between OTR and OTS. It's a hub-and-spoke system built on MSMQ's remote queue capability. There are three layers:

1. **Master Router** (typically called the "Store Controller") -- one per store, running the Transporter Engine with `Identity.IsMasterRouter = true`.
2. **Child nodes** (individual POS tills, kiosks, etc.) -- each runs its own Transporter Engine, configured with the Master Router's POS number, conduit, and endpoint in their local Identity file.
3. **Head Office (OTR)** -- does not run the OTS Transporter. The bridge is the `Mgmt2000MessageHandler` on the Master Router, which handles OTR-specific replication and service calls.

```
OTR (Head Office)
     |
     | (MSMQ remote queues on the store's Master Router machine)
     v
Store Controller / Master Router  <-- runs Mgmt2000MessageHandler
     |
     | (MSMQ remote queues to/from each POS)
     v
POS 1, POS 2, POS 3, Kiosks...
```

## How MSMQ Provides the "Port"

MSMQ doesn't use TCP ports in the traditional sense. Instead, each device has three named queues registered in the `NetConfigTransports` / `NetConfigTransportMasks` database tables:

- **DataMessagePath** -- for data/replication messages
- **SystemMessagePath** -- for system messages (acks, pings, etc.)
- **ServiceCallMessagePath** -- for synchronous-style request/response calls

The queue names are built from templates in `NetConfigTransportMasks` where `<deviceaddress>` is substituted with the actual machine name/IP:

```sql
REPLACE(ntm.[SystemMessagePathMask], '<deviceaddress>', nt.[DeviceAddress]) AS [SystemMessagePath],
REPLACE(ntm.[ServiceCallMessagePathMask], '<deviceaddress>', nt.[DeviceAddress]) AS [ServiceCallMessagePath],
REPLACE(ntm.[DataMessagePathMask], '<deviceaddress>', nt.[DeviceAddress]) AS [DataMessagePath]
```

The actual queue format is like `FormatName:DIRECT=OS:<machinename>\PRIVATE$\Ontempo Store Transporter`. The default incoming queue name is `.\PRIVATE$\Ontempo Store Transporter`.

MSMQ doesn't expose a listening port that you configure -- it uses its own internal TCP transport (port 1801 by default) managed by the Windows MSMQ service. What you configure in the Transporter is the queue name (which includes the remote machine's hostname), and MSMQ's native remote-queue feature handles the actual network delivery.

## Message Flow: Sending

When the Transporter wants to send a message:

1. `Engine.SendMessage()` is called with a `Message` that has a `Destination` like `"DEVICE:POS001"` or `"all"`.
2. The **Router** (`TreeRouter` or `GroupRouter`) resolves the destination to a list of target device keys. `TreeRouter` supports both flat (direct to each client) and tree-based routing depending on the `Transporter\Router\Use Tree Replication` parameter.
3. For each target, the Engine resolves the device key to a `NameEntry` via `ResolveByDestinationKey()`, which looks up the `NetConfigTransports` table to get the device's queue paths and conduit (transport type).
4. The Engine calls `transport.SendMessage(message, ne.DataPath)` -- the endpoint is the remote MSMQ queue path on the target machine.
5. The `MSMQTransport` opens the remote queue by name and does `queue.Send(msg)`. MSMQ handles delivery, buffering locally if the remote machine is offline (store-and-forward).

## Message Flow: Receiving

Each Transporter listens to its three local queues using async peek (`BeginPeek`). When a message arrives:

1. The `PeekCompleted` event fires, deserialising the MSMQ message into a Transporter `Message`.
2. The Engine dispatches to the appropriate `IMessageHandler` based on message type.
3. If routing is enabled (`UseRouting = true`), the Engine also forwards messages to child nodes that need them.

## Routing

The `TreeRouter` resolves destinations by:

- **Special destinations**: `"all"` sends to all known nodes (or just children if tree replication is on); `"local"` sends to self.
- **Device destinations**: `"DEVICE:<posnumber>"` resolves directly to a specific POS.
- **Group destinations**: looked up via `GroupMembers` / `GroupKeys` tables in the database, supporting nested groups.
- **Virtual routes**: destinations starting with `"Route:"` delegate to pluggable `IVirtualRoute` components.

The `GroupRouter` (flat router) is simpler -- it always sends directly to the message's destination without tree forwarding.

## OTR to OTS: The Master Router Bridge

OTR doesn't talk MSMQ to OTS directly. The Master Router in each store has:

- **`Mgmt2000MessageHandler`** -- the main bridge. It processes incoming replication from OTR (products, customers, pricing, promotions) and sends outgoing data back (dockets/transactions, customer edits, stock counts, etc.) as `REPLICATION_MGMT2000` messages.
- **`StandardSqlReplicationMonitor`** -- watches for changes in the local database's replication queue and packages them as outgoing messages. There are two monitors: one for the Mgmt2000 replication set (data going to/from OTR) and one for the OTS Master replication set (data distributed to child POS nodes within the store).
- **`GusIntegrationMessageHandler`** -- for certain operations that call the OTR GUS (Grand Unified Service) web service directly via WCF.

## Message Types

Messages have a `Type` string that determines which handler processes them. Examples:

- `REPLICATION_MGMT2000` -- outgoing replication data from store to head office
- `REPLICATION_OTSMASTER` -- data replicated within the store from master to tills
- `OTP_BOOTSTRAP_REQUESTDATABASE` / `OTP_BOOTSTRAP_DATABASESEGMENT` -- initial database provisioning for new tills
- `OTP_POISONED` -- messages that failed processing, sent back to the master router for retry/investigation
- Various `OTS_MGMT2000_*` types for specific business operations (dockets, customer edits, stock counts, etc.)

## Key Database Tables

- **`NetConfigIdentities`** -- every device in the network (POS number, parent, device type, identity)
- **`NetConfigTransports`** -- device-to-transport mapping (POS number, conduit type, device address)
- **`NetConfigTransportMasks`** -- queue name templates per conduit type
- **`GroupMembers` / `GroupKeys`** -- logical groups for routing (e.g., "all tills in branch X")

## Key Configuration Parameters

| Parameter Path | Default | Purpose |
|---|---|---|
| `Transporter\MSMQ Transport\Incoming Queue Name` | `.\PRIVATE$\Ontempo Store Transporter` | Local queue to listen on |
| `Transporter\MSMQ Transport\Use Recoverable Messages` | `true` | MSMQ persists messages to disk for durability |
| `Transporter\MSMQ Transport\Ping Server To Check Status` | `true` | Periodically ping master router to check connectivity |
| `Transporter\MSMQ Transport\Connected Check Interval (Minutes)` | `5` | How often to check if still connected |
| `Transporter\Router\Use Tree Replication` | `false` | If true, messages are forwarded through the tree hierarchy rather than sent direct |

## Poisoned Messages

When a message fails processing, it's sent back to the master router as an `OTP_POISONED` message. The master router can attempt to resend it later. There's also a `PoisonedMessageResender` web application for manual intervention. Messages are classified as retryable or not based on the exception type (transient DB errors, timeouts, etc. are retryable; business logic failures are not).

## Source Code Layout

```
Ontempo Store/Main/Transporter/
  Lib/Win32/              -- Core: Engine, ITransport, IRouter, Message, NameEntry, Identity
  MSMQTransport/Win32/    -- MSMQ transport implementation
  Mgmt2000MessageHandler/ -- OTR bridge (replication, service calls, GUS integration)
  Console/Win32/          -- Standalone console host
  Service/Win32/          -- Windows service host
  Monitor/Win32/          -- GUI monitoring tool
  PoisonedMessageHandler/ -- Failed message handling
  PoisonedMessageResender/-- Web UI for resending failed messages
  TableStorage/           -- Azure Table Storage for message tracking
  ConfigurationMessageHandler/ -- Remote configuration updates
  DeploymentMessageHandler/    -- Remote software deployment
  BranchTasks/            -- Store task management messages
  AdvancedOffers/         -- Promotion data replication
  StoreManager/           -- Store manager app integration
  WatchDogService/        -- Health monitoring
```
