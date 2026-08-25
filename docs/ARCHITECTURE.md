# Architecture

This document describes the runtime boundaries and real-time protocol used by Cats vs Sharks.

## Runtime Components

### Next.js client

The Next.js App Router serves two client-side experiences:

- `app/host/page.tsx` renders the shared arena with HTML Canvas. It listens for world snapshots and attack effects.
- `app/controller/page.tsx` provides the player join flow, virtual joystick, action buttons, and device-motion handling.
- `app/hooks/useDeviceMotion.tsx` owns browser motion/orientation listeners and permission state.

The root route redirects visitors to the controller route. Static game art is served from `public/assets/`.

### Game server

`server/index.js` creates an Express HTTP server and attaches a Geckos.io server to it. The server is authoritative for all gameplay state and runs a 60 Hz simulation loop.

State is held in memory:

```text
WORLD
├── players: Map<connectionId, Player>
├── gameOver: boolean
└── teams
    ├── shark.gold
    └── cat.gold
```

There is no database-backed match state currently. A server restart creates a new match.

## Connection Flow

1. A controller or host resolves `NEXT_PUBLIC_BACKEND_URL`.
2. If no URL is supplied, the client uses the current fallback backend configured in the page components.
3. Non-HTTPS connections use port `3001`; HTTPS connections rely on the URL's transport configuration.
4. A controller connects and emits `join` with the player's name.
5. The server creates a player, balances team assignment, and includes the player in subsequent `state` broadcasts.
6. On disconnect, the server removes the player from the world.

## Client-to-Server Events

| Event | Payload | Purpose |
| --- | --- | --- |
| `join` | `{ name: string }` | Add a player to the match |
| `input` | `{ dx: number, dy: number }` | Set normalized movement direction |
| `attack` | empty/object payload | Request an attack in the player's current facing direction |
| `shield` | `{ shield: boolean }` | Enable or disable the player's shield |

The server validates connection ownership, alive state, and team relationships before applying gameplay actions.

## Server-to-Client Events

| Event | Payload | Purpose |
| --- | --- | --- |
| `state` | `{ players, teams }` | Full current world snapshot |
| `attack_fx` | `{ x, y, angle }` | Render a transient attack effect |
| `game_over` | `{ winner: "cat" | "shark" }` | Announce the winning team |

The host uses `state` as its render source. It identifies each team's current gold leader to display the MVP sprite and shows attack effects for a short duration.

## Gameplay Model

- World dimensions: `17,000 x 8,000` world units.
- Simulation rate: `60` ticks per second.
- Player speed: `2,300` world units per second.
- Attack shape: a forward rectangle `4,000` units long and `1,200` units wide.
- Respawn delay: `3` seconds.
- A shielded player cannot be eliminated by an attack.
- Friendly players cannot damage one another.
- A defeated player drops carried gold to the attacker.
- Cats collect from the shark-side zone and sharks collect from the cat-side zone.
- Returning to the home side deposits carried gold into that team's bank.
- The match ends when one team's bank reaches zero.

## Local Development

Run the processes independently:

```bash
node server/index.js
npm run dev
```

For a multi-device session, make the Next.js dev server and port `3001` reachable on the local network, then set `NEXT_PUBLIC_BACKEND_URL` to the machine's reachable address before starting Next.js.

The server's `/` route is a simple health check and responds with `Game server is running successfully!` when the process is available.