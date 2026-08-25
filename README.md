# Cats vs Sharks

**Cats vs Sharks** is a browser-based multiplayer game built for Horizons Nexus. Players join one of two teams, collect the opposing team's gold, and carry it back to their own side while trying to outmaneuver and eliminate opponents.

> **Award:** 3rd place at Horizons Nexus.

The game is designed for a shared-screen setup: one browser displays the arena for the host, while each player uses a phone or another browser tab as a controller.

## How It Works

- Open `/host` on the shared display to show the live arena.
- Open `/controller` on each player's device.
- Players enter a name and join the game.
- Movement is controlled with the on-screen joystick.
- Supported devices can use motion gestures for attack and shield actions.
- Cats and sharks steal gold from the opposing side and return it to their own base.
- A successful attack eliminates an opposing player temporarily and transfers their carried gold.
- The first team to empty the opposing team's gold wins.

## Tech Stack

- **Next.js 16** with the App Router
- **React 19** and **TypeScript** for the web client
- **Node.js** and **Express 5** for the game server
- **Geckos.io** for real-time UDP/WebRTC-style client-server communication
- **Socket.IO** packages are included for networking experimentation and compatibility
- **Tailwind CSS 4** and PostCSS for styling support
- **HTML Canvas** for the host arena renderer
- **Device Motion and Device Orientation APIs** for gesture controls
- **Supabase client packages** are installed for future persistence or identity features

## Prerequisites

- Node.js 20 or newer
- npm
- A modern browser
- A network connection between the host display, player devices, and game server

For motion controls, use a device and browser that support `DeviceMotionEvent` and `DeviceOrientationEvent`. iOS browsers may require permission from a user gesture and generally require HTTPS outside local development.

## Getting Started

Install dependencies:

```bash
npm install
```

Start the real-time game server in one terminal:

```bash
node server/index.js
```

Start the Next.js app in a second terminal:

```bash
npm run dev
```

Then open:

- Host display: `http://localhost:3000/host`
- Player controller: `http://localhost:3000/controller`
- Server health check: `http://localhost:3001/`

The client defaults to the configured fallback backend when `NEXT_PUBLIC_BACKEND_URL` is not set. For local development, explicitly set the backend URL:

```bash
NEXT_PUBLIC_BACKEND_URL=http://localhost npm run dev
```

If players are joining from other devices, replace `localhost` with a hostname or LAN address reachable from those devices. The Geckos client uses port `3001` for non-HTTPS backend URLs.

## Controls

### Movement

Touch and drag anywhere on the controller surface to move the virtual joystick. The direction and magnitude are sent to the server at a fixed interval.

### Attack

- Tap **ATTACK** to attack in the current movement direction.
- On supported devices, a sharp motion gesture can trigger an attack.
- Attacks have a short cooldown and cannot be used while shielding.

### Shield

- Tap **SHIELD** to activate protection.
- On supported devices, a strong tilt can trigger the shield.
- The shield lasts briefly and prevents opposing attacks during that window.

## Project Structure

```text
.
├── app/
│   ├── controller/
│   │   └── page.tsx          # Player join screen and touch/motion controls
│   ├── host/
│   │   └── page.tsx          # Full-screen Canvas arena and game HUD
│   ├── hooks/
│   │   └── useDeviceMotion.tsx # Motion/orientation permission and event hook
│   ├── layout.tsx             # Root metadata and page shell
│   └── page.tsx               # Entry route; redirects to the controller
├── public/assets/
│   ├── map/                   # Arena backgrounds
│   ├── objects/               # Shield, fire, and other world objects
│   ├── sprites/               # Cat, shark, and MVP sprites
│   └── ui/                    # Controller and loading-screen imagery
├── server/
│   └── index.js               # Express + Geckos.io server and game loop
├── next.config.ts             # Next.js configuration
├── postcss.config.mjs         # PostCSS/Tailwind integration
└── package.json               # Scripts and dependencies
```

## Architecture

The browser client does not own authoritative game state. The server maintains players, teams, movement, attacks, respawns, gold, and win conditions. Clients send input and action events; the server broadcasts snapshots and short-lived visual effects.

The server runs a 60 Hz simulation loop. The controller sends movement input approximately 20 times per second. The host subscribes to state updates and renders the current world on a full-screen Canvas.

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the event protocol and gameplay model.

## Available Scripts

```bash
npm run dev    # Start the Next.js development server
npm run build  # Create a production build
npm run start  # Serve the production build
npm run lint   # Run ESLint
```

The game server currently runs directly with `node server/index.js`; it is intentionally separate from the Next.js process.

## Production Notes

- Set `NEXT_PUBLIC_BACKEND_URL` to the public game-server hostname or URL when deploying the web client.
- Ensure the game server's configured port is reachable by all clients.
- Use HTTPS when motion permissions or browser security policies require it.
- The current server stores all game state in memory. Restarting the server resets the match.
- The host and controller should connect to the same game-server instance.

## Credits

Built for Horizons Nexus by the Nexus Project team.