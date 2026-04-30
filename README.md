# openanchor
A peer-to-peer sailing app. Live fleet map, voyage logger, crowd-sourced anchorages. No servers. No company. Owned by sailors.
# OpenAnchor

**A living map of the sailing world. No servers. No company. No fees. Owned by the sailors who use it.**

OpenAnchor is an open source, peer-to-peer sailing community app — think flight radar for boats, a crowd-sourced anchorage database, a voyage logger, and a sailor social network, all in one. Built on [Holepunch / Pear Runtime](https://pears.com), every piece of data lives on sailors' devices and syncs directly between them. There is no central server that can be shut down, sold, or monetised.

---

## Why this exists

Every sailing app today makes the same trade: you give them your location, your logbook, your anchorage notes, and your community — and in return you get a subscription bill and a company that owns your data forever.

OpenAnchor refuses that trade.

When you log a passage on OpenAnchor, that log lives on your device. When you add a note to an anchorage, it syncs directly to other sailors nearby. When the company behind an app shuts down — and they do — your data doesn't disappear with it, because there is no company in the middle.

The closest thing to what OpenAnchor wants to be is what AIS is for vessel tracking — a shared, open standard that everyone benefits from and no one owns. Except OpenAnchor adds the social layer, the logbook, the crowd-sourced local knowledge, and eventually a Bitcoin Lightning layer so sailors can be rewarded directly for contributing.

---

## What it does

### The Chart — a living radar of the sailing world
A real-time map of every boat running OpenAnchor, anywhere on earth. Like Flightradar24 but for sailors, and community-owned. Tap any boat to see their vessel, speed, course, and what passage they're on. Pilot tracks layer shows the density of historical passages — trade wind routes glow amber, busy coastal corridors light up, rarely sailed passages show as faint threads.

### Passage — your voyage, logged properly
Start a passage with one tap ("Slip lines"). The app tracks your GPS position, and if your boat has electronic instruments (chartplotter, wind instruments, depth sounder) it reads them automatically over WiFi. Log entries use a smart chip system — tap "Full main", "Reefed main", "NW 20", "Squall", "Engine on" — no typing required. When you arrive ("Make fast"), the app auto-generates a complete passage log: miles sailed, average speed, sail configuration breakdown, wind summary, notable events, and a shareable link anyone can read.

### Harbours — crowd-sourced from real sailors
Anchorages, marinas, fuel docks, tide data — seeded on day one from OpenSeaMap and open government data, then continuously improved by the community. Every note, depth sounding, and conditions report is contributed by a sailor who was actually there. Filters by your vessel's draft so shallow spots warn you before you explore. Works fully offline — pre-cache a region before you sail.

### Fleet — your sailing community
Follow boats, send direct encrypted messages (Hail), organise into Fleets (buddy groups, rally fleets, passage groups). A crew list for finding crew or a berth. Musters for rally coordination. A feed of dispatches from boats you follow — passage reports, anchorage finds, sea state updates. All P2P — no feed algorithm, no ads, no data mining.

### Log — your ship's papers
Every passage ever logged, your lifetime miles and countries, your vessel profile and sail inventory. Multiple vessels supported — the whole app shifts context when you switch boats. Your data, exportable any time as GPX, CSV, or PDF.

---

## What makes it different

| | OpenAnchor | SeaPeople | NoForeignLand | Navionics |
|---|---|---|---|---|
| No central server | ✓ | ✗ | ✗ | ✗ |
| You own your data | ✓ | ✗ | ✗ | ✗ |
| No subscription fee | ✓ | Freemium | Freemium | ✗ |
| Works fully offline | ✓ | Partial | Partial | ✓ |
| Open source | ✓ | ✗ | ✗ | ✗ |
| Bitcoin Lightning | Planned | ✗ | ✗ | ✗ |
| NMEA instrument sync | ✓ | ✗ | ✗ | ✗ |
| Crowd anchorage data | ✓ | Partial | ✓ | ✗ |

---

## The Lightning layer (Phase 5)

OpenAnchor will integrate Bitcoin Lightning payments as a native feature — not bolted on, but designed in from the start.

**Waves** — the OpenAnchor equivalent of a like, but with two depths. Tap to wave (free acknowledgement). Hold to wave with sats (a micropayment directly to that sailor). When someone's anchorage note saves your keel, you can pay them for it. When a passage report helps you plan a crossing, you can wave the crew.

**Pirate treasure** — drop a bounty on the chart and lock sats inside it. Write a task: *"Can someone take a drone photo of the anchorage at 1770 in Queensland?"* or *"Verify the depth on the bar at Shoal Bay in NE 15kn"*. Any sailor nearby can claim the bounty by completing the task and submitting proof. A small group of nearby boats verifies the submission. Sats release automatically. No company takes a cut. The community funds better data for everywhere.

This is built on LNURL-pay and designed to be self-custody first — sailors keep their own keys.

---

## Nautical language throughout

The app speaks like a sailor. You don't "like" a post — you **wave**. You don't "follow" a boat — you **track** it. You don't "start a trip" — you **slip lines**. You don't "arrive" — you **make fast**. Notifications are **watch alerts**. Your profile is your **ship's papers**. History shows as **pilot tracks**. Reviews are **pilot notes**. Groups are **fleets**. Events are **musters**.

---

## How it's built

OpenAnchor is built on [Holepunch](https://github.com/holepunchto) — the same open source P2P stack that powers [Keet](https://keet.io).

```
Core P2P modules:
  Hypercore    — append-only log, one per data feed per vessel
  Hyperswarm   — peer discovery and NAT traversal, no servers
  Autobase     — multi-writer database for shared anchorage data
  Hyperbee     — fast geo-indexed search on top of Hypercore
  Hyperblobs   — media storage (photos, attachments)

Application layer:
  Pear Runtime — desktop app (Mac, Windows, Linux)
  Bare Runtime — mobile app (iOS, Android) — Phase 4
  React        — UI components
  MapLibre GL  — map rendering
  OpenSeaMap   — nautical chart tiles
  H3           — hexagonal geo-indexing for peer discovery
```

### Architecture principle

The `core/` directory contains all P2P logic and has zero UI dependencies. It runs identically on desktop, mobile, and the seed node. The UI layer in `app/` imports from `core/` — never the reverse. This is the discipline that makes the mobile port straightforward rather than a rewrite.

```
openanchor/
├── core/               ← pure P2P logic, no UI
│   ├── identity/       ← keypair, vessel profile, follow graph
│   ├── feeds/          ← Hypercore schemas (position, log, dispatch, vessel)
│   ├── network/        ← Hyperswarm, peer lifecycle, replication
│   ├── harbours/       ← Autobase anchorage DB, OpenSeaMap import
│   └── sync/           ← offline queue, reconnect logic
├── app/                ← React UI
│   ├── screens/        ← Chart, Passage, Fleet, Harbours, Log
│   ├── components/     ← shared UI components
│   └── hooks/          ← bridge between core and UI
├── seednode/           ← marina seed node (Raspberry Pi friendly)
└── scripts/            ← data import, dev tools
```

### Data model — four feeds per vessel

Each vessel has four Hypercore feeds, owned and signed by their keypair:

- **Position feed** — GPS entries appended every 60 seconds underway `{ lat, lon, sog, cog, ts, privacy }`
- **Log feed** — voyage log entries `{ voyageId, ts, chips[], instruments{}, note, pos }`
- **Dispatch feed** — social posts and passage reports `{ type, content, mediaRefs[], pos, ts }`
- **Vessel feed** — profile updates `{ name, type, rig, sailInventory, specs, about, tags }`

Feed public keys are derived deterministically from the vessel's master keypair — anyone who knows your public key can find all your feeds without you sharing them individually.

### Peer discovery — geographic topics

Boats announce on two Hyperswarm topics simultaneously:
1. A global fleet topic — for discovering followed boats anywhere in the world
2. A local H3 cell topic (resolution 3, ~800km cells) — for fast discovery of nearby boats

This two-tier approach means local discovery is fast even as the global fleet grows.

---

## Build roadmap

| Phase | Focus | Status |
|---|---|---|
| 1 | P2P data foundation — identity, feeds, Hyperswarm, Autobase | 🔨 In progress |
| 2 | Desktop + web MVP — Chart map, Passage tab, Harbours seeded | Planned |
| 3 | Community layer — Fleet, dispatches, crew list, musters, safety | Planned |
| 4 | NMEA integration + native mobile iOS/Android via Bare Runtime | Planned |
| 5 | Bitcoin Lightning — waves, pirate treasure bounties | Planned |

### Phase 1 milestone — the only test that matters

Two devices on different networks find each other with zero servers, share live position data, sync log entries, and query anchorage data — all offline-capable. When a Wireshark packet capture confirms no connection to any centralised server at any point, Phase 1 is done.

---

## Contributing

OpenAnchor is looking for its first collaborator — a developer comfortable with Node.js who is excited about P2P technology and ideally knows their way around a boat.

**What we need most right now:**
- Phase 1 P2P core implementation (Hypercore, Hyperswarm, Autobase)
- OpenSeaMap data import pipeline
- Basic Pear Runtime desktop shell

**Payment:** Bitcoin / Lightning. Tasks are scoped and bounties set per milestone. Start with one task, no long commitments required.

**To get involved:**
1. Read through the issues tab — Phase 1 tasks are broken down individually
2. Open a discussion if you want to talk through the architecture
3. Or just reach out directly — see contact below

All skill levels welcome for smaller tasks. You don't need to know Holepunch specifically — Node.js experience and enthusiasm for the problem is enough to get started.

---

## Licence

MIT — free to use, modify, and distribute. The data belongs to the sailors who create it, not to any company.

---

## The name

An open anchor is a traditional symbol used on nautical charts to mark an anchorage. It's also what this app is — open, shared, and a safe place to stop.

---

*Built by sailors, for sailors. No VC funding. No ads. No data harvesting. Just the sea.*
